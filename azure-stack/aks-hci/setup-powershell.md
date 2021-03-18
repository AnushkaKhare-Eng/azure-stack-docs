---
title: Quickstart to set up an Azure Kubernetes Service host on Azure Stack HCI using Windows PowerShell
description: Learn how to set up an Azure Kubernetes Service host on Azure Stack HCI with Windows PowerShell
author: jessicaguan
ms.topic: quickstart
ms.date: 02/12/2021
ms.author: jeguan
---
# Quickstart: Set up an Azure Kubernetes Service host on Azure Stack HCI using PowerShell

> Applies to: Azure Stack HCI, Windows Server 2019 Datacenter

In this quickstart, you'll learn how to set up an Azure Kubernetes Service host using PowerShell. To instead use Windows Admin Center, see [Set up with Windows Admin Center](setup.md).

## Before you begin

Make sure you have one of the following:
 - 2-4 node Azure Stack HCI cluster
 - Windows Server 2019 Datacenter failover cluster
 - Single node Windows Server 2019 Datacenter 
 
Before getting started, make sure you have satisfied all the prerequisites on the [system requirements](.\system-requirements.md) page. 
**We recommend having a 2-4 node Azure Stack HCI cluster.** If you don't have any of the above, follow instructions on the [Azure Stack HCI registration page](https://azure.microsoft.com/products/azure-stack/hci/hci-download/).    

## Download and install the AksHci PowerShell module

