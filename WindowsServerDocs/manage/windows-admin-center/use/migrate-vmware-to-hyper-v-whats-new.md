---

title: What's new in VM Conversion Extension for Windows Admin Center
description: Learn about the features and enhancements in VM Conversion that help to improve security, performance, and flexibility.
author: pmiddha
ms.author: pmiddha
ms.topic: release-notes
ms.date: 01/16/2026

---

# What's new in VM Conversion Extension for Windows Admin Center

This article provides release notes for the VM Conversion Extension in Windows Admin Center. Use these notes to learn about new features, enhancements, and bug fixes in each version.

The VM Conversion Extension helps you migrate VMware virtual machines to Hyper-V. Each release includes improvements to security, performance, reliability, and user experience.

---

## [Version 1.8.6](https://dev.azure.com/WindowsAdminCenter/Windows%20Admin%20Center%20Feed/_artifacts/feed/wac-public-extensions/NuGet/msft.sme.vm-conversion/overview/1.8.6) (January 2026)

## Improvements

- Improved pre-check validation by ensuring Change Block Tracking (CBT) is enabled before synchronization readiness checks.

- Enhanced VM creation to correctly refresh and apply CPU, memory, and operating system settings when values were incomplete or invalid.

## Bug fixes - Version 1.8.6

- Fixed an issue that could block migrations when required change tracking information was missing.

