# Engineering Runbook: Cross-Platform CI/CD Pipeline (Mac to Windows Docker Host)

## Overview & Architecture
This document details the configuration, troubleshooting decisions, and operational runbook for automating the Python-based SEC data ingestion pipeline (`fund-etl`).

* **CI/CD Controller:** Jenkins running natively on macOS (Apple Silicon).
* **Execution & Deployment Host:** Windows 11 running Docker Desktop with a native PostgreSQL instance on the host network.
* **Secret Management:** Doppler Cloud (`sec-pipeline` project, `prd` environment).
* **Communication Protocol:** Secure Shell (SSH) streaming an ephemeral build context from macOS directly into the Windows Docker daemon via standard input (`stdin`).

```
+-------------------+             SSH / Port 22             +-------------------------+
|   macOS (Host)    | ------------------------------------> |   Windows 11 (Host)     |
|                   |                                       |                         |
| 1. Jenkins Job    |  tar -czf - . | ssh docker build -    | 1. OpenSSH Server       |
| 2. Doppler Secret |                                       | 2. Docker Desktop       |
| 3. Local Git Repo |  ssh docker run --rm                  | 3. PostgreSQL (Native)  |
|                   |  -e DATABASE_URL                      |    host.docker.internal |
+-------------------+                                       +-------------------------+
```

---

## Part 1: Chronological Issues, Root Cause Analyses & Resolutions

### Phase 1: Pipeline Initialization & Secret Fetch Warnings

#### Issue 1.1: Insecure Groovy String Interpolation in Jenkinsfile
* **Symptom:**
  ```text
  Warning: A secret was passed to "sh" using Groovy String interpolation, which is insecure.
  Affected argument(s) used the following variable(s): [DOPPLER_TOKEN]
  ```
* **Root Cause:** Enclosing shell commands in double quotes (`"..."`) causes Groovy to evaluate and substitute secrets before invoking the shell process, exposing tokens to workspace script inspection.
* **Resolution:** Wrap the shell payload in single quotes (`'''...'''`) and let the native shell expand the environment variable injected by Jenkins credentials.
* **Implementation:**
  ```groovy
  sh script: '''
      curl -s --request GET \
        --url "[https://api.doppler.com/v3/configs/config/secrets/download?project=$](https://api.doppler.com/v3/configs/config/secrets/download?project=$){DOPPLER_PROJECT}&config=${DOPPLER_CONFIG}&format=json" \
        --header "accept: application/json" \
        --header "authorization: Bearer ${DOPPLER_TOKEN}"
  ''', returnStdout: true
  ```

---

### Phase 2: Host Network & SSH Connectivity

#### Issue 2.1: TCP Connection Timeout on Port 22
* **Symptom:**
  ```text
  + ssh -i **** -o StrictHostKeyChecking=no shiva@192.168.1.9 'docker build -t fund-etl:latest -'
  ssh: connect to host 192.168.1.9 port 22: Operation timed out
  tar: Write error
  ```
* **Root Causes:**
  1. Windows dynamic power management put the Wi-Fi adapter or CPU into a low-power sleep state, dropping the OpenSSH listener.
  2. Windows network adapter profile defaulted to `Public`, prompting Windows Defender Firewall to drop incoming port 22 packets.
* **Diagnostic Commands:**
  * **macOS:**
    ```bash
    nc -zv -w 3 192.168.1.9 22
    ```
  * **Windows (PowerShell Admin):**
    ```powershell
    Get-Service sshd | Select-Object Status, StartType
    Get-NetConnectionProfile | Select-Object InterfaceAlias, NetworkCategory
    ```
* **Resolution:**
  1. Set Windows Wi-Fi category to `Private`:
     ```powershell
     Get-NetConnectionProfile | Set-NetConnectionProfile -NetworkCategory Private
     ```
  2. In Windows Settings (`System > Power & battery`):
     * Set *When plugged in, put my device to sleep after* to **Never**.
  3. Added fast-fail timeout flags to all SSH commands: `-o ConnectTimeout=15`.

---

### Phase 3: Windows Named Pipe & Non-Interactive SSH Permissions

#### Issue 3.1: Docker Engine Named Pipe Inaccessible over SSH
* **Symptom:**
  ```text
  open //./pipe/docker_engine: The system cannot find the file specified.
  In the default daemon configuration on Windows, the docker client must be run with elevated privileges to connect.
  ```
* **Root Cause:**
  1. Docker Desktop communicates over the named pipe `\\.\pipe\docker_engine`.
  2. OpenSSH sessions spawn non-interactive shells that strip UAC elevation tokens, preventing standard users from accessing the pipe unless explicit local group memberships exist.
* **Resolution:**
  1. Add the SSH user to the local `docker-users` security group (PowerShell Admin):
     ```powershell
     net localgroup docker-users "shiva" /add
     ```
  2. In Docker Desktop Settings (`General`), enable **Expose daemon on tcp://localhost:2375 without TLS** as a local fallback.

---

### Phase 4: Docker Desktop WSL2 Hang & Memory Starvation

