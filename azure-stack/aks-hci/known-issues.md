---
title: Known issues for Azure Kubernetes Service on Azure Stack HCI 
description: Known issues for Azure Kubernetes Service on Azure Stack HCI 
author: abha
ms.topic: troubleshooting
ms.date: 09/22/2020
ms.author: abha
ms.reviewer: 
---

# Known Issues for Azure Kubernetes Service on Azure Stack HCI Public Preview
This article describes known issues with the public preview release of Azure Kubernetes Service on Azure Stack HCI.

## Recovering from a failed AKS on Azure Stack HCI deployment
If you're experiencing deployment issues or want to reset your deployment, make sure you close all Windows Admin Center instances connected to Azure Kubernetes Service on Azure Stack HCI before running Uninstall-AksHci from a PowerShell administrative window.

## Set-AksHciConfig fails with WinRM errors, but shows WinRM is configured correctly
When running [Set-AksHciConfig](./set-akshciconfig.md), you might encounter the following error:

```powershell
WinRM service is already running on this machine.
WinRM is already set up for remote management on this computer.
Powershell remoting to TK5-3WP08R0733 was not successful.
At C:\Program Files\WindowsPowerShell\Modules\Moc\0.2.23\Moc.psm1:2957 char:17
+ ...             throw "Powershell remoting to "+$env:computername+" was n ...
+                 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OperationStopped: (Powershell remo...not successful.:String) [], RuntimeException
    + FullyQualifiedErrorId : Powershell remoting to TK5-3WP08R0733 was not successful.
```

Most of the time, this error occurs as a result of a change in the user's security token (due to a change in group membership), a password change, or an expired password. In most cases, the issue can be remediated by logging off from the computer and logging back in. If this still fails, you can file an issue at [GitHub AKS HCI issues](https://aka.ms/aks-hci/issues).

## When using kubectl to delete a node, the associated VM might not be deleted
You'll meet this issue if you follow these steps:
* Create a Kubernetes cluster.
* Scale the cluster to more than two nodes.
* Use `kubectl` delete node <node-name> to delete a node.
* Run `kubectl` get nodes. The removed node isn't listed in the output.
* Open a PowerShell Admin Window.
* Run `get-vm`. The removed node is still listed

This failure leads to the system not recognizing the node is missing and a new node will not spin up. 

## The workload cluster may not be found if the IP address pools of two AKS on Azure Stack HCI deployments are the same or overlap

If you deploy two AKS on Azure Stack HCI hosts and use the same `AksHciNetworkSetting` configuration for both, PowerShell and WAC will potentially fail to find the workload cluster as the API server will be assigned the same IP address in both clusters resulting in a conflict.

The error message you receive will look similar to the example shown below.

```powershell
A workload cluster with the name 'clustergroup-management' was not found.
At C:\Program Files\WindowsPowerShell\Modules\Kva\0.2.23\Common.psm1:3083 char:9
+         throw $("A workload cluster with the name '$Name' was not fou ...
+         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OperationStopped: (A workload clus... was not found.:String) [], RuntimeException
    + FullyQualifiedErrorId : A workload cluster with the name 'clustergroup-management' was not found.
```

> [!NOTE]
> Your cluster name will be different.

## Time synchronization must be configured across all physical cluster nodes and in Hyper-V
To ensure gMSA and AD authentication works, ensure that the Azure Stack HCI cluster nodes are configured to synchronize their time with a domain controller or other
time source and that Hyper-V is configured to synchronize time to any virtual machines.

## Hyper-V manager shows high CPU and/or memory demands for the management cluster
When you check Hyper-V manager, high CPU and memory demands for the management cluster can be safely ignored. They are usually related to spikes in compute resource usage when provisioning workload clusters. Increasing the memory or CPU size for the management cluster has not shown a significant improvement and can be safely ignored.

## Special Active Directory permissions are needed for domain joined Azure Stack HCI nodes 
Users deploying and configuring Azure Kubernetes Service on Azure Stack HCI need to have "Full Control" permission to create AD objects in the Active Directory container the server and service objects are created in. 

## Azure Kubernetes Service PowerShell deployment doesn't check for available memory before creating a new target cluster
The **Aks-Hci** PowerShell commands do not validate the available memory on the host server before creating Kubernetes nodes. This can lead to memory exhaustion and virtual machines that do not start. This failure is currently not handled gracefully, and the deployment will stop responding with no clear error message.
If you have a deployment that stops responding, open `Eventviewer` and check for a Hyper-V-related error message indicating there's not enough memory to start the VM.