- Resolved a synchronization issue that could occur when VMware credentials contained a single quote (').

- Fixed an intermittent issue where the UI incorrectly reported that PowerCLI was not installed when it was already available.

---

## [Version 1.8.5](https://dev.azure.com/WindowsAdminCenter/Windows%20Admin%20Center%20Feed/_artifacts/feed/wac-public-extensions/NuGet/msft.sme.vm-conversion/overview/1.8.5) (December 2025)

## New Features & Enhancements

### Secure Boot Configuration Reliability
- Implemented logic to power off the VM automatically before applying Secure Boot settings.
- Resolves failures that occurred when Secure Boot was configured on a running VM.

### Early Change ID Validation
- Added prevalidation for missing disk Change IDs.
- Provides clear and early error messaging, avoiding unexpected failures later in the workflow.

### Power State Alignment
- Ensures the destination VMs power state consistently matches the source VM’s final power state after migration.
    - If the source VM is off and migration succeeds → destination VM remains off.
    - If the source VM is off and migration fails → source VM remains off.

### Enhanced Synchronization Experience
- Introduced asynchronous file-path validation in the Synchronization Confirmation dialog.
- Reduces UI blocking and improves responsiveness during sync initiation.

### Telemetry Improvements
- Added additional telemetry signals to improve:
    - Performance analysis
    - Workflow reliability tracking
    - Troubleshooting efficiency

### Security Improvements
- Implemented log sanitization in the PowerShell layer to mask sensitive data.
- Ensures secure handling of credentials across logs and event traces.

### VM List Component Update
- Reduced the VM synchronization and migration limit from 50 to 10 to improve reliability.
- Updated corresponding error and guidance messages to reflect the new threshold.

## Bug fixes - Version 1.8.5

- Fixed an issue where powering on a VM resulted in the error: 'Validation failed for one or more fields.'
- Resolved an issue causing: 'Failed to create destination VM: cpuCount must be a positive number.'
- Addressed a problem where closing the browser mid-migration caused workflows to appear stuck at 80% when returning to Windows Admin Center. The Import VM step now automatically resumes and completes correctly.

---

## [Version 1.8.3](https://dev.azure.com/WindowsAdminCenter/Windows%20Admin%20Center%20Feed/_artifacts/feed/wac-public-extensions/NuGet/msft.sme.vm-conversion/overview/1.8.3) (October 2025)

## New features and enhancements

### PowerCLI installation support
- Added a **PowerCLI installation option** on the landing page to help users set up required component on the gateway.  
- Introduced an **alert banner** on the VM List and vSphere List pages that notifies users if PowerCLI is missing, with a direct link to install it.

### Migration workflow improvements
- Improved stability, validation, and error handling for a smoother migration experience.

## Bug fixes - Version 1.8.4
- Fixed an issue where the **Submit** button in the vCenter credentials dialog could remain disabled after a failure.  
- Resolved a problem where migrations could get stuck at **80% progress**.  

---

## [Version 1.8.2](https://dev.azure.com/WindowsAdminCenter/Windows%20Admin%20Center%20Feed/_artifacts/feed/wac-public-extensions/NuGet/msft.sme.vm-conversion/overview/1.8.2) (October 2025)

### New Features

- **vCenter Version Display:**  
  You can now view the vCenter version directly on the **vCenter List** page for easier identification and management.

- **Migration Reconnection Banner:**  
  A new banner now appears, prompting users to stay signed in and refresh their session every 2 hours during migration to ensure continuity.

- **Quick Access to Documentation:**  
  The **“Open in New Window”** icon on the landing page now links directly to the official guide—
  [Migrate VMware Virtual Machines to Hyper-V in Windows Admin Center (Preview)](../use/migrate-vmware-to-hyper-v-overview.md).

### Other Improvements

- Enhanced telemetry for improved diagnostics and secure handling of environment information.

---

## [Version 1.8.0](https://dev.azure.com/WindowsAdminCenter/Windows%20Admin%20Center%20Feed/_artifacts/feed/wac-public-extensions/NuGet/msft.sme.vm-conversion/overview/1.8.0)  (September 2025)

### New Features

- **Bulk VM Migration with Queuing Support**

    To migrate multiple VMs, select up to **10 virtual machines per operation**. Queuing improves performance and stability during large-scale migrations.  

    >[!NOTE]  
    >Ensure you remain signed in to Windows Admin Center -> VM Conversion Extension -> vCenter, and refresh your session every 2 hours.  
    >The browser session must remain active during the final migration step.  

- **Static IP Batch Support**

    Bulk migration now supports **static IP migration** for both **Windows** and **Linux** Virtual Machines. This feature automates network configuration, reducing manual reconfiguration after migration.

- **Batch Uninstall of VMware Tools (Windows VMs)** 

    You can now uninstall VMware Tools from multiple Windows virtual machines in a single batch operation prior to migration, streamlining the preparation process.

    >[!NOTE]
    >While batch uninstall for Windows VMs is supported, Linux VMs still require manual removal.

- **BIOS UUID Migration**

    The migration process now preserves the BIOS UUID from the source VM, ensuring improved compatibility and identity synchronization on Hyper-V.

    >[!NOTE]
    >Only the BIOS UUID is migrated. BIOS Serial Number format differs between VMware and Hyper-V, which can affect licensing checks. For more information, see [FAQ](migrate-vmware-to-hyper-v-overview.md#frequently-asked-questions).

- **Standardized Destination Folder Structure**

    The destination VM folder structure now follows **Hyper-V conventions**. The Synchronization Confirmation dialog displays the folder path, helping administrators verify and predict destination locations.

- **Thick and Thin Disk Provisioning**

    During synchronization, VM disks are created as **thick (fixed)** or **thin (dynamic)** to match the **source VM’s configuration**, optimizing storage use, and simplifying post-migration management.

### Bug fixes - Version 1.8.0

- Resolved migration error: *Physical network adapter 'Ethernet' not found*.  
- Corrected VM listing issue where VMs already present in **Hyper-V Manager** were incorrectly marked as failed.  
- Improved notification accuracy during migration progress.  
- Enhanced prechecks for PowerCLI installation to catch failures early and provide clearer troubleshooting guidance.

### User Experience Changes

- **Session Persistence for Bulk Migration**

  - Stay logged in to Windows Admin Center -> VM Conversion Extension -> vCenter and keep your browser session active throughout migration.

  - The browser session must remain active during the final migration step.

- **Folder Structure Transparency**  

  - Destination folders now directly reflect **Hyper-V layout**.  
 
  -  - The Synchronization Confirmation dialog explicitly shows the destination path.

- **Linux VMs** - Install Hyper-V drivers on the guest OS before migration.  

- **Windows VMs** - VMware Tools batch uninstall is supported only for Windows VMs.  

- **Licensing Note** - Differences in BIOS Serial Number may affect licensing. See [FAQ](migrate-vmware-to-hyper-v-overview.md#frequently-asked-questions) for details.  