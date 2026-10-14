# OpenSSH Server Installation Troubleshooting Log

## Session Overview
This log documents the troubleshooting process for installing the OpenSSH Server capability on Windows using PowerShell, moving from a hung automated process to a manual deployment strategy.

---

### [Prompt 1]
> How much time these commands take to complete Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0 Start-Service sshd Set-Service -Name sshd -StartupType Automatic

### [Response 1]
In total, running these three commands typically takes **1 to 3 minutes** to complete. 

Here is the breakdown of the execution time for each specific command:

| Command | Expected Duration | What It Does |
| :--- | :--- | :--- |
| `Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0` | **45 seconds to 2.5 minutes** | This takes the vast majority of the time. Windows contacts Microsoft Update online to download and install the OpenSSH binaries, configure the firewall rules, and register the system services. |
| `Start-Service sshd` | **2 to 5 seconds** | Instantly initializes and starts the SSH background daemon (`sshd`). |
| `Set-Service -Name sshd -StartupType Automatic` | **Under 1 second** | Instantly updates the Windows Service configuration registry so it automatically launches on subsequent system reboots. |

#### Factors that can alter this timeline:
* **Internet Speed:** Since the `-Online` parameter downloads the capability package from Microsoft servers, a slow network connection will significantly lengthen the first command's duration.
* **System Performance:** Older mechanical hard drives (HDDs) or highly utilized CPUs will slow down the package decompression and installation phase. 

---

### [Prompt 2]
> I have run Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0 and from 5 minutes it is showin this
> Windows PowerShell
> Copyright (C) Microsoft Corporation. All rights reserved.
>                                                                                                                          Operation                                                                                                                  Running                                                                                                                 [oooooooooo                                                                                                  ]

### [Response 2]
If it has been stuck on that exact progress bar for over **5 minutes**, the installation has likely **hanged or timed out** in the background. 

This freeze usually happens because Windows Update is stuck processing a Cumulative Update in the background, or your network is blocked from reaching Microsoft's Features on Demand (FoD) servers. 

Follow these steps to safely handle it and fix the issue:

#### Step 1: Check if it actually finished or failed
Sometimes the PowerShell progress bar freezes visually even if the background process has completed or errored out. 
1. Open a **new, separate PowerShell window as Administrator**.
2. Run this status verification command:
   ```powershell
   Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH.Server*'
   ```
3. Check the **State** in the output:
   * If it says **`Installed`**, the installation succeeded despite the frozen progress bar. You can safely close the stuck window and proceed with `Start-Service sshd`.
   * If it says **`NotPresent`**, it is completely stuck. Close the frozen window and move to Step 2.

#### Step 2: Unfreeze and bypass the loop
If it is completely stuck at `NotPresent`, you need to restart the Windows servicing engine and try an alternative method:

1. Close your frozen PowerShell window. 
2. Open a fresh PowerShell window as Administrator and restart the Windows Update service to clear the stuck queue:
   ```powershell
   Restart-Service wuauserv
   ```
3. Try installing it via the graphical interface instead, which handles network timeouts better:
   * Press **Windows Key + i** to open **Settings**.
   * Navigate to **Apps** > **Optional features** (or **System** > **Optional features** depending on your Windows version).
   * Click **View features** or **Add a feature**.
   * Search for **OpenSSH Server**, check the box, and click **Install**.

---

### [Prompt 3]
> PS C:\WINDOWS\system32> Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH.Server*'
> 
> 
> Name  : OpenSSH.Server~~~~0.0.1.0
> State : NotPresent

### [Response 3]
Since the state is **`NotPresent`**, the installation has failed and is permanently stuck. 

The most common reason for this error on `C:\WINDOWS\system32` is that your machine is configured to look for updates through a local corporate server (WSUS or SCCM) rather than the official Microsoft Update servers, which blocks optional features from downloading.

Follow these steps to bypass this block and force the installation to finish successfully:

#### Step 1: Kill the stuck deployment process
Before trying again, you must stop the frozen installation task.
1. Close all your current PowerShell windows.
2. Open a **new** PowerShell window as **Administrator**.
3. Run this command to restart the Windows Update engine:
   ```powershell
   Restart-Service wuauserv
   ```

#### Step 2: Temporarily bypass WSUS/Group Policy
You can tell Windows to bypass local update policies and fetch the OpenSSH package directly from Microsoft's public servers.
1. Run this command to temporarily alter your registry (it tells Windows to check the internet for features):
   ```powershell
   Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" -Name "UseWUServer" -Value 0
   ```
2. Restart the Windows Update engine to apply the temporary tweak:
   ```powershell
   Restart-Service wuauserv
   ```
3. Run the installation command again:
   ```powershell
   Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
   ```
   *(It should now complete successfully within 1–2 minutes).*

#### Step 3: Re-enable your original policy settings
Once the installation finishes, you must change the registry back to its original value so your regular system updates continue to work correctly:
```powershell
Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" -Name "UseWUServer" -Value 1
Restart-Service wuauserv
```

---

### [Prompt 4]
> PS C:\WINDOWS\system32> Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" -Name "UseWUServer" -Value 0
> Set-ItemProperty : Cannot find path 'HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU' because it does not
> exist.
> At line:1 char:1
> + Set-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\Win ...
> + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>     + CategoryInfo          : ObjectNotFound: (HKLM:\SOFTWARE\...indowsUpdate\AU:String) [Set-ItemProperty], ItemNotFo
>    undException
>     + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.SetItemPropertyCommand

### [Response 4]
The error **`PathNotFound`** means your machine is not managed by a corporate WSUS server, which is actually good news. It confirms that the registry path doesn't exist because your computer natively points to the public internet for updates.

