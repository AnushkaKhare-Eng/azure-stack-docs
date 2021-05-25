---
title: Quickstart to create a Kubernetes cluster using Windows Admin Center
description: Learn how to create a Kubernetes cluster using Windows Admin Center
author: davannaw-msft
ms.topic: quickstart
ms.date: 05/25/2021
ms.author: dawhite
---
# Quickstart: Create a Kubernetes cluster on Azure Stack HCI using Windows Admin Center

> Applies to: Azure Stack HCI, Windows Server 2019 Datacenter

After you have set up your Azure Kubernetes Service host, you can create a Kubernetes cluster using Windows Admin Center. To instead use PowerShell, see [Create a Kubernetes cluster with PowerShell](kubernetes-walkthrough-powershell.md).

Before proceeding to the Kubernetes cluster create wizard, make sure you have [Set up Azure Kubernetes Service](setup.md) and check the [system requirements](system-requirements.md) if you haven't already. The Kubernetes cluster create wizard can be reached through the [All Connections](#create-a-kubernetes-cluster-from-the-all-connections-page) page or the [Azure Kubernetes Service host dashboard](#create-a-kubernetes-cluster-from-the-azure-kubernetes-service-host-dashboard).

## Create a Kubernetes cluster from the All Connections page 

There are two ways to create a Kubernetes cluster in Windows Admin Center. One of these options is to do so from the **All Connections** page. Follow these steps, then proceed to the [Kubernetes cluster create wizard](#the-kubernetes-cluster-create-wizard) section: 

1. To begin creating a Kubernetes cluster in Windows Admin Center, press the **Add** button on the gateway screen. 
2. In the **Add or create resources** panel, under **Kubernetes cluster (preview)**, select **Create new** to launch the Kubernetes cluster wizard. While the “Add” button under Kubernetes clusters is present, it is nonfunctional. At any point during the Kubernetes cluster create process, you may exit the wizard, but note that your progress won't be saved. 

    ![Illustrates the Add or create resources blade in Windows Admin Center, which now includes the new tile for Kubernetes clusters.](.\media\create-kubernetes-cluster\add-connection.png)
  
## Create a Kubernetes cluster from the Azure Kubernetes Service host dashboard  

You may also create a Kubernetes cluster through the Azure Kubernetes Service host dashboard. This dashboard can be found in the Azure Kubernetes Service tool when you are connected to the system that has an Azure Kubernetes Service host deployed on it. Follow these steps, then proceed to the [Kubernetes cluster create wizard](#the-kubernetes-cluster-create-wizard) section: 

1. Connect to the system where you wish to create your Kubernetes cluster and navigate to the **Azure Kubernetes Service** tool. This system should already have an Azure Kubernetes Service host set up.

2. Select the **Add cluster** button under the **Kubernetes cluster** heading.

   ![Illustrates the Azure Kubernetes Service tool dashboard that appears after you set up an Azure Kubernetes Service host.](.\media\setup\dashboard.png)
  
## The Kubernetes cluster create wizard
You've reached the Kubernetes cluster create wizard through the All Connections page or the Azure Kubernetes Service tool. Let's get started:  

1. Review the prerequisites for the system that will host the Kubernetes cluster and those for Windows Admin Center. When finished, select **Next**. 

2. On the **Basics** page, configure information about your Kubernetes cluster. The Azure Kubernetes Service host field requires the fully qualified domain name of the Azure Stack HCI or Windows Server 2019 Datacenter cluster that you used when walking through the [setup](setup.md) page. You must have completed the host setup for this system through the Azure Kubernetes Service tool. When complete, select **Next**.

    [ ![Illustrates the Basics page of the Kubernetes cluster wizard.](.\media\create-kubernetes-cluster\basics.png) ](.\media\create-kubernetes-cluster\basics.png#lightbox)
 
3. Configure node pools to run your workloads on the **Node pools** page. You may add up to one Windows node pool and one Linux node pool (in addition to the primary node pool). If you'd like to enable Azure Arc integration later in this wizard, you will need to configure a Linux node pool with atleast 1 Linux worker node. When you're finished, select **Next**.

4. In the **Authentication** step, you may select if you'd like to enable Active Directory authentication. If you do choose to enable this feature, you will need to provide information such as your API Server service principal name, a Keytab file, and a cluster admin group or user name. When you're finished, select **Next**.

5. Specify your network configuration on the **Networking** page. You can either select an existing virtual network or create a new one by clicking on **Add network interface**. If you select the **Flannel** container network interface (CNI), keep in mind that only Windows or hybrid clusters are supported. Once **Flannel** is set, it cannot be changed, and the cluster will not support any network policy. If the **Calico** CNI is selected, note that it is not needed to support Calico Network Policy and Calico will be the default option for your network policy under **Security**. When complete, select **Next: Review + Create**.

    [ ![Illustrates the Networking, static page of the Kubernetes cluster wizard.](.\media\create-kubernetes-cluster\networking-static.png) ](\media\create-kubernetes-cluster\networking-static.png#lightbox)

    [ ![Illustrates the Networking, DHCP page of the Kubernetes cluster wizard.](.\media\create-kubernetes-cluster\networking-dhcp.png) ](\media\create-kubernetes-cluster\networking-dhcp.png#lightbox)
 
 
6. Review your selections on the **Review + create** page. When you're satisfied, select **Create** to begin deployment. Your deployment progress will be shown at the top of this page. 

7. After your deployment is complete, the **Next steps** page details how to manage your cluster. If you chose to disable the Azure Arc integration in the previous step, some of the information and instructions on this page may not be available or functional.


## Next steps

In this quickstart, you deployed a Kubernetes cluster. To learn more about Azure Kubernetes Service on Azure Stack HCI and walk through how to deploy and manage Linux applications on AKS on Azure Stack HCI, continue to the following tutorial:

- [Tutorial: Deploy Linux applications in Azure Kubernetes Service on Azure Stack HCI](deploy-linux-application.md)