Download the `AKS-HCI-Public-Preview-Mar-2021` from the [Azure Kubernetes Service on Azure Stack HCI registration page](https://aka.ms/AKS-HCI-Evaluate). The zip file `AksHci.Powershell.zip` contains the PowerShell module.

**Close all PowerShell windows.** Delete any existing directories for AksHci, AksHci.Day2, Kva, MOC, and MSK8sDownloadAgent located in the path `%systemdrive%\program files\windowspowershell\modules`. Once the directories are deleted, you can extract the contents of the new zip file. Make sure to extract the zip file in the correct location (`%systemdrive%\program files\windowspowershell\modules`).

   ```powershell
   Import-Module AksHci
   ```

**Close all PowerShell windows** and reopen a new administrative session to check if you have the latest version of the PowerShell module.
  
   ```powershell
   Get-Command -Module AksHci
   ```

 **Output:**
```
CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           Initialize-AksHciNode                              0.2.25     AksHci
Function        Add-AksHciGMSACredentialSpec                       0.2.25     AksHci
Function        Get-AksHciCluster                                  0.2.25     AksHci
Function        Get-AksHciClusterUpgrades                          0.2.25     AksHci
Function        Get-AksHciConfig                                   0.2.25     AksHci
Function        Get-AksHciCredential                               0.2.25     AksHci
Function        Get-AksHciEventLog                                 0.2.25     AksHci
Function        Get-AksHciKubernetesVersion                        0.2.25     AksHci
Function        Get-AksHciLogs                                     0.2.25     AksHci
Function        Get-AksHciUpdates                                  0.2.25     AksHci
Function        Get-AksHciVersion                                  0.2.25     AksHci
Function        Get-AksHciVmSize                                   0.2.25     AksHci
Function        Install-AksHci                                     0.2.25     AksHci
Function        Install-AksHciAdAuth                               0.2.25     AksHci
Function        Install-AksHciArcOnboarding                        0.2.25     AksHci
Function        Install-AksHciGMSAWebhook                          0.2.25     AksHci
Function        New-AksHciCluster                                  0.2.25     AksHci
Function        New-AksHciNetworkSetting                           0.2.25     AksHci
Function        Remove-AksHciCluster                               0.2.25     AksHci
Function        Remove-AksHciGMSACredentialSpec                    0.2.25     AksHci
Function        Repair-AksHciClusterCerts                          0.2.25     AksHci
Function        Restart-AksHci                                     0.2.25     AksHci
Function        Set-AksHciClusterNodeCount                         0.2.25     AksHci
Function        Set-AksHciConfig                                   0.2.25     AksHci
Function        Uninstall-AksHci                                   0.2.25     AksHci
Function        Uninstall-AksHciAdAuth                             0.2.25     AksHci
Function        Uninstall-AksHciArcOnboarding                      0.2.25     AksHci
Function        Uninstall-AksHciGMSAWebhook                        0.2.25     AksHci
Function        Update-AksHci                                      0.2.25     AksHci
Function        Update-AksHciCluster                               0.2.25     AksHci
```

After running the above command, close all PowerShell windows and reopen an administrative session to run the commands in the following steps.

## Step 1: Prepare your machine(s) for deployment

Run checks on every physical node to see if all the requirements are satisfied to install Azure Kubernetes Service on Azure Stack HCI.
Open PowerShell as an administrator and run the following command.

```powershell
Initialize-AksHciNode
```

When the checks are finished, you'll see "Done" displayed in green text.

## Step 2: Create a virtual network

To get the names of your available external switches, run this command:

```powershell
Get-VMSwitch
```

You must use an external switch.

Sample Output:
```output
Name SwitchType NetAdapterInterfaceDescription
---- ---------- ------------------------------
extSwitch External Mellanox ConnectX-3 Pro Ethernet Adapter
```

To create a virtual network for the nodes in your deployment to use, create an environment variable with the [new-akshcinetworksetting](.\new-akshcinetworksetting.md) PowerShell command. This will be used later to configure a deployment that uses static IP. If you want to configure your AKS deployment with DHCP, visit [new-akshcinetworksetting](.\new-akshcinetworksetting.md) for examples.

```powershell
#static IP
$vnet = New-AksHciNetworkSetting -vnetName "extSwitch" -k8sNodeIpPoolStart "172.16.10.0" -k8sNodeIpPoolEnd "172.16.10.255" -vipPoolStart "172.16.255.0" -vipPoolEnd
"172.16.255.254" -ipAddressPrefix "172.16.0.0/16" -gateway "172.16.0.1" -dnsServers "172.16.0.1"
```

> [!NOTE]
> The values given in this example command will need to be customized for your environment.

## Step 3: Configure your deployment

Set the configuration settings for the Azure Kubernetes Service host using the [set-akshciconfig](./set-akshciconfig.md) command. **If you're deploying on a 2-4 node Azure Stack HCI cluster or a Windows Server 2019 Datacenter failover cluster, you must specify the `imageDir` and `cloudConfigLocation` parameters.** If you want to reset your config details, run the command again with new parameters.

Configure your deployment with the following command.

```powershell
Set-AksHciConfig -imageDir c:\clusterstorage\volume1\Images -cloudConfigLocation c:\clusterstorage\volume1\Config -vnet $vnet -enableDiagnosticData -cloudservicecidr "172.16.10.10/16" 
```

> [!NOTE]
> The value for `-cloudservicecidr` given in this example command will need to be customized for your environment.

Use the `-enableDiagnosticData` parameter to consent to send required diagnostic data to Microsoft. If this parameter is not used, then you will be prompted to enable diagnostic data when `Set-AksHciConfig` is run. `Install-AksHci` will fail if you do not accept the consent prompt when `Set-AksHciConfig` presents it.

## Step 4: Start a new deployment

After you've configured your deployment, you must start it. This will install the Azure Kubernetes Service on Azure Stack HCI agents/services and the Azure Kubernetes Service host.

To begin deployment, run the following command.

```powershell
Install-AksHci
```

### Verify your deployed Azure Kubernetes Service host

To ensure that your Azure Kubernetes Service host was deployed, run the [get-akshcicluster](./get-akshcicluster.md) command. You will also be able to get Kubernetes clusters using the same command after deploying them.

```powershell
Get-AksHciCluster
```

**Output:**
```

Name            : clustergroup-management
Version         : v1.18.14
Control Planes  : 1
Linux Workers   : 0
Windows Workers : 0
Phase           : provisioned
Ready           : True
```

## Access your clusters using kubectl

To access your Azure Kubernetes Service host using `kubectl`, run the [get-akshcicredential](./get-akshcicredential.md) command. This will use the specified cluster's _kubeconfig_ file as the default _kubeconfig_ file for `kubectl`. You can also use this command to access other Kubernetes clusters after they are deployed.

```powershell
Get-AksHciCredential -name clustergroup-management
```

## Get logs

To get logs from your all your pods, run the [get-akshcilogs](./get-akshcilogs.md) command. This command will create an output zipped folder called `akshcilogs` in the path `c:\%workingdirectory%\%AKS HCI release number%\%filename%` (for example, `c:\AksHci\0.9.6.0\akshcilogs.zip`).

```powershell
Get-AksHciLogs
```

## Next steps

- [Create a Kubernetes cluster for your applications](create-kubernetes-cluster-powershell.md)
