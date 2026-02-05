---
title: Migrate VMware Virtual Machines to Hyper-V in Windows Admin Center (Preview)
description: Learn how to migrate VMware virtual machines (VMs) to Hyper-V using the Windows Admin Center VM Conversion extension. Discover step-by-step instructions and benefits.
author: robinharwood
ms.topic: how-to
ms.date: 08/13/2025
ms.author: roharwoo
---
# Migrate VMware virtual machines to Hyper-V in Windows Admin Center (Preview)

> [!IMPORTANT]
> The VM Conversion extension is currently in PREVIEW.
> This document provides information about a prerelease product that might change substantially before its release. Microsoft makes no warranties, expressed or implied, with respect to the information provided here.

> [!IMPORTANT]
> As a preview extension, the VM Conversion extension is governed by the [Windows Admin Center prerelease extension software license terms](/legal/windows-server/windows-admin-center/windows-pre-release-extension-eula).
> Microsoft isn't obligated under this agreement to provide any support services for the software. Issues, questions, and feedback not covered in this documentation can be filed [here](https://github.com/MicrosoftDocs/Windows-Admin-Center-Ideas-and-Feedback).

You can use Windows Admin Center to migrate VMware virtual machines from vCenter to Hyper-V with the **VM Conversion extension**. This lightweight tool enables online replication with minimal downtime for both Windows and Linux virtual machines. The conversion tool is easy and fast to set up, at no cost to customers.

In this article, you learn how to install and configure the extension, follow the synchronization and migration workflow, and find answers to common questions.

## Feature overview

The **VM Conversion extension** provides the following key features:

- **Bulk migration**: Supports a batch of 10 virtual machines migration at-a-time. This feature enables admins to group virtual machines based on:

  - **Application dependency** – virtual machines that are part of the same application stack or service.
  - **Cluster dependency** – virtual machines that need to be distributed on nodes within same cluster.
  - **Business boundaries** – virtual machines servicing different business within a company. For example, test machines and preproduction machines.
  - **Rack dependency** – virtual machines running on hosts on a rack.

- **Cluster-aware migration**: Supports migration virtual machines from eSXI hosts to Windows Server Failover clusters.

- **Static IP configurations**: Persists the static IP configurations of virtual machines from source to destination Hyper-V hosts. This functionality reduces post-migration tasks and enables seamless network continuity.

- **Secure Boot and UEFI template configuration**: Provides enhanced security and compliance.
  - Integrated osType across the migration flow for accurate secure boot and Unified Extensible Firmware Interface (UEFI) setup.
  - Secure boot settings are dynamically configured based on OS, either Windows or Linux.
  - Added error handling for unsupported OS types.

- **Localization support**: Improves user experience using this tool in different languages.

- **Multiple vCenter connections**: Users can add multiple vCenter endpoints in order to switch between vCenters.

- **Multi-disk support**: Ensures all attached virtual disks are migrated and synchronized for virtual machines running complex workloads.​

- **Prechecks**: Catches failures early in replication, and migration phases.

- **Cleanup**: Removes VMware Tools from Windows VMs post-migration.

:::image type="content" source="media/migrate-vmware-to-hyper-v-overview/supported-scenario-topology.png" alt-text="Diagram showing the supported scenario topology for VM migration from VMware vCenter to Hyper-V through Windows Admin Center.":::

> [!Note]
> **Best practice:** Deploy the Windows Admin Center gateway in the same site as the ESXi and Hyper-V hosts for VM conversion. This ensures minimal WAN traffic, lower latency, and a reliable migration experience.

## Prerequisites

Before you begin, review the prerequisites and ensure your environment meets the requirements.

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

### vCenter or ESXi host prerequisites

None.

### Windows Server host prerequisites

None.

### Supported vCenter versions

- The extension supports VMware VCenter version 6.x or 7.x or 8.x.
- To connect to vCenter from the extension, keep the following vCenter information at hand:
  - Fully Qualified Domain Name (FQDN)
  - Username
  - Password

### Supported guest operating systems

The following operating systems can use the VM Conversion extension:

Windows operating systems:

- Windows Server 2025, 2022, 2022 Azure Edition, 2019, 2016, 2012 R2
- Windows 10

Debian-based operating systems:

- Ubuntu Linux
- Ubuntu 20.04, 24.04
- Debian 11, 12

RHEL-based operating systems:

- Alma Linux
- CentOS
- Red Hat Linux 9.0

For Linux guests, Hyper-V drivers must be installed before initiating migration. The Hyper-V drivers are essential to ensure successful post-migration boot.

