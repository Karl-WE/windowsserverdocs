---
title: Migrate VMware VMs to Hyper-V in Windows Admin Center (Preview)
description: Learn how to install and use the VM Conversion extension to migrate VMware virtual machines to Hyper-V using Windows Admin Center.
author: pmiddha
ms.topic: how-to
ms.date: 08/13/2025
ms.author: pmiddha
ms.reviewer: shsathee,pmiddha
#customer intent: As a virtualization administrator, I want to migrate VMware virtual machines to Hyper-V so that I can consolidate my virtualization infrastructure.
---
# Migrate VMware VMs to Hyper-V in Windows Admin Center (Preview)

> [!IMPORTANT]
> The VM Conversion extension is currently in PREVIEW.
> This document provides information about a prerelease product that might change substantially before its release. Microsoft makes no warranties, expressed or implied, with respect to the information provided here.
>
> As a preview extension, the VM Conversion extension is governed by the [Windows Admin Center prerelease extension software license terms](/legal/windows-server/windows-admin-center/windows-pre-release-extension-eula).
> Microsoft isn't obligated under this agreement to provide any support services for the software. Issues, questions, and feedback not covered in this documentation can be filed [here](https://github.com/MicrosoftDocs/Windows-Admin-Center-Ideas-and-Feedback).

This article describes how to migrate VMware virtual machines to Hyper-V using the VM Conversion extension in Windows Admin Center. You learn how to install and configure the extension, synchronize virtual machines, and complete the migration.

For an overview of the VM Conversion extension, including key features and supported scenarios, see [Overview of VMware to Hyper-V migration](migrate-vmware-to-hyper-v-overview.md).

## Prerequisites

Before you begin, review the prerequisites and make sure your environment meets the requirements.

### Windows Admin Center Gateway prerequisites

- Install PowerCLI.

    - Open **PowerShell** as an administrator.

    - Run the following command to install the PowerCLI module:

    ```powershell
    Install-Module -Name VMware.PowerCLI
    ```

    - Verify that the module is installed:

    ```powershell
    Get-Module -Name VMware.PowerCLI -ListAvailable
    ```

    - Test the connection to a vCenter Server by running:

    ```powershell
    Connect-VIServer -Server "<vCenterServerFQDN_or_IP>" -User "<username>" -Password "<password>" -Force
    ```

    > [!NOTE]
    > Replace `<vCenterServerFQDN_or_IP>`, `<username>`, and `<password>` with your actual vCenter credentials.


