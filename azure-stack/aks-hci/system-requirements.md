---
title: Azure Kubernetes Service on Azure Stack HCI requirements
description: Before you begin Azure Kubernetes Service on Azure Stack HCI
ms.topic: conceptual
author: abhilashaagarwala
ms.author: abha
ms.date: 05/21/2021
---

# System requirements for Azure Kubernetes Service on Azure Stack HCI

> Applies to: Azure Stack HCI, Windows Server 2019 Datacenter

This article covers the requirements for setting up Azure Kubernetes Service on Azure Stack HCI or on Windows Server 2019 Datacenter and using it to create Kubernetes clusters. For an overview of Azure Kubernetes Service on Azure Stack HCI, see [AKS on Azure Stack HCI overview](overview.md).

## Determine hardware requirements

Microsoft recommends purchasing a validated Azure Stack HCI hardware/software solution from our partners. These solutions are designed, assembled, and validated against our reference architecture to ensure compatibility and reliability so you get up and running quickly. Check that the systems, components, devices, and drivers you are using are Windows Server 2019 Certified per the Windows Server Catalog. Visit the [Azure Stack HCI solutions](https://azure.microsoft.com/overview/azure-stack/hci) website for validated solutions.

## General requirements

For Azure Kubernetes Service on Azure Stack HCI or Windows Server 2019 Datacenter to function optimally in an Active Directory environment, ensure the following requirements are fulfilled:

- Ensure time synchronization is set up and the divergence is not greater than 2 minutes across all cluster nodes and the domain controller. For information on setting time synchronization, see [Windows Time Service](/windows-server/networking/windows-time-service/windows-time-service-top).

- Ensure the user account(s) that adds updates, and manages Azure Kubernetes Service on Azure Stack HCI or Windows Server 2019 Datacenter clusters has the correct permissions in Active Directory. If you are using Organizational Units (OUs) to manage group policies for servers and services, the user account(s) will require list, read, modify, and delete permissions on all objects in the OU.

- We recommend using a separate OU for the servers and services to which you add your Azure Kubernetes Service on Azure Stack HCI or Windows Server 2019 Datacenter clusters. Using a separate OU allows you to control access and permissions with more granularity.

- If you are using GPO templates on containers in Active Directory, ensure deploying AKS on Azure Stack HCI is exempt from the policy. Server hardening will be available in a subsequent preview release.

## Azure requirements

You must have an Azure resource group in the East US, Southeast Asia, or West Europe Azure region available before registration. If you don’t already have an Azure account, [create one](https://azure.microsoft.com). You can use an existing subscription of any type:
- Free account with Azure credits for [students](https://azure.microsoft.com/free/students/) or [Visual Studio subscribers](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/)
- [Pay-as-you-go](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/) subscription with credit card
- Subscription obtained through an Enterprise Agreement (EA)
- Subscription obtained through the Cloud Solution Provider (CSP) program

If you're using Windows Admin Center to deploy an AKS Host or an AKS workload cluster, you must have an Azure subscription on which you are an **Owner**. You can check your access level by navigating to your subscription, clicking on **Access control (IAM)** on the left hand side of the Azure portal and then clicking on **View my access**.

If you're using PowerShell to deploy an AKS Host or an AKS workload cluster, the user registering the cluster must have **at least one** of the following:
- A user account with the built-in **Owner** role 
- A service principal with either the built-in **Kubernetes Cluster - Azure Arc Onboarding** role (minimum), the built-in **Contributer** role, or the built-in **Owner** role

If your Azure subscription is through an EA or CSP, the easiest way to get the required permissions is to ask your Azure subscription admin to give you a service principal with a built-in role mentioned above. Check the below section on how to create a service principal.

### Optional: Create a new service principal

Run the following steps to create a new service principal with the built-in **Microsoft.Kubernetes connected cluster** role. Note that only subscription owners can create service principals with the right role assignment. You can check your access level by navigating to your subscription, clicking on **Access control (IAM)** on the left hand side of the Azure portal and then clicking on **View my access**.

Install and import the following Azure PowerShell modules:

```powershell
Install-Module -Name Az.Accounts -Repository PSGallery -RequiredVersion 2.2.4
Import-Module Az.Accounts 
Install-Module -Name Az.Resources -Repository PSGallery -RequiredVersion 3.2.0
Import-Module Az.Resources
Install-Module -Name AzureAD -Repository PSGallery -RequiredVersion 2.0.2.128
Import-Module AzureAD
```
**Close all PowerShell windows** and reopen a new administrative session.

Log in to Azure using the [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) PowerShell command: 

```powershell
Connect-AzAccount
```

Set the subscription you want to use to register your AKS host for billing as the default subscription by running the [Set-AzContext](/powershell/module/az.accounts/set-azcontext) command.
```powershell
Set-AzContext -Subscription myAzureSubscription
```

Verify that your login context is correct by running the [Get-AzContext](/powershell/module/az.accounts/get-azcontext) PowerShell command. Verify that the subscription, tenant and account is what you want to use to register your AKS host for billing.

```powershell
Get-AzContext
```

```output
Name                                     Account                      SubscriptionName             Environment                  TenantId
----                                     -------                      ----------------             -----------                  --------
myAzureSubscription (92391anf-...        user@contoso.com             myAzureSubscription          AzureCloud                   xxxxxx-xxxx-xxxx-xxxxxx
```

Create a service principal by running the [New-AzADServicePrincipal](/powershell/module/az.resources/new-azadserviceprincipal) PowerShell command. This command creates a service principal with the  "Microsoft.Kubernetes connected cluster" role and sets the scope at a subscription level. For more information on creating service principals, visit [create an Azure service principal with Azure PowerShell](/powershell/azure/create-azure-service-principal-azureps?view=azps-5.9.0&preserve-view=true).

```powershell
$sp = New-AzADServicePrincipal -role "Microsoft.Kubernetes connected cluster"
```

Retrieve the password for the service principal by running the following command:

```powershell
$secret = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto([System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($sp.Secret))
Write-Host "Application ID: $($sp.ApplicationId)"
Write-Host "App Secret: $secret"
```   

From the output above, you now have the **application ID** and the **secret** available when deploying AKS on Azure Stack HCI. You should take a note of these items and store them safely.
With that created, in the **Azure portal**, under **Subscriptions**, **Access Control**, and then **Role Assignments**, you should see your new Service Principal.

## Compute requirements

- For test environments: An Azure Stack HCI cluster or a Windows Server 2019 Datacenter failover cluster with a maximum of four servers in the cluster. We recommend that each server in the cluster have at least 8 (recommended 16) CPU cores and at least 256 GB RAM.

- For production environments: An Azure Stack HCI cluster or a Windows Server 2019 Datacenter failover cluster with a maximum of four servers in the cluster. We recommend that each server in the cluster have at least 16 (recommended 32) CPU cores and at least 256 GB RAM. The final sizing will depend on the application and number of worker nodes you are planning to deploy on the Azure Stack HCI cluster.

- While you can technically run Azure Kubernetes Service on a single node Windows Server 2019 Datacenter, we do not recommend doing so. However, you can run Azure Kubernetes Service on a single node Windows Server 2019 Datacenter for evaluation purposes.

- Other compute requirements for Azure Kubernetes Service on Azure Stack HCI are in line with Azure Stack HCI’s requirements. Visit [Azure Stack HCI system requirements](../hci/concepts/system-requirements.md#server-requirements) for more details on Azure Stack HCI server requirements.

- This preview release requires that you install the Azure Stack HCI operating system on each server in the cluster using the EN-US region and language selections; changing them after installation isn't sufficient at this time.

## General network requirements

The following requirements apply to an Azure Stack HCI cluster as well as a Windows Server 2019 Datacenter cluster:

- Verify that you have an existing, external virtual switch configured if you’re using Windows Admin Center. For Azure Stack HCI clusters, this switch and its name must be the same across all cluster nodes. 

- Verify that you have disabled IPv6 on all network adapters.

- For a successful deployment, the Azure Stack HCI cluster nodes and the Kubernetes cluster VMs must have external internet connectivity.

- Make sure all subnets you define for the cluster are routable amongst each other and to the internet.
  
- Make sure that there is network connectivity between Azure Stack HCI hosts and the tenant VMs.

- DNS name resolution is required for all nodes to be able to communicate with each other.

## IP address assignment  

In AKS on Azure Stack HCI, virtual networks are used to allocate IP addresses to the Kubernetes resources that require them, as listed above. There are two networking models to choose from, depending on your desired AKS on Azure Stack HCI networking architecture.

> [!NOTE]
 > The virtual networking architecture defined here for your AKS on Azure Stack HCI deployments is different from the underlying physical networking architecture in your data center.

- Static IP networking - The virtual network allocates static IP addresses to the Kubernetes cluster API server, Kubernetes nodes, underlying VMs, load balancers and any Kubernetes services you run on top of your cluster.

- DHCP networking - The virtual network allocates dynamic IP addresses to the Kubernetes nodes, underlying VMs and load balancers using a DHCP server. The Kubernetes cluster API server and any Kubernetes services you run on top of your cluster are still allocated static IP addresses.

### Minimum IP address reservation

At a minimum, you should reserve the following number of IP addresses for your deployment:

| Cluster type  | Control plane node | Worker node | For update operations | Load balancer  |
| ------------- | ------------------ | ---------- | ----------| -------------|
| AKS Host |  1 IP |  NA  |  2 IP |  NA  |
| Workload cluster  |  1 IP per node  | 1 IP per node |  5 IP  |  1 IP |

Additionally, you should reserve the following number of IP addresses for your VIP pool:

| Resource type  | Number of IP addresses |
| ------------- | ------------------ |
| Cluster API server |  1 per cluster |
| Kubernetes Services  |  1 per service |

As you can see, the number of required IP addresses is variable depending on the AKS on Azure Stack HCI architecture and the number of services you run on your Kubernetes cluster. We recommend reserving a total of 256 IP addresses (/24 subnet) for your deployment.

For more information on networking requirements, visit [node networking concepts in AKS on Azure Stack HCI](./concepts-node-networking.md) and [container networking concepts in AKS on Azure Stack HCI](./concepts-container-networking.md).

### Network port and URL requirements

When creating an Azure Kubernetes Cluster on Azure Stack HCI, the following firewall ports are automatically opened on each server in the cluster.

| Firewall port               | Description     |
| ---------------------------- | ------------ |
| 45000           | wssdagent GPRC   server port     |
| 45001             | wssdagent GPRC authentication port  |
| 55000           | wssdcloudagent GPRC   server port      |
| 65000            | wssdcloudagent GPRC authentication port  |

Firewall URL exceptions are needed for the Windows Admin Center machine and all nodes in the Azure Stack HCI cluster.

| URL        | Port | Service | Notes |
| ---------- | ---- | --- | ---- |
<https://helm.sh/blog/get-helm-sh/>  | 443 | Download Agent, WAC | Used to download the Helm binaries
<https://storage.googleapis.com/>  | 443 | Cloud Init | Downloading Kubernetes binaries
<https://azurecliprod.blob.core.windows.net/> | 443 | Cloud Init | Downloading binaries and containers
<https://aka.ms/installazurecliwindows> | 443 | WAC | Downloading Azure CLI
<https://443> | 443 | TCP | Used to support Azure Arc agents  
*.blob.core.windows.net | 443 | TCP | Required for downloads
*.dl.delivery.mp.microsoft.com, *.do.dsp.mp.microsoft.com. | 80, 443 | Download Agent | Downloading VHD images
ecpacr.azurecr.io | 443 | Kubernetes | Downloading container images
git://:9418 | 9418 | TCP | Used to support Azure Arc agents
https://azurestackhci.azurefd.net | | AzureFrontDoor.Frontend | Azure Stack HCI Cloud Service

> [!NOTE]
> To view all ports, URLs, and endpoints for Azure Arc enabled Kubernetes, see [network requirements](/azure/azure-arc/kubernetes/quickstart-connect-cluster#meet-network-requirements).

## Storage requirements

The following storage implementations are supported by Azure Kubernetes Service on Azure Stack HCI:

|  Name                         | Storage type | Required capacity |
| ---------------------------- | ------------ | ----------------- |
| Azure Stack HCI Cluster          | CSV          | 1 TB              |
| Windows Server 2019 Datacenter failover cluster          | CSV          | 1 TB              |
| Single Node Windows Server 2019 Datacenter | Direct Attached Storage | 500 GB|

## Review maximum supported hardware specifications

Azure Kubernetes Service on Azure Stack HCI deployments that exceed the following specifications are not supported:

| Resource                     | Maximum |
| ---------------------------- | --------|
| Physical servers per cluster | 4       |
| Kubernetes Clusters            | 4       |
| Total number of VMs          | 200     |

## Windows Admin Center requirements

Windows Admin Center is the user interface for creating and managing Azure Kubernetes Service on Azure Stack HCI. To use Windows Admin Center with Azure Kubernetes Service on Azure Stack HCI, you must meet all the criteria in the list below.

Here are the requirements for the machine running the Windows Admin Center gateway: 

- Windows 10 or Windows Server machine
- [Registered with Azure](/windows-server/manage/windows-admin-center/azure/azure-integration)
- In the same domain as the Azure Stack HCI or Windows Server 2019 Datacenter cluster
- An Azure subscription on which you are an Owner. You can check your access level by navigating to your subscription and clicking on **Access control (IAM)** on the left hand side of the Azure portal and then clicking on **View my access**.

## Next steps

After you have satisfied all of the prerequisites above, you can set up an Azure Kubernetes Service host on Azure Stack HCI using:

- [Windows Admin Center](setup.md)
- [PowerShell](kubernetes-walkthrough-powershell.md)
