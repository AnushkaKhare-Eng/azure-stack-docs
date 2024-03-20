---
title: System requirements and support matrix for AKS enabled by Azure Arc on VMware (preview)
description: Learn about system requirements and the support matrix for AKS enabled by Azure Arc on VMware.
ms.date: 03/18/2024
ms.topic: conceptual
author: sethmanheim
ms.author: sethm 

---

# System requirements and support matrix (preview)

This article describes the system requirements and support matrix for setting up AKS enabled by Azure Arc on VMware. For an overview of AKS Arc on VMware, [see the overview article](aks-vmware-overview.md).

## Arc-enabled VMware vSphere requirements

To use the AKS Arc on VMware preview, you must first onboard [Arc-enabled VMware vSphere](/azure/azure-arc/vmware-vsphere/overview) by connecting vCenter to Azure through the [Arc Resource Bridge](/azure/azure-arc/resource-bridge/overview), with the Kubernetes Extension for AKS Arc Operators installed. If you already completed this step, you can proceed with the AKS Arc on VMware requirements.

### Support matrix

The support matrix for Arc-enabled VMware vSphere is documented in the [Support matrix for Azure Arc-enabled VMware vSphere](/azure/azure-arc/vmware-vsphere/support-matrix-for-arc-enabled-vmware-vsphere) requirements article.

### Plan deployment

For the prerequisites to onboard Arc-enabled VMware vSphere, see [Quickstart: Connect VMware vCenter Server to Azure Arc by using the helper script](/azure/azure-arc/vmware-vsphere/quick-start-connect-vcenter-to-arc-using-script).

### Required input

A typical onboarding process that uses the script takes 30 minutes. During this process, you are prompted for the details specified in [Inputs for the script to deploy Arc-enabled VMware vSphere](/azure/azure-arc/vmware-vsphere/quick-start-connect-vcenter-to-arc-using-script#inputs-for-the-script).

## AKS enabled by Azure Arc on VMware requirements

This section describes the requirements for deploying AKS enabled by Azure Arc on VMware.

### VMware vCenter requirements

Before you deploy AKS on VMware, you must set up a few things in VMware vCenter. After that, you deploy the AKS clusters in the same resource pool, VM folder, and datastore in which you deployed the Arc Resource Bridge.

#### Entitlement

You need a designated VMware administration user for the AKS clusters. This user should have the following permissions:

- This role can read all inventory, deploy, and update virtual machines (VMs) to all the resource pools (or clusters), networks, and virtual machine templates that you plan to use with AKS on VMware.

#### Resource pool

In this preview release, the Arc Resource Bridge and the target clusters share a resource pool. To set this up, create a resource pool for the Arc Resource Bridge and the target cluster(s) with the following minimum specifications:

| Cluster type                  | Memory  | vCPUs | Storage  |
|-------------------------------|---------|-------|----------|
| Arc Resource Bridge           | 16 GiB  | 4     | 100 GiB  |
| Target cluster control plane  | 16 GiB  | 4     | 100 GiB  |
| Target cluster worker node    | 4  GiB  | 2     | 100 GiB  |

For information about supported VM size options, see the [AKS Arc on VMware scale requirements](aks-vmware-scale-requirements.md).

> [!WARNING]
> For the target cluster control plane, there is a known issue with the VM size **Standard_A4_v2**, which is currently deployed with 2 vCPUs and 8 GiB memory. Until this issue is fixed, we recommend you deploy target cluster control plane with 16 GiB, and 4 vCPUs. For more information about support size options, see the [AKS Arc on VMware scale requirements](aks-vmware-scale-requirements.md). For known issues, see [troubleshooting/ Known issues](aks-vmware-known-issues.md).

#### VM folder and VM templates

You should create a folder for VM templates to store the Arc Resource Bridge and CBL Mariner Linux VM templates used to create AKS on VMware clusters.

## Azure requirements

You must connect to your Azure account. For more example:

```azurecli  
az login --use-device-code
```

For more information, see [Connect to Azure using the Azure CLI](/cli/azure/authenticate-azure-cli-interactively).

### Azure Checklist

| Parameter                     | Parameter details  |
|-------------------------------|--------------------|
| $appliance_Name               | Name of the Arc Resource Bridge created to connect vCenter with Azure  | 
| $custom_Location              | Name or ID of the custom location created for Arc Resource Bridge and the same name applies to AKS extension  | 
| $subscriptionID               | The Azure subscription ID where you install the Azure Arc Resource Bridge and custom location  | 
| $resource_Group               | The resource group in the Azure subscription where you install the Arc Resource Bridge and custom location  | 
| $network_name                 | Name of the VMware network resource enabled in Azure  | 


### Microsoft Entra permissions, role and access level

You must have sufficient permissions to register an application with your Microsoft Entra tenant. To check that you have sufficient permissions, follow these steps:

- Go to the Azure portal and select [Roles and administrators](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RolesAndAdministrators) under Microsoft Entra ID to check your role.
- If your role is **User**, you must make sure that non-administrators can register applications.
- To check if you can register applications, go to [User settings](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/UserSettings) under the Microsoft Entra service to check if you have permission to register an application.

If the app registrations setting is set to **No**, only users with an administrator role can register these types of applications. To learn about the available administrator roles and the specific permissions in Microsoft Entra ID that are given to each role, see [Microsoft Entra built-in roles](/azure/active-directory/roles/permissions-reference#all-roles). If your account is assigned the **User** role, but the app registration setting is limited to admin users, ask your administrator either to assign you one of the administrator roles that can create and manage all aspects of app registrations, or to enable users to register apps.

If you don't have permissions to register an application and your admin can't give you these permissions, the easiest way to deploy AKS is to ask your Azure admin to create a service principal with the right permissions. Admins can check the following section to learn how to create a service principal.

### Azure resource group

You must have an Azure resource group in the supported regions before registration.

#### Supported regions

You can use the AKS Arc on VMware preview in the following supported regions:
- East US


> [!WARNING]
> The AKS Arc on VMware preview currently supports cluster creation exclusively within the specified Azure regions. If you attempt to deploy in a region outside of this list, a deployment failure occurs.

#### Data residency

AKS Arc on VMware doesn't store or process customer data outside the region in which the customer deploys the service instance.



## Next steps

- If you already connected vCenter to Azure Arc and want to add the AKS extension, [see the Quickstart](aks-vmware-quickstart-deploy.md).
- If your vCenter is not connected to Azure Arc and you want to add the AKS extension, see the Quickstart: Connect VMware vCenter Server to Azure Arc using the help script".
