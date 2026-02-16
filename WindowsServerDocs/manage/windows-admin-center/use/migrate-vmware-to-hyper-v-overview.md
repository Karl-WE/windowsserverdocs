---
title: Overview of VMware to Hyper-V migration in Windows Admin Center (Preview)
description: Learn about the VM Conversion extension for Windows Admin Center, which enables migration of VMware virtual machines to Hyper-V with minimal downtime.
author: robinharwood
ms.topic: overview
ms.date: 08/13/2025
ms.author: roharwoo
ms.reviewer: shsathee,pmiddha
---
# Overview of VMware to Hyper-V migration in Windows Admin Center (Preview)

> [!IMPORTANT]
> The VM Conversion extension is currently in PREVIEW.
> This document provides information about a prerelease product that might change substantially before its release. Microsoft makes no warranties, expressed or implied, with respect to the information provided here.
>
> As a preview extension, the VM Conversion extension is governed by the [Windows Admin Center prerelease extension software license terms](/legal/windows-server/windows-admin-center/windows-pre-release-extension-eula).
> Microsoft isn't obligated under this agreement to provide any support services for the software. Issues, questions, and feedback not covered in this documentation can be filed [here](https://github.com/MicrosoftDocs/Windows-Admin-Center-Ideas-and-Feedback).

The **VM Conversion extension** for Windows Admin Center enables you to migrate VMware virtual machines from vCenter to Hyper-V. This lightweight tool provides online replication with minimal downtime for both Windows and Linux virtual machines. The extension is easy to set up and is available at no extra cost to customers.

## Key features

The VM Conversion extension provides the following capabilities:

- **Bulk migration**: Supports batch migration of up to 10 virtual machines at a time. This feature enables administrators to group virtual machines based on:

  - **Application dependency**: Virtual machines that are part of the same application stack or service.
  - **Cluster dependency**: Virtual machines that need to be distributed on nodes within the same cluster.
  - **Business boundaries**: Virtual machines servicing different businesses within a company, such as test machines and preproduction machines.
  - **Rack dependency**: Virtual machines running on hosts on a rack.

- **Cluster-aware migration**: Supports migration of virtual machines from ESXi hosts to Windows Server Failover clusters.

- **Static IP configurations**: Preserves static IP configurations of virtual machines from source to destination Hyper-V hosts. This functionality reduces post-migration tasks and enables seamless network continuity.

- **Secure Boot and UEFI template configuration**: Provides enhanced security and compliance.
  - Integrated OS type across the migration flow for accurate Secure Boot and Unified Extensible Firmware Interface (UEFI) setup.
  - Secure Boot settings are dynamically configured based on OS, either Windows or Linux.
  - Added error handling for unsupported OS types.

- **Localization support**: Improves user experience when using this tool in different languages.

- **Multiple vCenter connections**: Users can add multiple vCenter endpoints to switch between vCenters.

- **Multi-disk support**: Ensures all attached virtual disks are migrated and synchronized for virtual machines running complex workloads.

- **Prechecks**: Catches failures early in synchronization and migration phases.

- **Cleanup**: Removes VMware Tools from Windows VMs post-migration.

## How it works

The VM Conversion extension uses a synchronization and migration workflow to move virtual machines from VMware to Hyper-V with minimal downtime.

:::image type="content" source="media/migrate-vmware-to-hyper-v-overview/supported-scenario-topology.png" alt-text="Diagram showing the supported scenario topology for VM migration from VMware vCenter to Hyper-V through Windows Admin Center.":::

The conversion process consists of two main phases:

### Phase 1: Synchronization

The extension connects to your vCenter environment and performs an initial full copy of the virtual machine's disks. During this phase:

- A snapshot is created on the source VM to track changes.
- The virtual disk data is copied to the destination Hyper-V host.
- A Hyper-V Virtual Hard Disk (VHDX) file is created at the specified location.
- The source VM continues to run during synchronization.

### Phase 2: Migration

The extension uses Change Block Tracking (CBT) to identify and replicate only the changed blocks. This approach:

- Minimizes the amount of data transferred during the final migration.
- Reduces the cutover window significantly.
- Allows you to schedule the final migration at a convenient time.

When you're ready to complete the migration:

- A delta sync captures any remaining changes.
- The source VM is powered off.
- A final delta captures any remaining changes during cut over windows.
- The VM is imported into Hyper-V with the correct configuration.
- The migrated VM is ready to use on Hyper-V.

> [!NOTE]
> **Best practice:** Deploy the Windows Admin Center gateway in the same site as the ESXi and Hyper-V hosts for VM conversion. This deployment ensures minimal WAN traffic, lower latency, and a reliable migration experience.

## Scenarios

The VM Conversion extension supports the following migration scenarios:

### Datacenter modernization

Organizations looking to modernize their datacenter infrastructure can use the VM Conversion extension to migrate workloads from VMware to Hyper-V. This scenario is common for organizations that want to:

- Consolidate their virtualization platform to reduce licensing costs.
- Simplify management by using a single hypervisor technology.
- Prepare for hybrid cloud deployments with Azure integration.

### Test and development environments

Quickly migrate test and development environments from VMware to Hyper-V to validate workloads before production migration or to leverage Hyper-V-specific features.

### Disaster recovery planning

Use the VM Conversion extension as part of your disaster recovery strategy to create Hyper-V-based replicas of VMware workloads.

## Supported versions and operating systems

### Supported vCenter versions

The extension supports VMware vCenter versions 6.x, 7.x, and 8.x.

### Supported guest operating systems

The following operating systems can use the VM Conversion extension:

**Windows operating systems:**

- Windows Server 2025, 2022, 2022 Azure Edition, 2019, 2016, 2012 R2
- Windows 10

**Debian-based operating systems:**

- Ubuntu Linux
- Ubuntu 20.04, 24.04
- Debian 11, 12

**RHEL-based operating systems:**

- Alma Linux
- CentOS
- Red Hat Linux 9.0

> [!IMPORTANT]
> For Linux guests, you must install Hyper-V drivers before starting migration. The Hyper-V drivers are essential to ensure successful post-migration boot.

## Related content

- [Migrate VMware VMs to Hyper-V in Windows Admin Center](migrate-vmware-to-hyper-v-how-to.md)
- [Troubleshoot VM Conversion Extension](migrate-vmware-to-hyper-v-troubleshoot.md)
- [What's new in VM Conversion Extension](migrate-vmware-to-hyper-v-whats-new.md)
- [VMware to Hyper-V migration FAQ](migrate-vmware-to-hyper-v-faq.yml)
