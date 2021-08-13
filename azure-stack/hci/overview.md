---
title: Azure Stack HCI solution overview
description: Azure Stack HCI is a hyperconverged infrastructure (HCI) cluster solution that hosts virtualized Windows and Linux workloads and their storage in a hybrid on-premises environment. Azure hybrid services enhance the cluster with capabilities such as cloud-based monitoring, Site Recovery, and VM backups, as well as a central view of all of your Azure Stack HCI deployments in the Azure portal.
ms.topic: overview
author: khdownie
ms.author: v-kedow
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 07/08/2021
ms.custom: e2e-hybrid
---

# Azure Stack HCI solution overview

> Applies to: Azure Stack HCI, version 20H2

Azure Stack HCI is a hyperconverged infrastructure (HCI) cluster solution that hosts virtualized Windows and Linux workloads and their storage in a hybrid environment that combines on-premises infrastructure with cloud services. Azure hybrid services enhance the cluster with capabilities such as cloud-based monitoring, Site Recovery, and virtual machine (VM) backups, as well as a central view of all of your Azure Stack HCI deployments in the Azure portal. You can manage the cluster with your existing tools including Windows Admin Center and PowerShell.

   > [!NOTE]
   > Azure Stack HCI is intended as a virtualization host, so most apps and server roles must run inside of VMs. Exceptions include Hyper-V, Network Controller, and other components required for Software Defined Networking (SDN) or for the management and health of hosted VMs.