Since the pathway is clear but the command is still freezing, your local Windows Update agent components are corrupted, or a network firewall is silently dropping the connection to Microsoft's servers. 

Instead of fighting the broken Windows Update engine, you can bypass it entirely by using **Deployment Image Servicing and Management (DISM)** or an official manual installation. 

Try **Option 1** first, as it is the fastest fix.

#### Option 1: Force an installation via DISM
DISM uses a different method to pull down features and often succeeds when PowerShell's `Add-WindowsCapability` fails.

1. Open PowerShell as **Administrator**.
2. Run this command:
   ```powershell
   DISM /Online /Add-Capability /CapabilityName:OpenSSH.Server~~~~0.0.1.0
   ```
3. Wait up to 3 minutes. If it succeeds, it will say *"The operation completed successfully."*

#### Option 2: The manual GitHub installation (Guaranteed to work)
If DISM also freezes or fails, you can manually download the official, Microsoft-supported OpenSSH package directly from their GitHub repository.

1. Download the latest 64-bit release zip file (e.g., `OpenSSH-Win64.zip`) from the official Microsoft repository:  
   👉 **[Official Microsoft OpenSSH Release Page](https://github.com/PowerShell/Win32-OpenSSH/releases)**
2. Extract the folder and rename it to `OpenSSH`.
3. Move the folder to your Program Files directory so it sits exactly at:  
   `C:\Program Files\OpenSSH`
4. Open PowerShell as **Administrator**, navigate into that folder:
   ```powershell
   cd "C:\Program Files\OpenSSH"
   ```
5. Run the built-in installation script:
   ```powershell
   .\install-sshd.ps1
   ```
6. Open the firewall port for SSH by running this command:
   ```powershell
   New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
   ```

Once either option finishes, you can proceed with your original commands to start the service:
```powershell
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
```

---

### [Prompt 5]
> Give me repository link for Download the latest 64-bit release zip file (e.g., OpenSSH-Win64.zip) from the official Microsoft repository:
> 👉 Official Microsoft OpenSSH Release Page

### [Response 5]
You can access the direct link here: 

👉 **[PowerShell Win32-OpenSSH Releases Page](https://github.com/powershell/win32-openssh/releases)**

Scroll down to the **Assets** section of the latest release (or preview) to download the **`OpenSSH-Win64.zip`** package.

## Additional Help
```text
Prompt-1
Give me command to run install-sshd.ps1 on powershell under C:\Program Files\OpenSSH\OpenSSH-Win64

Response-1
Use an elevated PowerShell window and run the script with a quoted path: & "C:\Program Files\OpenSSH\OpenSSH-Win64\install-sshd.ps1".

📌 Exact command
Run PowerShell as Administrator, then execute:
Command:& "C:\Program Files\OpenSSH\OpenSSH-Win64\install-sshd.ps1"

🔐 If script execution is blocked
PowerShell may prevent running scripts. Temporarily allow it:
Command:Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process

Then run the script again:
Command:& "C:\Program Files\OpenSSH\OpenSSH-Win64\install-sshd.ps1"

🧩 Why the & is required
& is the call operator in PowerShell.
It is required when running a script whose path contains spaces (like Program Files).

✅ Alternative: change directory first
Command:cd "C:\Program Files\OpenSSH\OpenSSH-Win64"
Command:.\install-sshd.ps1

✅ After installation (optional)
Start the SSH services:
Command:Start-Service sshd
Command:Start-Service ssh-agent

Prompt-2
I have opened power shell as administrator at C:\Program Files\OpenSSH\OpenSSH-Win64

Response-2
Open PowerShell as Administrator, navigate to the OpenSSH folder, optionally adjust execution policy, and run the script using .\install-sshd.ps1.
Below is the complete, step‑by‑step method (safe and correct for Windows 10/11).

🟦 1. Open PowerShell as Administrator
Running scripts in Program Files requires elevated rights.
Press Start
Type PowerShell
Right‑click Windows PowerShell → Run as administrator
Confirm the UAC prompt
You should now see Administrator: Windows PowerShell in the title bar.

🟦 2. Navigate to the OpenSSH installation directory
Because the path contains spaces, you must use quotes:
Command:cd "C:\Program Files\OpenSSH\OpenSSH-Win64"

To verify you are in the correct folder:
Command:Get-ChildItem
You should see install-sshd.ps1 listed.

🟦 3. Allow script execution (if blocked)
Windows may prevent running .ps1 files depending on your execution policy.
Check your current policy:
Command:Get-ExecutionPolicy

If it returns Restricted, temporarily allow script execution:
Command:Set-ExecutionPolicy -Scope Process RemoteSigned

Scope Process means the change applies only to this PowerShell window
It resets automatically when you close the window
This is the safest method

🟦 4. Run the script
Use the dot‑slash prefix to run a script in the current directory:
Command:.\install-sshd.ps1

If the script requires administrative privileges, it will now run correctly because the session is already elevated.

🟦 5. Confirm the SSH service is installed
After the script completes, check the service:
Command:Get-Service sshd
If installed, you can start it:
Command:Start-Service sshd
And optionally set it to start automatically:
Command:Set-Service -Name sshd -StartupType Automatic

🟦 Common Problems & Fixes
❗ “File cannot be loaded because running scripts is disabled”
→ Use the temporary execution policy fix shown above.
❗ “Access is denied”
→ You are not running PowerShell as Administrator. Reopen it with elevated rights.
❗ “The term ‘install-sshd.ps1’ is not recognized”
→ You are not in the correct directory. Run:
Command:cd "C:\Program Files\OpenSSH\OpenSSH-Win64"
```