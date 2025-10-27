# Install Windows 11 on a Dell Laptop and Register with Intune Autopilot

This guide provides step-by-step instructions to create a bootable Windows 11 USB drive using the Media Creation Tool, install Windows 11 on a Dell laptop, and register the device with Microsoft Intune Autopilot during the Out-of-Box Experience (OOBE) by running a PowerShell script from the USB drive. The process uses your Microsoft account credentials to upload the device’s hardware hash to Intune, enabling automatic OOBE configuration via Autopilot.

## Prerequisites

Before beginning this guide, ensure the following prerequisites are met to successfully install Windows 11 and register the device with Microsoft Intune Autopilot:

- **Microsoft Entra ID (Azure AD) Tenant**: You must have an active Microsoft Entra ID tenant (formerly Azure AD). This is required for device registration and automatic enrollment during Autopilot provisioning.
- **Licenses**:
  - **Admin Account**: An Intune Administrator or Global Administrator account with Microsoft Entra ID Premium P1 (or higher) and Microsoft Intune licenses. This allows the admin to upload hardware hashes and configure Autopilot.
  - **User Licenses**: Each end-user who will enroll devices needs a Microsoft Entra ID Premium P1 (or higher) license for automatic MDM enrollment, plus an Intune license (often bundled in Microsoft 365 E3/E5 or Enterprise Mobility + Security plans). These are required for device join and management.