#### Issue 4.1: Orange Docker Desktop Tray Icon (Infinite Starting State)
* **Symptom:** Docker Desktop remained stuck in the "starting" state for over 5 minutes.
* **Root Cause:** WSL2 attempted to allocate more memory than was available on the host, causing an RPC deadlock between Docker's VHDX storage engine and Windows host services.
* **Resolution:**
  1. Force-terminate hung subsystems:
     ```powershell
     Get-Process "*docker*", "*wsl*" -ErrorAction SilentlyContinue | Stop-Process -Force
     wsl --shutdown
     Restart-Service LxssManager -Force
     ```
  2. Limit WSL resource allocation via `C:\Users\shiva\.wslconfig`:
     ```ini
     [wsl2]
     memory=4GB
     processors=2
     swap=2GB
     guiApplications=false
     ```
  3. Restart Docker Desktop:
     ```powershell
     Start-Process "C:\Program Files\Docker\Docker\Docker Desktop.exe"
     ```

---

### Phase 5: Windows Credential Helper & Non-Interactive Session Conflicts

#### Issue 5.1: Windows Logon Session Error During BuildKit Resolution
* **Symptom:**
  ```text
  #3 [internal] load metadata for docker.io/library/python:3.14-slim
  #3 ERROR: error getting credentials - err: exit status 1, out: `A specified logon session does not exist. It may already have been terminated.`
  ```
* **Root Cause:** Docker Desktop delegates authentication to `docker-credential-desktop.exe` or `wincred.exe`. Over an SSH shell without an interactive desktop logon session, the Windows Credential Manager fails when evaluating credentials—even for public registry images.

#### Issue 5.2: UTF-8 BOM Corruption in Docker Configuration
* **Symptom:**
  ```text
  WARNING: Error parsing config file (C:\Users\shiva\.docker\config.json): invalid character 'ï' looking for beginning of value
  ```
* **Root Cause:** PowerShell's default output encoding wrote `config.json` with a UTF-8 Byte Order Mark (`0xEF, 0xBB, 0xBF`), which broke Go's JSON parser.
* **Resolution:**
  1. Write a clean, BOM-free ASCII JSON payload directly to the user profile:
     ```powershell
     [System.IO.File]::WriteAllText("C:\Users\shiva\.docker\config.json", '{"auths":{},"credsStore":""}', [System.Text.Encoding]::ASCII)
     ```
  2. Verify byte structure contains no leading BOM (`7B 22 61 75 74 68 73` corresponds to `{"auths`):
     ```powershell
     Format-Hex "C:\Users\shiva\.docker\config.json"
     ```
  3. Neutralize the fallback executable on Windows:
     ```powershell
     Rename-Item "C:\Program Files\Docker\Docker\resources\bin\docker-credential-desktop.exe" "docker-credential-desktop.exe.bak" -Force
     ```

---

### Phase 6: Build Layering, Dependency Caching & Entrypoint Optimization

#### Issue 6.1: `uv sync` Build Failure Due to Premature Local Wheel Resolution
* **Symptom:**
  ```text
  error: Failed to build `fund-etl @ file:///app`
    cause: Expected a Python module at: src/fund_etl/__init__.py
  ```
* **Root Cause:** In the `Dockerfile`, `RUN uv sync --frozen --no-dev` was executed *before* `COPY src/ ./src/`. `uv` attempted to install the workspace root as an editable/wheel package before its source files were in the build context.
* **Resolution:** Split the caching logic by running `uv sync --no-install-project` first, copy the application code, and rely on `ENV PYTHONPATH=/app/src` rather than building an unnecessary wheel.

#### Issue 6.2: Missing PostgreSQL Client Libraries (`libpq`) at Runtime
* **Symptom:**
  ```text
  ImportError: no pq wrapper available.
  Attempts made:
  - couldn't import psycopg 'c' implementation: No module named 'psycopg_c'
  - couldn't import psycopg 'binary' implementation: No module named 'psycopg_binary'
  - couldn't import psycopg 'python' implementation: libpq library not found
  ```
* **Root Cause:** `python:3.14-slim` is a minimal Debian image with no pre-installed `libpq.so.5`[cite: 1]. While macOS resolved the library via Homebrew dynamic linkers, the container lacked both compiled C binaries and system headers.
* **Production Resolution:** Standardize on `psycopg[binary]>=3.2.0` in `pyproject.toml`. This bundles audited `libpq` binaries inside the wheel, keeping the runtime container slim and eliminating the need for `build-essential` or `libpq-dev` system packages.

#### Issue 6.3: CLI Argument Parsing and Unnecessary On-Boot Package Compilation
* **Symptom:**
  ```text
  Building fund-etl @ file:///app
  Downloading pygments
  usage: main.py [-h] [--dry-run] --year YEAR --quarter {1,2,3,4} --ticker TICKER [--force]
  main.py: error: the following arguments are required: --year, --quarter, --ticker
  ```
* **Resolution:**
  1. Update `ENTRYPOINT` in `Dockerfile` to `["uv", "run", "--no-sync", "python", "main.py"]` to prevent `uv` from evaluating dependencies dynamically during container startup.
  2. Update the `Jenkinsfile` argument construction to include all required parameters (`--year`, `--quarter`, `--ticker`) regardless of whether `--dry-run` is active.

---