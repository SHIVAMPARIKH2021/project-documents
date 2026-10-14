## Part 1: Production Artifacts
- Make sure below files are available in the code repository.

### 1. `Dockerfile`
```dockerfile
FROM python:3.14-slim

WORKDIR /app

# Pin uv to maintain consistency with the build-backend requirement
RUN pip install "uv>=0.12.13,<0.13.0"

# Copy dependency specifications
COPY pyproject.toml uv.lock ./

# Install external dependencies strictly into the virtual environment
RUN uv sync --frozen --no-dev --no-install-project

# Copy project files
COPY README.md* ./
COPY src/ ./src/
COPY main.py ./

# Make source tree discoverable to Python runtime
ENV PYTHONPATH=/app/src

# Execute directly using pre-compiled virtual environment without re-sync
ENTRYPOINT ["uv", "run", "--no-sync", "python", "main.py"]
```

### 2. `pyproject.toml`
```toml
[project]
name = "fund-etl"
version = "0.1.0"
description = "SEC EDGAR Form N-PORT & N-1A Ingestion Pipeline"
requires-python = ">=3.14"
dependencies = [
    "psycopg[binary]>=3.2.0",
    "pydantic>=2.10.0",
    "requests>=2.32.0",
]

[build-system]
requires = ["uv-build>=0.12.13,<0.13.0"]
build-backend = "uv_build"
```

### 3. `Jenkinsfile`
```groovy
pipeline {
    agent any

    parameters {
        booleanParam(name: 'DRY_RUN', defaultValue: true, description: 'Verify database and SEC endpoint connectivity without writes')
        string(name: 'TICKER', defaultValue: 'SPY', description: 'Fund ticker symbol to process')
        choice(name: 'FILING_YEAR', choices: ['2026', '2025', '2024'], description: 'Target filing year')
        choice(name: 'FILING_QUARTER', choices: ['1', '2', '3', '4'], description: 'Target filing quarter')
        booleanParam(name: 'FORCE_DOCKER_NO_CACHE', defaultValue: false, description: 'Force Docker to rebuild layers from scratch')
    }

    environment {
        DOPPLER_TOKEN   = credentials('doppler-personal-token')
        DOPPLER_PROJECT = 'sec-pipeline'
        DOPPLER_CONFIG  = 'prd'
        SSH_CREDS_ID    = 'windows-laptop-ssh'
        IMAGE_BASE_NAME = 'fund-etl'
    }

    stages {
        stage('Fetch Secrets from Doppler Cloud') {
            steps {
                script {
                    def response = sh(
                        script: '''
                            curl -s --request GET \
                              --url "[https://api.doppler.com/v3/configs/config/secrets/download?project=$](https://api.doppler.com/v3/configs/config/secrets/download?project=$){DOPPLER_PROJECT}&config=${DOPPLER_CONFIG}&format=json" \
                              --header "accept: application/json" \
                              --header "authorization: Bearer ${DOPPLER_TOKEN}"
                        ''',
                        returnStdout: true
                    ).trim()

                    def secrets = readJSON text: response

                    env.WINDOWS_HOST   = secrets.WINDOWS_HOST
                    env.WINDOWS_USER   = secrets.WINDOWS_USER
                    env.DATABASE_URL   = secrets.DATABASE_URL
                    env.SEC_USER_AGENT = secrets.SEC_USER_AGENT

                    echo "Doppler secrets loaded successfully."
                }
            }
        }

        stage('Checkout & Sync Latest Git Code') {
            steps {
                checkout scm
                script {
                    sh '''
                        git fetch --all --prune
                        git checkout master
                        git reset --hard origin/master
                    '''
                    env.GIT_COMMIT_SHORT = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()
                    echo "Deploying commit: ${env.GIT_COMMIT_SHORT}"
                }
            }
        }

        stage('Stream & Build Docker Image on Windows') {
            steps {
                script {
                    def noCacheFlag = params.FORCE_DOCKER_NO_CACHE ? "--no-cache" : ""
                    withCredentials([sshUserPrivateKey(credentialsId: env.SSH_CREDS_ID, keyFileVariable: 'SSH_KEY')]) {
                        sh """
                            tar -czf - . | ssh -i "\$SSH_KEY" \
                                -o StrictHostKeyChecking=no \
                                -o ConnectTimeout=15 \
                                ${WINDOWS_USER}@${WINDOWS_HOST} \
                                "docker build ${noCacheFlag} \
                                    -t ${IMAGE_BASE_NAME}:latest \
                                    -t ${IMAGE_BASE_NAME}:${GIT_COMMIT_SHORT} -"
                        """
                    }
                }
            }
        }

        stage('Execute ETL (Dry Run / Scheduled)') {
            steps {
                script {
                    def baseArgs = "--year ${params.FILING_YEAR} --quarter ${params.FILING_QUARTER} --ticker ${params.TICKER}"
                    def runArgs  = params.DRY_RUN ? "${baseArgs} --dry-run" : baseArgs

                    withCredentials([sshUserPrivateKey(credentialsId: env.SSH_CREDS_ID, keyFileVariable: 'SSH_KEY')]) {
                        sh """
                            ssh -i "\$SSH_KEY" \
                                -o StrictHostKeyChecking=no \
                                -o ConnectTimeout=15 \
                                ${WINDOWS_USER}@${WINDOWS_HOST} \
                                "docker run --rm \
                                    --add-host=host.docker.internal:host-gateway \
                                    -e DATABASE_URL='${DATABASE_URL}' \
                                    -e SEC_USER_AGENT='${SEC_USER_AGENT}' \
                                    ${IMAGE_BASE_NAME}:${GIT_COMMIT_SHORT} ${runArgs}"
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success {
            echo "Pipeline run completed successfully for commit ${env.GIT_COMMIT_SHORT}."
        }
        failure {
            echo "Pipeline execution failed. Review logs above."
        }
    }
}
```

