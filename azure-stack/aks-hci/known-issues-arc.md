---
title: Resolve errors when enabling and disabling Azure Arc on AKS on Azure Stack HCI workload clusters
description: Find solutions to known issues and errors when enabling and disabling Azure Arc on AKS on Azure Stack HCI workload clusters
author: abha
ms.topic: troubleshooting
ms.date: 09/30/2021
ms.author: v-susbo
ms.reviewer: 
---

# Resolve errors when enabling or disabling Azure Arc on your AKS workload clusters

This article describes errors you may encounter (and their workarounds) while connecting or disconnecting your AKS on Azure Stack HCI workload clusters to Azure Arc using the PowerShell cmdlets [Enable-AksHciArcConnection](./reference/ps/enable-akshciarcconnection.md) and [Disable-AksHciArcConnection](./reference/ps/disable-akshciarcconnection.md). For issues that are not covered in this article, see [troubleshooting Arc enabled Kubernetes](/azure/azure-arc/kubernetes/troubleshooting).

You can also [open a support issue](./help-support.md) if none of the workarounds listed below apply to you.

## Error: `addons.msft.microsoft "demo-arc-onboarding" already exists`

This error usually means that you have already connected your AKS on Azure Stack HCI cluster to Arc enabled Kubernetes. To confirm it's connected, go to the [Azure portal](https://portal.azure.com) and check under the subscription and resource group you provided when running [Set-AksHciRegistration](./reference/ps/set-akshciregistration.md) (if you've used default values) or [Enable-AksHciArcConnection](./reference/ps/enable-akshciarcconnection.md) (if you haven't used default values). You can also confirm if your AKS on Azure Stack HCI cluster is connected to Azure by running the [az connectedk8s show ](/cli/azure/connectedk8s?view=azure-cli-latest#az_connectedk8s_show&preserve-view=true) Azure CLI command. If you do not see your workload cluster, run `Disable-AksHciArcConnection` and try again.

## Error: `Connection to Azure failed. Please run 'Set-AksHciRegistration' and try again`

This error means that your login credentials to Azure have expired. Use [Set-AksHciRegistration](./reference/ps/set-akshciregistration.md) to log in to Azure before running the [Enable-AksHciArcConnection](./reference/ps/enable-akshciarcconnection.md) command again. When rerunning `Set-AksHciRegistration`, make sure you use the same subscription and resource group details you used when you first registered the AKS host to Azure for billing. If you rerun the command with a different subscription or resource group, they will not be registered. Once the subscription and resource group are set in `Set-AksHciRegistration`, they cannot be changed without uninstalling AKS on Azure Stack HCI.


## Error: `System.Management.Automation.RemoteException Starting onboarding process Cluster "azure-arc-onboarding" set...`

The following error may occur when you use Windows Admin Center to create a workload cluster and connect it to Arc enabled Kubernetes:

`System.Management.Automation.RemoteException Starting onboarding process Cluster "azure-arc-onboarding" set. User "azure-arc-onboarding" set. Context "azure-arc-onboarding" created. Switched to context "azure-arc-onboarding". Azure login az login: error: argument --password/-p: expected one argument usage: az login [-h] [--verbose] [--debug] [--only-show-errors] [--output {json,jsonc,yaml,yamlc,table,tsv,none}] [--query JMESPATH] [--username USERNAME] [--password PASSWORD] [--service-principal] [--tenant TENANT] [--allow-no-subscriptions] [-i] [--use-device-code] [--use-cert-sn-issuer] : Job Failed Condition]`

To resolve this issue, review the options below:

- Option 1: Delete the workload cluster and try again using Windows Admin Center. 
- Option 2: In PowerShell, check if the cluster has been successfully created by running the [Get-AksHciCluster](./reference/ps/get-akshcicluster.md) command, and then use [Enable-AksHciArcConnection](./reference/ps/enable-akshciarcconnection.md) to connect your cluster to Arc.

## Error: `Timed out waiting for the condition`

This error usually points to one of the following issues:

- The clusters were created in an Azure VM in a virtualized environment, or if you're deploying AKS on Azure Stack HCI on multiple levels of virtualization. 
- Due to a slow internet.

If one of the above scenarios applies to you, run [Disable-AksHciArcConnection](./reference/ps/disable-akshciarcconnection.md) and try connecting again. If the above scenario doesn't apply to you,  [open a support issue](./help-support.md) on AKS on Azure Stack HCI.


## Error: `Azure subscription is not properly configured`

You may encounter this issue if you have not configured your Azure subscription with the Arc enabled Kubernetes' resource providers. We currently check that `Microsoft.Kubernetes` and `Microsoft.KubernetesConfiguration` are configured. For more information on enabling these resource providers, see [enabling resource providers for Arc enabled Kubernetes](/azure/azure-arc/kubernetes/quickstart-connect-cluster?tabs=azure-cli#1-register-providers-for-azure-arc-enabled-kubernetes).


## Error: `A workload cluster with the name 'my-aks-cluster' was not found`

This error means that you have not created the workload cluster, or you have incorrectly spelled the name of the workload cluster. Run [Get-AksHciCluster](./reference/ps/get-akshcicluster.md) to confirm you have the correct name or that the cluster you want to connect to Arc exists.

## Error: `'My-Cluster' is not a valid cluster name. Names must be lower-case and match the regular expression pattern: '^[a-z0-9][a-z0-9-]*[a-z0-9]$'`

This error indicates that the workload cluster does not follow the Kubernetes naming convention. As the error suggests, make sure the cluster name is lower-case and matches the regular expression pattern: '^[a-z0-9][a-z0-9-]*[a-z0-9]$'.

## Error: `Cluster addons arc uninstall Error: namespaces "azure-arc" not found`

This error usually means that you have already uninstalled Arc agents from your workload cluster, or you have manually deleted the `azure-arc` namespace using the `kubectl` command. Go to the [Azure portal](https://portal.azure.com) to confirm that you do not have any leaked resources. For example, verify that you do not see a `connectedCluster` resource in the subscription and resource group.

## Error: `Secrets "sh.helm.release.v1.azure-arc.v1" not found`

This error indicates that your Kubernetes API server could not be reached. Try running the [Disable-AksHciArcConnection](./reference/ps/disable-akshciarcconnection.md) command again, and then go to the [Azure portal](https://portal.azure.com) to confirm that your `connectedCluster` resource has actually been deleted. You can also run `kubectl get ns -a` to confirm that the namespace, `azure-arc`, does not exist on your cluster.

## Error: `autorest/azure: Service returned an error. Status=404 Code="ResourceNotFound"...`

The error below means that Azure could not find the `connectedCluster` ARM resource associated with your cluster:

`autorest/azure: Service returned an error. Status=404 Code="ResourceNotFound" Message="The Resource 'Microsoft.Kubernetes/connectedClusters/my-workload-cluster' under resource group 'AKS-HCI2' was not found. For more details please go to https://aka.ms/ARMResourceNotFoundFix"]`

You may encounter this error if: 

- You supplied an incorrect resource group or subscription while running the `Disable-AksHciArcConnection` cmdlet. 
- You manually deleted the resource on the Azure portal.
- ARM cannot find your Azure resource.

To resolve this error, as indicated in the error message, see [resolve resource not found errors](/azure/azure-resource-manager/templates/error-not-found).

## Enable-AksHciArcConnection fails if Connect-AzAccount is used to log in to Azure

When you use [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount?view=azps-6.4.0&preserve-view=true) to log in to Azure, you might end up setting a different subscription as your default context than the one that you gave as an input to [Set-AksHciRegistration](./reference/ps/set-akshciregistration.md). When you then run [Enable-AksHciArcConnection](./reference/ps/enable-akshciarcconnection.md), the command expects the subscription used in `Set-AksHciRegistration`. However, `Enable-AksHciArcConnection` gets the default subscription set using the `Connect-AzAccount`, and therefore, might cause an error. To prevent this error, follow one of the options below:

- Option 1: Run `Set-AksHciRegistration` to log in to Azure with the same parameters (subscription and resource group) you used when you first ran the command to connect your AKS host to Azure for billing. You can then use `Enable-AksHciArcConnection -Name <ClusterName>` with default values, and your cluster will be connected to Arc under the AKS host billing subscription and resource group. 

- Option 2: Run `Enable-AksHciArcRegistration` with all parameters, `subscription`, `resource group`, `location`, `tenant`, and `secret`, to connect your cluster to Azure Arc under a different subscription and resource group than the AKS host. You should also run `Enable-AksHciArcRegistration` if you do not have enough permissions to connect your cluster to Azure Arc using your Azure Account (for example, if you are not the subscription owner).

## Next steps

- [Troubleshooting Arc enabled Kubernetes](/azure/azure-arc/kubernetes/troubleshooting)
- [Known issues](./known-issues.md)