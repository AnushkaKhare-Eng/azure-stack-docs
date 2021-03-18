---
title: Restart, remove, or reinstall Azure Kubernetes Service on Azure Stack HCI 
description: Learn how to restart, remove, or reinstall Azure Kubernetes Service on Azure Stack
author: jessicaguan
ms.topic: article
ms.date: 03/02/2021
ms.author: jeguan
---

# Restart, remove, or reinstall Azure Kubernetes Service on Azure Stack HCI

After deploying Azure Kubernetes Service on Azure Stack HCI, you can restart, remove, or reinstall your deployment, if necessary.

## Restart Azure Kubernetes Service on Azure Stack HCI

Restarting Azure Kubernetes Service on Azure Stack HCI will remove all of your Kubernetes clusters (if any) and the Azure Kubernetes Service host. The restart process will also uninstall the Azure Kubernetes Service on Azure Stack HCI agents and services from the nodes. Then, it will go back through the original install process steps until the host is recreated. The Azure Kubernetes Service on Azure Stack HCI configuration that you configured via [Set-AksHciConfig](./set-akshciconfig.md) and the downloaded VHDX images are preserved. The `Set-AksHciConfig` command will remove the current VMs and create new ones.

To restart Azure Kubernetes Service on Azure Stack HCI with the same configuration settings, run the following command.

```powershell
Restart-AksHci
```

## Remove Azure Kubernetes Service on Azure Stack HCI

To remove Azure Kubernetes Service on Azure Stack HCI, run the following [Uninstall-AksHci](./uninstall-akshci.md) command. This command will remove the old configuration, and you will have to run [Set-AksHciConfig](./set-akshciconfig.md) again when you reinstall. If your clusters are Arc-enabled, delete any Azure resources before proceeding. To delete any associated Arc resources for your on-premises cluster, follow the guidance for [cleaning up Azure Arc resources](https://docs.microsoft.com/azure/azure-arc/kubernetes/quickstart-connect-cluster#clean-up-resources).

```powershell
Uninstall-AksHci
``` 

If you want to retain the old configuration, run the following command:

```powershell
Uninstall-AksHci -SkipConfigCleanup
```

## Reinstall configuration settings and reinstall Azure Kubernetes Service on Azure Stack HCI

To reinstall Azure Kubernetes Service on Azure Stack HCI after uninstalling it, follow the instructions below.

If you ran the `Uninstall-AksHci` command with the `-SkipConfigCleanup` parameters, your old configuration settings were retained. To reinstall, run the following command.

```powershell
Install-AksHci
```

If you didn't use the `-SkipConfigCleanup` parameter when uninstalling, then you will need to reset your configuration settings with the commands below. This example command creates a virtual network with a static IP. If you want to configure your AKS deployment with DHCP, see [new-akshcinetworksetting](.\new-akshcinetworksetting.md) for examples on how to configure DHCP.


```powershell
#static IP
$vnet = New-AksHciNetworkSetting -vnetName "extSwitch" -k8sNodeIpPoolStart "172.16.10.0" -k8sNodeIpPoolEnd "172.16.10.255" -vipPoolStart "172.16.255.0" -vipPoolEnd
"172.16.255.254" -ipAddressPrefix "172.16.0.0/16" -gateway "172.16.0.1" -dnsServers "172.16.0.1"

Set-AksHciConfig -imageDir c:\clusterstorage\volume1\Images -cloudConfigLocation c:\clusterstorage\volume1\Config -vnet $vnet -enableDiagnosticData -cloudservicecidr "172.16.10.10/16"

Install-AksHci
```

## Next steps

In this article, you learned how to restart, remove, or reinstall Azure Kubernetes Service on Azure Stack HCI. Next, you can:
- [Deploy a Linux applications on a Kubernetes cluster](./deploy-linux-application.md).
- [Deploy a Windows Server application on a Kubernetes cluster](./deploy-windows-application.md).