## Azure Kubernetes Service deployment fails on an Azure Stack HCI configured with static IPs, VLANs, SDN, or proxies.
While deploying Azure Kubernetes Service on an Azure Stack HCI cluster that has static IPs, VLANs, SDN, or proxies, the deployment fails at cluster creation. 

## IPv6 must be disabled in the hosting environment
If both IPv4 and IPv6 addresses are bound to the physical NIC, the `cloudagent` service for clustering uses the IPv6 address for communication. Other components in the deployment framework only use IPv4. This will result in Windows Admin Center unable to connect to the cluster and will report a remoting failure when trying to connect to the machine.
Workaround: Disable IPv6 on the physical network adapters.
This issue will be fixed in a future release

## Moving virtual machines between Azure Stack HCI cluster nodes quickly leads to VM startup failures
When using the cluster administration tool to move a VM from one node (Node A) to another node (Node B) in the Azure Stack HCI cluster, the VM may fail to start on the new node. After moving the VM back to the original node it will fail to start there as well.
This issue happens because the logic to clean up the first migration runs asynchronously. As a result, Azure Kubernetes Service's "update VM location" logic finds the VM on the original Hyper-V on node A, and deletes it, instead of unregistering it.
Workaround: Ensure the VM has started successfully on the new node before moving it back to the original node.
This issue will be fixed in a future release.

## Load balancer in Azure Kubernetes Service requires DHCP reservation
The load balancing solution in Azure Kubernetes Service on Azure Stack HCI uses DHCP to assign IP addresses to service endpoints. If the IP address changes for the service endpoint due to a service restart, DHCP lease expires due to a short expiration time. The service will therefore become inaccessible because the IP address in the Kubernetes configuration is different from what it is on the end point. This can lead to the Kubernetes cluster becoming unavailable.
To get around this issue, use a MAC address pool for the load balanced service endpoints and reserve specific IP addresses for each MAC address in the pool.
This issue will be fixed in a future release.

## Cannot deploy Azure Kubernetes Service to an environment that has separate storage and compute clusters
Windows Admin Center will not deploy Azure Kubernetes Service to an environment with separate storage and compute clusters as it expects the compute and storage resources to be provided by the same cluster. In most cases, it will not find CSVs exposed by the compute cluster and will refuse to continue with deployment.
This issue will be fixed in a future release.

## Windows Admin Center only supports Azure Kubernetes Service for Azure Stack HCI in desktop mode
In preview, all Azure Kubernetes Service for Azure Stack HCI functionality is only supported in Windows Admin Center desktop mode. The Windows Admin Center gateway must be installed on a Windows 10 PC. For more information about Windows Admin Center installation options, visit the [Windows Admin Center documentation](/windows-server/manage/windows-admin-center/plan/installation-options). Additional scenarios will be supported in a future release.

## Azure Kubernetes Service host setup fails in Windows Admin Center if reboots are required
The Azure Kubernetes Service host setup wizard will fail if the one or more servers you are using need to be rebooted to install roles like PowerShell or Hyper-V. The current workaround is to exit the wizard and try again on the same system after the servers come back online. This issue will be fixed in a future release.

## Azure registration step in Azure Kubernetes Service host setup asks to try again
When using Windows Admin Center to set up the Azure Kubernetes Service host, you may be asked to try again after entering the required information on the Azure registration page. You may need to sign into Azure again on the Windows Admin Center gateway to proceed with this step. This issue will be fixed in a future release.

## Windows Admin Center doesn't have an Arc offboarding experience
Windows Admin Center does not currently have a process to offboard a cluster from Azure Arc. To delete Arc aganets on a cluster that has been destroyed, navigate to the resource group the of the cluster in the Azure portal and manually delete the Arc content. To delete Arc agents on a cluster that is still up and running, users should run the following command:
```PowerShell
az connectedk8s delete
```

## When setting up an Azure Kubernetes Service host using Windows Admin Center, setup may fail if File Explorer is open
If File Explorer is open and in the **C:\Program Files\AksHci** directory when you reach the "Review + create" step, your creation may fail with the error "The process could not access the file 'C:\Program Files\AksHci\wssdcloudagent.exe'. This is because it's being used by another process. To avoid this error, close File Explorer or navigate to a different directory before reaching this step. 

## Cannot connect Windows Admin Center to Azure as create new Azure App ID fails
If you're unable to connect Windows Admin Center to Azure because you can't automatically create and use an Azure App ID on the gateway, create an Azure App ID and assign it the right permissions on the portal. Then, select **Use existing in the gateway**. For more information, visit [connecting your gateway to Azure.](/windows-server/manage/windows-admin-center/azure/azure-integration).