- **Intune and Autopilot Configuration**: Windows Autopilot must be enabled and configured in the Microsoft Intune admin center[](https://endpoint.microsoft.com/). This includes setting up automatic MDM enrollment (under Devices > Enroll devices > Automatic enrollment) and creating deployment profiles if needed.
- **Device Requirements**: The Dell laptop must meet Windows 11 hardware requirements (e.g., TPM 2.0, Secure Boot, compatible CPU) and have internet access during OOBE for hardware hash upload and provisioning.
- **Other Recommendations** (not required but helpful):
  - Microsoft 365 Apps for enterprise license for easy app deployment via Intune.
  - Windows Enterprise edition (via subscription activation) for advanced features like disabling consumer experiences.
  - Access to the Intune Connector for Active Directory if using hybrid Entra ID join (for on-premises AD integration).

Verify these in the Microsoft 365 admin center[](https://admin.microsoft.com/) under Billing > Licenses and the Intune admin center under Tenant administration > Roles.

## Step 1: Create a Bootable Windows 11 USB Drive

1. On a Windows PC, open a web browser and go to [https://www.microsoft.com/en-us/software-download/windows11](https://www.microsoft.com/en-us/software-download/windows11).
2. Under "Create Windows 11 Installation Media," click **Download Now** to download the Media Creation Tool (`MediaCreationTool.exe`).
3. Save the file to your computer and double-click it to run as administrator (right-click and select "Run as administrator").
4. Accept the license terms.
5. Select **Create installation media (USB flash drive, DVD, or ISO file) for another PC** and click **Next**.
6. Choose your language, Windows edition (e.g., Windows 11 Home or Pro), and architecture (64-bit). Click **Next**.
7. Select **USB flash drive** and click **Next**.
8. Insert a USB drive (at least 8 GB, will be erased) and select it from the list. Click **Next**.
9. The tool will download Windows 11 and create the bootable USB. This may take 30-60 minutes.
10. Once complete, safely eject the USB drive.

## Step 2: Add the PowerShell Script to the USB Drive

1. Re-insert the USB drive into your computer.
2. Open File Explorer and navigate to the root of the USB drive.
3. Ensure File Explorer shows file extensions:
   - In File Explorer, click **View** > **Show** > **File name extensions** (or go to **File** > **Options** > **View** and uncheck "Hide extensions for known file types," then click **OK**).
4. Right-click in the root of the USB drive, select **New** > **Text Document**, and name it `AutoPilotRegister.ps1`. Ensure the full name, including the `.ps1` extension, is entered, and press **Enter**.
5. If prompted with a warning: "If you change a file name extension, the file might become unusable. Are you sure you want to change it?" click **Yes** to confirm.
6. Verify the file icon changes to a PowerShell script icon (not a Notepad icon).
7. Open the file in a text editor (e.g., Notepad) by right-clicking it and selecting **Open with** > **Notepad**.
8. Paste the following PowerShell script:

   ```powershell
   # Set the execution policy to allow running scripts for the current process 
   Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned -Force
   
   # Install the NuGet package provider 
   Install-PackageProvider -Name NuGet -Force -ErrorAction SilentlyContinue
   
   # Install the Get-WindowsAutoPilotInfo script 
   Install-Script -Name Get-WindowsAutoPilotInfo -Force -ErrorAction SilentlyContinue
   
   # Run the Get-WindowsAutoPilotInfo script to register the device with Autopilot
   Get-WindowsAutoPilotInfo -Online
   ```

9. Save the file by clicking **File** > **Save** (or press **Ctrl + S**). Confirm in File Explorer that the file is named `AutoPilotRegister.ps1` (not `AutoPilotRegister.ps1.txt`).
10. Safely eject the USB drive.

## Step 3: Install Windows 11 and Enter OOBE

1. Insert the bootable USB drive into the Dell laptop.
2. Power on or restart the laptop and press **F12** repeatedly to access the boot menu.
3. Select the USB drive (e.g., "UEFI: USB") and press **Enter**.
4. On the Windows Setup screen, select your language, time format, and keyboard layout, then click **Next**.
5. Click **Install Now**.
6. Enter a product key if prompted or select **I don’t have a product key** to continue.
7. Accept the license terms and click **Next**.
8. Choose **Custom: Install Windows only (advanced)**.
9. Select the target drive (this will erase all data) and click **Next**.
10. The installation will proceed, restarting the laptop several times.
11. After installation, the laptop will enter the **Out-of-Box Experience (OOBE)**.
    - If the hardware hash of the laptop is **already registered with Autopilot**, the installation will skip the region selection screen and proceed automatically with the laptop provisioning process. **No need to continue further steps of this guide.**
    - If the device is **not yet registered with Autopilot** and you see the **region selection screen**, **proceed to Step 4.**

## Step 4: Run the PowerShell Script During OOBE

1. At the OOBE screen (e.g., region or language selection), press **Shift + F10** (or **Shift + Fn + F10** on some Dell laptops) to open a Command Prompt.
2. Make sure to click in the prompt to get focus on the window so you can input your commands.
3. Identify the USB drive letter (e.g., `D:`) by running:

   ```powershell
   Get-Volume
   ```

4. Execute the PowerShell script using the drive letter acquired in the previous step:

   ```powershell
   powershell.exe -ExecutionPolicy Bypass -File [drive letter]:\AutoPilotRegister.ps1
   ```

   Replace `[drive letter]` with the removable drive letter identified in Step 4, item 3 (e.g., `D:` if `Get-Volume` shows it as `D:`).

7. A pop-up will prompt for your Microsoft account credentials:
   - Enter your admin username (e.g., `admin@yourdomain.com`).
   - Enter your password.
   - Complete any Microsoft Authenticator challenge if multi-factor authentication is enabled.
8. The script will collect the hardware hash and upload it to Intune. You will see messages like:

   ```
   Waiting for 1 of 1 to be imported
   ```

9. Once complete, the script will display:

   ```
   1 devices imported successfully
   ```

## Step 5: Restart and Complete OOBE

1. In the PowerShell window, type:

   ```powershell
   Restart-Computer
   ```

2. Press **Enter** to restart the laptop.
3. Next time it will automatically walk through the OOBE process, applying your organization’s Autopilot configuration (e.g., joining Intune, installing apps, applying policies).

This completes the process. Your Dell laptop is now installed with Windows 11 and registered with Intune Autopilot, ready for automated setup.
