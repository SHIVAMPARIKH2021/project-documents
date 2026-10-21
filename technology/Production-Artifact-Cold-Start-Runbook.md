## Part 1: One-Time Setup: Docker Credential Shim

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

5. Allow Client machine to connect to RabbitMQ UI
```powershell
New-NetFirewallRule -DisplayName "RabbitMQ (AMQP & Management)" `
    -Direction Inbound `
    -LocalPort 5672, 15672 `
    -Protocol TCP `
    -Action Allow 
```

## Part 2: Cold-Start Operational Checklist

Follow this procedure when both machines (MacBook Pro and Windows host) have been powered on after a full shutdown.

### Step 1: Windows Host Initialization

Log into the Windows desktop, open **PowerShell as Administrator**, and execute:

1. Verify IP Address (must match Doppler's WINDOWS_HOST value, if not ***change it in Doppler***)
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

1. Test TCP reachability to Windows OpenSSH port
```bash
nc -zv -w 3 <WINDOWS_HOST> 22
```

2. Verify remote non-interactive Docker access over SSH
```bash
ssh shiva@<WINDOWS_HOST> "docker version"
```

3. Start Jenkins service
```bash
brew services start jenkins-lts
```

4. Check Jenkins port readiness
```bash
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