Azure Stack HCI, version 20H2 is now [available for download](https://azure.microsoft.com/products/azure-stack/hci/hci-download/). You can either [purchase validated servers](https://hcicatalog.azurewebsites.net) from a Microsoft hardware partner with the Azure Stack HCI operating system pre-installed, or buy validated bare metal servers from an OEM and install the operating system yourself.

Azure Stack HCI is delivered as an Azure service and [billed to an Azure subscription](concepts/billing.md).

## Azure Stack HCI features and architecture

Azure Stack HCI is a world-class, integrated virtualization stack built on proven technologies that have already been deployed at scale, including Hyper-V, Storage Spaces Direct, and Azure-inspired SDN. It's part of the [Azure Stack family](/azure-stack/), using the same software-defined compute, storage, and networking software as [Azure Stack Hub](https://azure.microsoft.com/products/azure-stack/hub/).

Watch the video on the high-level features of Azure Stack HCI:

> [!VIDEO https://www.youtube.com/embed/fw8RVqo9dcs]

Each Azure Stack HCI cluster consists of between 2 and 16 physical, validated servers. The clustered servers share common configuration and resources by leveraging the Windows Server Failover Clustering feature.

Azure Stack HCI combines the following:

- Azure Stack HCI operating system
- Validated hardware from an OEM partner
- Azure hybrid services
- Windows Admin Center
- Hyper-V-based compute resources
- Storage Spaces Direct-based virtualized storage
- SDN-based virtualized networking using Network Controller (optional)

:::image type="content" source="media/overview/azure-stack-hci-solution.png" alt-text="The Azure Stack HCI OS runs on top of validated hardware, is managed by Windows Admin Center, and connects to Azure" border="false":::

Azure Stack HCI, version 20H2 provides new functionality not present in Windows Server, such as the ability to use Windows Admin Center to create a hyperconverged cluster that uses Storage Spaces Direct for superior storage price-performance. This includes the option to stretch the cluster across sites and use automatic failover. See [What's new in Azure Stack HCI](#whats-new-in-azure-stack-hci) for details.

## Why Azure Stack HCI?

There are many reasons customers choose Azure Stack HCI, including:

- It's familiar for Hyper-V and server admins, allowing them to leverage existing virtualization and storage concepts and skills.
- It works with existing data center processes and tools such as Microsoft System Center, Active Directory, Group Policy, and PowerShell scripting.
- It works with popular third-party backup, security, and monitoring tools.
- Flexible hardware choices allow customers to choose the vendor with the best service and support in their geography.
- Joint support between Microsoft and the hardware vendor improves the customer experience.
- Seamless, full-stack updates make it easy to stay current.
- A flexible and broad ecosystem gives IT professionals the flexibility they need to build a solution that best meets their needs.

### Common use cases for Azure Stack HCI

Customers often choose Azure Stack HCI in the following scenarios.

| Use case                                          | Description                             |
| :------------------------------------------------ | :-------------------------------------- |
| Branch office and edge                            | For branch office and edge workloads, you can minimize infrastructure costs by deploying two-node clusters with inexpensive witness options, such as Cloud Witness or a USB drive–based file share witness. Another factor that contributes to the lower cost of two-node clusters is support for switchless networking, which relies on crossover cable between cluster nodes instead of more expensive high-speed switches. Customers can also centrally view remote Azure Stack HCI deployments in the Azure portal. To learn more about this workload, see [Deploy branch office and edge on Azure Stack HCI](deploy/branch-office-edge.md).|
| Virtual desktop infrastructure (VDI)              | Azure Stack HCI clusters are well suited for large-scale VDI deployments with RDS or equivalent third-party offerings as the virtual desktop broker. Azure Stack HCI provides additional benefits by including centralized storage and enhanced security, which simplifies protecting user data and minimizes the risk of accidental or intentional data leaks. To learn more about this workload, see [Deploy virtual desktop infrastructure (VDI) on Azure Stack HCI](deploy/virtual-desktop-infrastructure.md).|
| Highly performant SQL Server                      | Azure Stack HCI provides an additional layer of resiliency to highly available, mission-critical Always On availability groups-based deployments of SQL Server. This approach also offers extra benefits associated with the single-vendor approach, including simplified support and performance optimizations built into the underlying platform. To learn more about this workload, see [Deploy SQL Server on Azure Stack HCI](deploy/sql-server.md).|
| Trusted enterprise virtualization                 | Azure Stack HCI satisfies the trusted enterprise virtualization requirements through its built-in support for Virtualization-based Security (VBS). VBS relies on Hyper-V to implement the mechanism referred to as virtual secure mode, which forms a dedicated, isolated memory region within its guest VMs. By using programming techniques, it's possible to perform designated, security-sensitive operations in this dedicated memory region while blocking access to it from the host OS. This considerably limits potential vulnerability to kernel-based exploits. To learn more about this workload, see [Deploy Trusted Enterprise Virtualization on Azure Stack HCI](deploy/trusted-enterprise-virtualization.md).|
| Azure Kubernetes Service (AKS)                    | You can leverage Azure Stack HCI to host container-based deployments, which increases workload density and resource usage efficiency. Azure Stack HCI also further enhances the agility and resiliency inherent to Azure Kubernetes deployments. Azure Stack HCI manages automatic failover of VMs serving as Kubernetes cluster nodes in case of a localized failure of the underlying physical components. This supplements the high availability built into Kubernetes, which automatically restarts failed containers on either the same or another VM. To learn more about this workload, see [What is Azure Kubernetes Service on Azure Stack HCI?](../aks-hci/overview.md).|
| Scale-out storage                                 | Storage Spaces Direct is a core technology of Azure Stack HCI that uses industry-standard servers with locally attached drives to offer high availability, performance, and scalability. Using Storage Spaces Direct results in significant cost reductions compared with competing offers based on storage area network (SAN) or network-attached storage (NAS) technologies. These benefits result from an innovative design and a wide range of enhancements, such as persistent read/write cache drives, mirror-accelerated parity, nested resiliency, and deduplication.|
| Disaster recovery for virtualized workloads       | An Azure Stack HCI stretched cluster provides automatic failover of virtualized workloads to a secondary site following a primary site failure. Synchronous replication ensures crash consistency of VM disks.|
| Data center consolidation and modernization       | Refreshing and consolidating aging virtualization hosts with Azure Stack HCI can improve scalability and make your environment easier to manage and secure. It's also an opportunity to retire legacy SAN storage to reduce footprint and total cost of ownership. Operations and systems administration are simplified with unified tools and interfaces and a single point of support.|

### Azure integration benefits

Azure Stack HCI allows you to take advantage of cloud and on-premises resources working together and natively monitor, secure, and back up to the cloud.

After you [register your Azure Stack HCI cluster](deploy/register-with-azure.md) with Azure, you can use the Azure portal initially for:

- **Monitoring:** View all of your Azure Stack HCI clusters in a single, global view where you can group them by resource group and tag them.
- **Billing:** Pay for Azure Stack HCI through your Azure subscription.

You can also subscribe to additional Azure hybrid services.

## What you need for Azure Stack HCI

To get started, you'll need:

- A cluster of two or more servers from the [Azure Stack HCI Catalog](https://hcicatalog.azurewebsites.net), purchased from your preferred Microsoft hardware partner.
- An [Azure subscription](https://azure.microsoft.com/).
- Operating system licenses for your workload VMs – for example, Windows Server. See [Activate Windows Server VMs](manage/vm-activate.md). 
- An internet connection for each server in the cluster that can connect via HTTPS outbound traffic to well-known Azure endpoints at least every 30 days. See [Azure connectivity requirements](manage/configure-firewalls.md) for more information.
- For clusters stretched across sites, you'll need at least one 1 Gb connection between sites (a 25 Gb RDMA connection is preferred), with an average latency of 5 ms round trip if you want to do synchronous replication where writes occur simultaneously in both sites.
- If you plan to use SDN, you'll need a virtual hard disk (VHD) for the Azure Stack HCI operating system to create Network Controller VMs (see [Plan to deploy Network Controller](concepts/network-controller.md)).

Make sure your hardware meets the [System requirements](concepts/system-requirements.md) and that your network meets the [physical network](concepts/physical-network-requirements.md) and [host network](concepts/host-network-requirements.md) requirements for Azure Stack HCI.

For Azure Kubernetes Service on Azure Stack HCI requirements, see [AKS requirements on Azure Stack HCI](../aks-hci/overview.md#what-you-need-to-get-started).

Azure Stack HCI is priced on a per core basis on your on-premises servers. For current pricing, see [Azure Stack HCI pricing](https://azure.microsoft.com/pricing/details/azure-stack/hci/).

## Hardware and software partners

You can purchase validated Azure Stack HCI solutions from your preferred Microsoft partner to get up and running without lengthy design and build time. Microsoft partners also offer a single point of contact for implementation and support services. You can either purchase validated nodes or an integrated system, which includes the Azure Stack HCI operating system pre-installed as well as partner extensions for driver and firmware updates.

Visit the [Azure Stack HCI solutions](https://azure.microsoft.com/overview/azure-stack/hci) page or browse the [Azure Stack HCI Catalog](https://hcicatalog.azurewebsites.net) to view 70+ Azure Stack HCI solutions currently available from Microsoft partners such as ASUS, Axellio, Blue Chip, DataON, Dell EMC, Fujitsu, HPE, Hitachi, Huawei, Lenovo, NEC, primeLine Solutions, QCT, SecureGUARD, and Supermicro.

There is also a variety of Microsoft partners working on software that extends the capabilities of Azure Stack HCI while allowing IT admins to use familiar tools. To learn more, see [Utility applications for Azure Stack HCI](concepts/utility-applications.md).

## What's new in Azure Stack HCI

Windows Admin Center version 2103 adds a number of features and improvements to Azure Stack HCI, including the following:

- **Windows Admin Center updates automatically**: Windows Admin Center and extensions now update automatically when a new release is available.

- **Deploy Network Controller using Windows Admin Center**: An update to the Cluster Creation extension in Windows Admin Center allows you to set up a Network Controller for Software Defined Networking (SDN) deployments.

- **Use Windows Admin Center in the Azure portal (Preview)**: Manage the Windows Server operating system running in an Azure VM by using Windows Admin Center directly in the Azure portal.
To learn more, see [Windows Admin Center in the Azure portal](https://cloudblogs.microsoft.com/windowsserver/2021/03/02/announcing-public-preview-of-window-admin-center-in-the-azure-portal/).

- **Event tool redesign (Preview)**: We’ve redesigned the Events tool for servers and PCs for the first time in ages. To check it out, open the Events tool and then toggle **Preview Mode**.

- **Install and manage Azure IoT Edge for Linux on Windows**: Install, manage, and troubleshoot IoT Edge for Linux on Windows from within Windows Admin Center.
To learn more, see [Enabling Linux based Azure IoT Edge Modules on Windows IoT](https://techcommunity.microsoft.com/t5/internet-of-things/enabling-linux-based-azure-iot-edge-modules-on-windows-iot/ba-p/2075882?ocid=wac2103).

- **Open tools in separate windows**: Connect to a system and then on the **Tools** menu, hover over a tool and select **Open tool in a separate window**.

- **Virtual machines tool improvements**: You can now create your own VM groups in the tool and edit columns. We’ve also made moving a VM easier, allowing you to reassign virtual switches while moving a VM.

For details on the new features and improvements, see [Windows Admin Center version 2103 is now generally available](https://techcommunity.microsoft.com/t5/windows-admin-center-blog/windows-admin-center-version-2103-is-now-generally-available/ba-p/2176438).

Clusters running Azure Stack HCI, version 20H2 have the following new features:

- **Azure Kubernetes Service hosting**: Azure Stack HCI now includes the ability to host [Azure Kubernetes Service on Azure Stack HCI](../aks-hci/overview.md).
- **New capabilities in Windows Admin Center**: With the ability to create and update hyperconverged clusters via an intuitive UI, Azure Stack HCI is easier than ever to use.
- **Stretched clusters for automatic failover**: Multi-site clustering with Storage Replica replication and automatic VM failover provides native disaster recovery and business continuity.
- **Affinity and anti-affinity rules**: These can be used similarly to how Azure uses Availability Zones to keep VMs and storage together or apart in clusters with multiple fault domains, such as stretched clusters.
- **Azure portal integration**: The Azure portal experience for Azure Stack HCI is designed to view all of your Azure Stack HCI clusters across the globe, with new features in development.
- **GPU acceleration for high-performance workloads**: AI/ML applications can benefit from boosting performance with GPUs.
- **BitLocker encryption**: You can now use BitLocker to encrypt the contents of data volumes on Azure Stack HCI, helping government and other customers stay compliant with standards such as FIPS 140-2 and HIPAA.
- **Improved Storage Spaces Direct volume repair speed**: Repair volumes quickly and seamlessly.

To learn more about what's new in Azure Stack HCI 20H2, [watch this video](https://www.youtube.com/watch?v=DPG7wGhh3sAa) from Microsoft Inspire.

## Video-based learning

The Azure extended network video is here:

- [Seamless connectivity to Azure with Windows Server and hybrid networking](https://www.youtube.com/watch?v=do2_4Y2p9dk)

Videos about the original, Windows Server 2019-based version of Azure Stack HCI:

- [Azure Stack and Azure Stack HCI overview view](https://aka.ms/AzureStackOverviewVideo)
- [Microsoft Ignite Live 2019 - Getting Started with Azure Stack HCI](https://www.youtube.com/watch?v=vueHIBqNIEU)
- [Discover Azure Stack HCI](https://www.youtube.com/watch?v=4aGZK0Ndmh8&list=PLQXpv_NQsPICdXZoH-EzlIFa4P6VS5m11&index=13&t=0s)
- [Modernize your retail stores or branch offices with Azure Stack HCI](https://www.youtube.com/watch?v=-JzLhjfkhmM&list=PLQXpv_NQsPICdXZoH-EzlIFa4P6VS5m11&index=9&t=0s)
- [What's new for Azure Stack HCI: 45 things in 45 minutes](https://www.youtube.com/watch?v=C5J4IEnlS_E&list=PLQXpv_NQsPICdXZoH-EzlIFa4P6VS5m11&index=12&t=0s)
- [Jumpstart your Azure Stack HCI deployment](https://www.youtube.com/watch?v=gxaPJLrWy5w&list=PLQXpv_NQsPICdXZoH-EzlIFa4P6VS5m11&index=11&t=0s)
- [The case of the shrinking data: Data Deduplication in Azure Stack HCI](https://www.youtube.com/watch?v=fmm4iDbDiY4&list=PLQXpv_NQsPICdXZoH-EzlIFa4P6VS5m11&index=23&t=0s)
- [Dave Kawula's notes from the field on Azure Stack HCI](https://www.youtube.com/watch?v=OXv7fLlz0ew&list=PLQXpv_NQsPICdXZoH-EzlIFa4P6VS5m11&index=2&t=0s)

Here's a video from a Hybrid Cloud Virtual Event:

- [Azure Stack HCI | Hybrid Cloud Virtual Event](https://www.youtube.com/watch?v=nxpoEva-R2Y)

## Next steps

- [Download Azure Stack HCI](https://azure.microsoft.com/products/azure-stack/hci/hci-download/)
- [Create an Azure Stack HCI cluster and register it with Azure](deploy/deployment-quickstart.md)
- [Use Azure Stack HCI with Windows Admin Center](get-started.md)
- [Compare Azure Stack HCI to Windows Server](concepts/compare-windows-server.md)
- [Compare Azure Stack HCI to Azure Stack Hub](concepts/compare-azure-stack-hub.md)