---
## Install the VM Conversion (Preview) extension in Windows Admin Center

Complete the following steps to install the **VM Conversion** extension.

1. Open Windows Admin Center.

1. Select the **Settings** button in the top-right. In the left pane, select **Extensions**.

1. The Available Extensions tab lists the extensions on the feed that are available for installation.

1. Search for **VM Conversion (Preview)** in **Available extensions** and select **Install.**

    :::image type="content" source="media/migrate-vmware-to-hyper-v-overview/vm-conversion-available-extensions.png" alt-text="Screenshot of the VM Conversion (Preview) extension in the list of all Available extensions." lightbox="media/migrate-vmware-to-hyper-v-overview/vm-conversion-available-extensions.png":::

1. Once installed, ensure VM Conversion extension is visible in the Windows Admin Center under: **Extensions** > **VM Conversion (Preview)**.

## Connect to vCenter

When you first visit the extension, you need to connect your vSphere client endpoint.

1. Select **Connect to vCenter**.

    :::image type="content" source="media/migrate-vmware-to-hyper-v-overview/connect-to-v-center.png" alt-text="Screenshot of the connect to vCenter option." lightbox="media/migrate-vmware-to-hyper-v-overview/connect-to-v-center.png":::

1. Enter the vCenter FQDN, vCenter username, and vCenter password.

    :::image type="content" source="media/migrate-vmware-to-hyper-v-overview/configure-vmware-settings.png" alt-text="Screenshot showing how to configure VMware settings." lightbox="media/migrate-vmware-to-hyper-v-overview/configure-vmware-settings.png":::

## Synchronize virtual machines using the VM Conversion (Preview) extension

### Synchronization prechecks

A set of prechecks are run before the synchronization begins. Confirm you have all the following steps complete before continuing on with the steps to synchronize virtual machines:

1. No active snapshots exist on the virtual machine.

1. VMware PowerCLI is installed on the Windows Admin Center Gateway machine.

1. Microsoft Visual C++ Redistributable packages (versions 2013 and latest) are installed on the Windows Admin Center Gateway machine.

1. VDDK package is present at: `C:\Program Files\WindowsAdminCenter\Service\VDDK` on the Windows Admin Center Gateway machine.

1. Target disk path for synchronization is valid.

1. Destination Hyper-V host has sufficient memory and disk space.

1. Change Block Tracking (CBT) is supported on the VM.

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

A set of prechecks are run before the migration begins. Confirm you have all the following steps complete before continuing on with the steps to migrate virtual machines:

1. Destination Hyper-V host has sufficient vCPU availability.

1. No existing virtual machine with the same name on the destination Hyper-V host.

1. Hyper-V role is enabled on the target Hyper-V host.

1. Synchronized `.vhdx` file exists.

1. No active snapshots on the virtual machine.

### Migrate virtual machines

Complete the following steps to migrate VMware virtual machines to Hyper-V in Windows Admin Center.

1. Go to the **Migrate** tab, and select the VM to migrate. Select **Migrate**.

    :::image type="content" source="media/migrate-vmware-to-hyper-v-overview/vm-selection-for-migration.png" alt-text="Screenshot of the migrate tab and virtual machines selected to migrate." lightbox="media/migrate-vmware-to-hyper-v-overview/vm-selection-for-migration.png":::

1. In the Migrate VM window, select **Proceed** to start the migration.

    :::image type="content" source="media/migrate-vmware-to-hyper-v-overview/confirm-migration.png" alt-text="Screenshot of the dialog confirming that the migration can start." lightbox="media/migrate-vmware-to-hyper-v-overview/confirm-migration.png":::

    During the migration, the following steps are performed: run migration prechecks, ensure sufficient disk space, perform delta replication, power off source VM, execute final delta sync, and import VM into Hyper-V.

1. Wait for virtual machine migration to complete.

    :::image type="content" source="media/migrate-vmware-to-hyper-v-overview/migration-in-progress.png" alt-text="Screenshot of the progress of virtual machine migration." lightbox="media/migrate-vmware-to-hyper-v-overview/migration-in-progress.png":::

1. The migrated virtual machine can be managed using the Hyper-V Manager, or in Windows Admin Center.

    > [!NOTE]
    > Migration requires the user to stay signed in with an active browser session. If the session is closed or times out, the
    > migration may pause or stop progressing.

---

## View logs

### Browser console logs

1. Open your browser settings, and navigate to **More Tools** > **Developer Tools**.
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