- Install:
  - [Microsoft Visual C++ Redistributable](/cpp/windows/latest-supported-vc-redist)
  - [Visual C++ Redistributable Packages for Visual Studio 2013](https://www.microsoft.com/download/details.aspx?id=40784)

- Download [VMware Virtual Disk Development Kit (VDDK) version 8.0.3](https://developer.broadcom.com/sdks/vmware-virtual-disk-development-kit-vddk/latest/). Extract the contents, and copy to the directory: *'C:\Program Files\WindowsAdminCenter\Service\VDDK'*.

    > [!NOTE]
    > Ensure you download **VDDK version 8.0.3** specifically. Other versions aren't supported.

- Ensure that the Hyper-V role is installed. This setting is typically enabled by default.

- [Use Windows Admin Center Gateway V2](https://aka.ms/downloadWAC) – version `2410` build number `2.4.12.10`

For supported vCenter versions and guest operating systems, see [Overview of VMware to Hyper-V migration](migrate-vmware-to-hyper-v-overview.md#supported-versions-and-operating-systems).

## Install the VM Conversion (Preview) extension in Windows Admin Center

Complete the following steps to install the **VM Conversion** extension.

1. Open Windows Admin Center.

1. Select the **Settings** button in the top-right. In the left pane, select **Extensions**.

1. The **Available Extensions** tab lists the extensions on the feed that you can install.

1. Search for **VM Conversion (Preview)** in **Available extensions** and select **Install**.

    :::image type="content" source="media/migrate-vmware-to-hyper-v-overview/vm-conversion-available-extensions.png" alt-text="Screenshot of the VM Conversion (Preview) extension in the list of all Available extensions." lightbox="media/migrate-vmware-to-hyper-v-overview/vm-conversion-available-extensions.png":::

1. After installation, make sure the **VM Conversion** extension appears in Windows Admin Center under: **Installed Extensions** > **VM Conversion (Preview)**.

1. Navigate to the Windows Admin Center home page by selecting the **Windows Admin Center** logo in the top-left corner.

1. Select **+ Add**, then select **Server** or **Cluster** to add and connect to your Hyper-V host.

## Connect to vCenter

When you first visit the extension, you need to connect your vSphere client endpoint.

1. Select **Connect to vCenter**.

    :::image type="content" source="media/migrate-vmware-to-hyper-v-overview/connect-to-v-center.png" alt-text="Screenshot of the connect to vCenter option." lightbox="media/migrate-vmware-to-hyper-v-overview/connect-to-v-center.png":::

1. Enter the vCenter FQDN, vCenter username, and vCenter password.

    :::image type="content" source="media/migrate-vmware-to-hyper-v-overview/configure-vmware-settings.png" alt-text="Screenshot showing how to configure VMware settings." lightbox="media/migrate-vmware-to-hyper-v-overview/configure-vmware-settings.png":::

## Synchronize virtual machines using the VM Conversion (Preview) extension

### Synchronization prechecks

Before you synchronize virtual machines, make sure the following requirements are met. The extension runs these prechecks automatically and reports any issues.

- No active snapshots exist on the virtual machine.
- VMware PowerCLI is installed on the Windows Admin Center Gateway machine.
- Microsoft Visual C++ Redistributable packages (versions 2013 and latest) are installed on the Windows Admin Center Gateway machine.
- VDDK package is present at: `C:\Program Files\WindowsAdminCenter\Service\VDDK` on the Windows Admin Center Gateway machine.
- Target disk path for synchronization is valid.
- Destination Hyper-V host has sufficient memory and disk space.
- Change Block Tracking (CBT) is supported on the VM.

### Synchronize virtual machines

Complete the following steps to synchronize VMware virtual machines in Windows Admin Center.

1. Connect to the Hyper-V server in Windows Admin Center that you want the VM to be migrated.

1. Go to the VM Conversion extension in the left panel under **Extensions** > **VM Conversion (Preview)**.

1. In the virtual machine list, select up to 10 virtual machines to synchronize.

    :::image type="content" source="media/migrate-vmware-to-hyper-v-overview/bulk-vm-selection-for-synchronization.png" alt-text="Screenshot of the synchronize tab." lightbox="media/migrate-vmware-to-hyper-v-overview/bulk-vm-selection-for-synchronization.png":::

1. In the Synchronize VM window, enter in the **Path to store data**. Select **Synchronize**.

    :::image type="content" source="media/migrate-vmware-to-hyper-v-overview/synchronize-vm-path-selection.png" alt-text="Screenshot of the dialog to enter the path to store data and confirm the synchronization can start." lightbox="media/migrate-vmware-to-hyper-v-overview/synchronize-vm-path-selection.png":::

1. You see notifications appear with the progress for: running prechecks, preparing the environment, creating a snapshot, and finalizing synchronization. Confirm that the Hyper-V Virtual Hard Disk (VHDX) file is created in the folder path specified.

    :::image type="content" source="media/migrate-vmware-to-hyper-v-overview/synchronization-in-progress.png" alt-text="Screenshot of the notifications that appear while the synchronization is in progress." lightbox="media/migrate-vmware-to-hyper-v-overview/synchronization-in-progress.png":::

1. Wait for the sync to complete.

## Migrate virtual machines using the VM Conversion (Preview) extension

### Migration prechecks
Before migration begins, the extension automatically runs a set of prechecks. These prechecks verify:

- The destination Hyper-V host has enough available vCPUs.
- No virtual machine with the same name exists on the destination Hyper-V host.
- The Hyper-V role is enabled on the target Hyper-V host.
- The synchronized `.vhdx` file exists at the specified destination storage path on the Hyper-V host.
- The virtual machine has no active snapshots.

### Migrate virtual machines

Complete the following steps to migrate VMware virtual machines to Hyper-V in Windows Admin Center.

1. Go to the **Migrate** tab, and select the VM to migrate. Select **Migrate**.

    :::image type="content" source="media/migrate-vmware-to-hyper-v-overview/vm-selection-for-migration.png" alt-text="Screenshot of the migrate tab and virtual machines selected to migrate." lightbox="media/migrate-vmware-to-hyper-v-overview/vm-selection-for-migration.png":::

1. In the Migrate VM window, select **Proceed** to start the migration.

    :::image type="content" source="media/migrate-vmware-to-hyper-v-overview/confirm-migration.png" alt-text="Screenshot of the dialog confirming that the migration can start." lightbox="media/migrate-vmware-to-hyper-v-overview/confirm-migration.png":::

    During the migration, the process performs the following steps: runs migration prechecks, ensures sufficient disk space, performs delta replication, powers off source VM, executes final delta sync, and imports VM into Hyper-V.

1. Wait for virtual machine migration to complete.

    :::image type="content" source="media/migrate-vmware-to-hyper-v-overview/migration-in-progress.png" alt-text="Screenshot of the progress of virtual machine migration." lightbox="media/migrate-vmware-to-hyper-v-overview/migration-in-progress.png":::

1. You can manage the migrated virtual machine using the Hyper-V Manager or Windows Admin Center.

    > [!NOTE]
    > Migration requires the user to stay signed in with an active browser session. If the session is closed or times out, the
    > migration might pause or stop progressing.

## View logs
During synchronization and migration, you can view logs in the following locations to help troubleshoot any issues that arise:

- [Browser console logs](#browser-console-logs)
- [Event viewer logs](#event-viewer-logs)
- [VM conversion logs](#vm-conversion-logs)

### Browser console logs

1. Open your browser settings, and go to **More Tools** > **Developer Tools**.
1. Check the **Console** tab.
1. Look for any error or warning messages and share them as needed.

### Event viewer logs

1. On the Windows Admin Center server, open **Event Viewer**.
1. Expand **Applications and Services Logs** in the left pane.
1. Select **WindowsAdminCenter**.
1. Filter and review logs for **Errors**, **Warnings**, and **Informational** messages relevant to the VM Conversion extension.

### VM conversion logs

1. Connect to the Windows Admin Center server.
1. Find the file located at `C:\Program Files\WindowsAdminCenter\Service\VMConversion_log.txt`.

## Related content

- [Overview of VMware to Hyper-V migration](migrate-vmware-to-hyper-v-overview.md)
- [Troubleshoot VM Conversion Extension](troubleshoot-vm-conversion-extension.md)
- [What's new in VM Conversion Extension](migrate-vmware-to-hyper-v-whats-new.md)