---

## Part 2: One-Time Setup: Docker Credential Shim

Run this once in PowerShell as Administrator:

1. Restore the original binary if it was renamed to .bak
```powershell
$credDesktopBak = "C:\Program Files\Docker\Docker\resources\bin\docker-credential-desktop.exe.bak"
$credDesktopOrig = "C:\Program Files\Docker\Docker\resources\bin\docker-credential-desktop.exe"
if (Test-Path $credDesktopBak) {
    Move-Item -Path $credDesktopBak -Destination $credDesktopOrig -Force
}
```

2. Create the dummy shim directory
```powershell
$helperDir = "C:\ProgramData\docker-cli-helpers"
if (-not (Test-Path $helperDir)) {
    New-Item -ItemType Directory -Path $helperDir -Force
}
```

3. Create the dummy batch script
```powershell
@'
@echo off
if "%1"=="get" (
    echo {}
    exit /b 0
)
exit /b 0
'@ | Set-Content -Path "$helperDir\docker-credential-desktop.bat" -Encoding ASCII
```

4. Prepend shim directory to System PATH
```powershell
$systemPath = [Environment]::GetEnvironmentVariable("Path", "Machine")
if ($systemPath -notmatch [regex]::Escape($helperDir)) {
    [Environment]::SetEnvironmentVariable("Path", "$helperDir;$systemPath", "Machine")
}
```

## Part 3: Cold-Start Operational Checklist

Follow this procedure when both machines (MacBook Pro and Windows host) have been powered on after a full shutdown.

### Step 1: Windows Host Initialization

Log into the Windows desktop, open **PowerShell as Administrator**, and execute:

1. Verify IP Address (must match Doppler's WINDOWS_HOST value, nominally 192.168.1.9)
```powershell 
    Get-NetIPAddress -AddressFamily IPv4 | Where-Object { $_.InterfaceAlias -notmatch "vEthernet|Loopback" } | Select-Object InterfaceAlias, IPAddress
```

2. Ensure connection profile is Private (allows inbound SSH port 22 through firewall)
```powershell
   $profile = Get-NetConnectionProfile | Select-Object -First 1
   if ($profile.NetworkCategory -ne "Private") {
      Set-NetConnectionProfile -InterfaceAlias $profile.InterfaceAlias -NetworkCategory Private
   } 
```

3. Ensure native PostgreSQL service is running
```powershell
    $pg = Get-Service -Name "*postgres*" -ErrorAction SilentlyContinue | Select-Object -First 1
    if ($pg -and $pg.Status -ne "Running") {
      Start-Service $pg.Name
    }
```


4. Ensure OpenSSH service is running
```powershell
   $sshd = Get-Service sshd -ErrorAction SilentlyContinue
   if ($sshd.Status -ne "Running") {
      Start-Service sshd
   } else {
      Restart-Service sshd
   }
```

5. Launch Docker Desktop
```powershell
   $dockerPath = "C:\Program Files\Docker\Docker\Docker Desktop.exe"
   if (Test-Path $dockerPath) {
      Start-Process $dockerPath
   } else {
      Write-Warning "Docker Desktop executable not found at $dockerPath"
   }
```

6. Verify Docker engine after icon turns solid green
```powershell
   docker version
```

### Step 2: macOS Controller Initialization

Log into your MacBook Pro and open **Terminal**:

```bash
# 1. Test TCP reachability to Windows OpenSSH port
nc -zv -w 3 192.168.1.9 22

# 2. Verify remote non-interactive Docker access over SSH
ssh shiva@192.168.1.9 "docker version"

# 3. Start Jenkins service
brew services start jenkins-lts

# 4. Check Jenkins port readiness
curl -I http://localhost:8080
```

### Step 3: Trigger the Pipeline

1. Open `http://localhost:8080` in your browser.
2. Select **`fund-etl-pipeline`** $\rightarrow$ **Build with Parameters**.
3. Configure run settings:
   * **`DRY_RUN`**: `true` (safe verification) or `false` (mutates database).
   * **`TICKER`**: `SPY` (or desired fund symbol).
   * **`FILING_YEAR`**: `2026`
   * **`FILING_QUARTER`**: `1`
   * **`FORCE_DOCKER_NO_CACHE`**: Check only if dependency locks or OS layers were revised.
4. Click **Build**.