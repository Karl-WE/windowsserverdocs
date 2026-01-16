---
title: Troubleshooting guide for VM Conversion Extension in Windows Admin Center
description: This article introduces a troubleshooting guide for VM Conversion issues.
ms.date: 1/6/2026
ms.topic: troubleshooting
author: pmiddha
ms.author: pmiddha
---

# Troubleshooting guide for VM Conversion Extension in Windows Admin Center

This article provides troubleshooting steps for common issues encountered when using the VM Conversion Extension to migrate VMware virtual machines to Hyper-V in Windows Admin Center.



>[!NOTE]
>For issues or questions not covered in this documentation, you can submit feedback [here](https://github.com/MicrosoftDocs/Windows-Admin-Center-Ideas-and-Feedback).

## Issue 1: VM resync/remigrate required, or migration stuck at a certain percentage (session timeout)

**Symptom:**

- User wants to resync or remigrate a VM.
- Migration is stuck at a certain percentage due to a session timeout.

**Resolution:**

1. Delete the relevant entries from the following files on the Windows Admin Center gateway machine:

   - `C:\Program Files\Windows Admin Center\Service\migrationStatus.json`
   - `C:\Program Files\Windows Admin Center\Service\syncStatus.json`

1. If the VM already exists in **Hyper-V Manager**, delete it before reinitiating the migration.

---

## Issue 2: Cancel VM synchronization or migration in progress

**Symptom:**

- User wants to cancel a synchronization or migration while it is in progress.

**Resolution:**
Cancellation isn't supported directly in the extension. As a workaround:

1. Stop the **Windows Admin Center service**.

1. Restart the service. This releases all running threads.

1. Delete the relevant entries from the following files to ensure status doesn't continue to show as "In Progress":

   - `C:\Program Files\Windows Admin Center\Service\migrationStatus.json`
   - `C:\Program Files\Windows Admin Center\Service\syncStatus.json`

---

## Issue 3: Migration precheck fails with error

**Error message:**

> *"Failed to retrieve the list of VMs from the destination server. Please ensure the destination server is reachable and retry the operation."*

**Resolution:**

- Ensure there are no **failed virtual machines** present on the same destination server.

---

## Issue 4: Static IP Configuration Is Not Preserved Without Guest Credentials

**Scenario:**

The VM Conversion tool can automatically migrate static IP settings for Windows virtual machines when guest operating system credentials are provided.

If the user does not provide guest credentials, the tool cannot migrate static IP settings automatically.

**Symptom:**

After migration, the virtual machine:
- Receives a DHCP-assigned IP address, or
- Does not retain its original static IP configuration

**Cause:**

Automatic static IP migration requires access to the guest operating system to read and reapply network settings.

When guest credentials are not provided due to security or compliance requirements, the VM Conversion tool cannot capture static IP configuration during migration.

**Resolution: Manual Static IP Migration (Guest Credentials Not Required)**

Use the following workflow to preserve static IP configuration without providing guest credentials.

> [!IMPORTANT]  
> Complete these steps **after synchronization and before starting migration**.  
> Running the script after migration does not preserve the static IP configuration.

1. Download the [**Static IP migration package (.zip)**](https://aka.ms/hci-migrate-static-ip-download).  
   The package includes scripts for **Windows and Linux** virtual machines.

2. After synchronization completes, copy the package to a location inside the **source guest virtual machine** and extract the contents.

3. Open **PowerShell as Administrator** inside the guest virtual machine.

4. Navigate to the extracted folder.

5. Run the following command to capture the static IP configuration:

   ```powershell
   .\Prepare-MigratedVM.ps1 -StaticIPMigration -Verbose
   ```

---

## Issue 5: Synchronization Fails Due to an Invalid Change ID

**Error Message:**

> The generated Change ID is invalid. Please verify the Change ID and try again.

**When This Issue Occurs:**

This issue occurs during **synchronization** when the VM Conversion extension cannot retrieve or validate the **Change ID** for one or more virtual disks on the source VMware virtual machine.

The VM Conversion extension relies on **VMware Changed Block Tracking (CBT)** to identify incremental disk changes. If the Change ID is missing, stale, or invalid, synchronization cannot continue.

**Possible Causes:**

This issue can occur for one or more of the following reasons:

- Changed Block Tracking (CBT) is disabled or not fully enabled on the source virtual machine.
- Snapshot-related issues, including:
  - Orphaned or stale snapshots
  - Snapshot chain corruption
- Unsupported disk configurations, such as:
  - Independent disks (persistent or non-persistent)
  - Physical mode RDMs
- Recent virtual machine operations, including:
  - Storage vMotion
  - Virtual disk resize
  - Virtual machine restore from backup
- CBT metadata corruption caused by host crashes or unexpected power events.

**What to Check:**

Before retrying synchronization, verify the following:

1. Ensure CBT is enabled on the source virtual machine:
   - `ctkEnabled` is set to `true` at the virtual machine level.
   - `ctkEnabled` is set to `true` for each virtual disk.
2. Verify that no active snapshots exist:
   - Remove or consolidate snapshots if required.
3. Validate disk configuration:
   - Ensure disks are not configured as **Independent**.
   - Verify RDMs use **Virtual Compatibility Mode**, if applicable.
4. Review recent virtual machine operations:
   - Check for recent storage migrations, disk changes, or restore operations.

**How to Fix the Issue:**

If synchronization fails due to an invalid Change ID, regenerate CBT metadata on the source VMware virtual machine using one of the following methods.

### Option 1: Regenerate CBT using PowerCLI

You can also reset CBT programmatically using **VMware PowerCLI**, which is useful for automation scenarios or environments where GUI access is limited.

#### Prerequisites

- VMware PowerCLI installed
- Network access to the vCenter Server
- Sufficient permissions to reconfigure virtual machines

#### PowerCLI script

```powershell
Connect-VIServer -Server "<vCenterServer>" -User "<Username>" -Password "<Password>" -Force

$vm = Get-VM "<VMName>" | Get-View
$vmConfigSpec = New-Object VMware.Vim.VirtualMachineConfigSpec

# Disable CBT
$vmConfigSpec.changeTrackingEnabled = $false
$vm.ReconfigVM($vmConfigSpec)

# Re-enable CBT
$vmConfigSpec.changeTrackingEnabled = $true
$vm.ReconfigVM($vmConfigSpec)

Disconnect-VIServer -Server "<vCenterServer>" -Force -Confirm:$false
```

### Option 2: Regenerate CBT using the vSphere Client (GUI)

1. Disable **Changed Block Tracking (CBT)** on the source virtual machine.
2. Power off the virtual machine.
3. Re-enable **Changed Block Tracking (CBT)**.
4. Power on the virtual machine.
5. Allow the system to generate a new Change ID.
6. Retry the synchronization operation.

> [!NOTE]  
> Power cycling the virtual machine is required to regenerate valid CBT metadata.

**Learn More**

For more information about CBT limitations and troubleshooting guidance, see the following Broadcom documentation:

- [Troubleshooting Changed Block Tracking (CBT) – Broadcom Knowledge Base](https://knowledge.broadcom.com/external/article/320557/changed-block-tracking-cbt-on-virtual-ma.html)
- [Troubleshooting Changed Block Tracking (CBT) – VDDK Programming Guide](https://techdocs.broadcom.com/us/en/vmware-cis/vsphere/vsphere-sdks-tools/8-0/virtual-disk-development-kit-programming-guide/error-codes-and-open-source/troubleshooting-changed-block-tracking-cbt.html)

**Best Practice**

Validate CBT health on the source virtual machine before starting synchronization, especially after snapshot operations or virtual machine configuration changes.