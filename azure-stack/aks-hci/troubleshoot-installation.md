---
title: Troubleshoot known issues and errors when installing Azure Kubernetes Service on Azure Stack HCI 
description: Find solutions to known issues and errors when installing Azure Kubernetes Service on Azure Stack HCI 
author: v-susbo
ms.topic: troubleshooting
ms.date: 09/15/2021
ms.author: v-susbo
ms.reviewer: 
---

# Known issues and errors during an AKS on Azure Stack HCI installation

This article describes known issues and errors you may encounter when running an installation of AKS on Azure Stack HCI.

## Install-AksHci failed on a multi-node installation with the error _Nodes have not reached active state_

When running [Install-AksHci](./reference/ps/install-akshci.md) on a single-node setup, the installation worked, but when setting up the failover cluster, the installation fails with the error message _Nodes have not reached active state_. However, pinging the cloud agent showed the CloudAgent was reachable.

To ensure all nodes can resolve the CloudAgent's DNS, run the following command on each node:

```powershell
Resolve-DnsName <FQDN of cloudagent>
```

When the step above succeeds on the nodes, make sure the nodes can reach the CloudAgent port to verify that a proxy is not trying to block this connection and the port is open. To do this, run the following command on each node:

```powershell
Test-NetConnection  <FQDN of cloudagent> -Port <Cloudagent port - default 65000>
```

## Install-AksHci timed out with a _Waiting for API server_ error

After running [Install-AksHci](./reference/ps/install-akshci.md), the installation stopped and displayed the following _waiting for API server_ error message:

```Output
\kubectl.exe --kubeconfig=C:\AksHci\0.9.7.3\kubeconfig-clustergroup-management 
get akshciclusters -o json returned a non zero exit code 1 
[Unable to connect to the server: dial tcp 192.168.0.150:6443: 
connectex: A connection attempt failed because the connected party 
did not properly respond after a period of time, or established connection 
failed because connected host has failed to respond.]
```

There are multiple reasons why an installation might fail with the _waiting for API server_ error. See the following sections for possible causes and solutions for this error.

### Reason 1: Incorrect IP gateway configuration
If you're using static IP and you received the following error message, confirm that the configuration for the IP address and gateway is correct. 
```PowerShell
Install-AksHci 
C:\AksHci\kvactl.exe create –configfile C:\AksHci\yaml\appliance.yaml  --outfile C:\AksHci\kubeconfig-clustergroup-management returned a non zero exit code 1 [ ]
```

To check whether you have the right configuration for your IP address and gateway, run the following: 

```powershell
ipconfig /all
```

In the displayed configuration settings, confirm the configuration. You could also attempt to ping the IP gateway and DNS server. 

If these methods don't work, use [New-AksHciNetworkSetting](./reference/ps/new-akshcinetworksetting.md) to change the configuration.

### Reason 2: Incorrect DNS server
If you’re using static IP, confirm that the DNS server is correctly configured. To check the host's DNS server address, use the following command:

```powershell
Get-NetIPConfiguration.DNSServer | ?{ $_.AddressFamily -ne 23} ).ServerAddresses
```

Confirm that the DNS server address is the same as the address used when running `New-AksHciNetworkSetting` by running the following command:

```powershell
Get-MocConfig
```

If the DNS server has been incorrectly configured, reinstall AKS on Azure Stack HCI with the correct DNS server. For more information, see [Restart, remove, or reinstall Azure Kubernetes Service on Azure Stack HCI ](./restart-cluster.md).

The issue was resolved after deleting the configuration and restarting the VM with a new configuration.

## Install-AksHci failed with a _Failed to wait for addon arc-onboarding_ error

After running [Install-AksHci](./reference/ps/install-akshci.md), a _Failed to wait for addon arc-onboarding_ error occurred.

To resolve this issue, use the following steps:

1. Open PowerShell and run [Uninstall-AksHci](./reference/ps/uninstall-akshci.md).
2. Open the Azure portal and navigate to the resource group you used when running `Install-AksHci`.
3. Check for any connected cluster resources that appear in a _Disconnected_ state and include a name shown as a randomly generated GUID. 
4. Delete these cluster resources.
5. Close the PowerShell session and open new session before running `Install-AksHci` again.

## After a failed installation, running Install-AksHci does not work
If your installation fails using [Install-AksHci](./reference/ps/uninstall-akshci.md), you should run [Uninstall-AksHci](./reference/ps/uninstall-akshci.md) before running `Install-AksHci` again. This issue happens because a failed installation may result in leaked resources that have to be cleaned up before you can install again.

## During deployment, the error _Waiting for pod ‘Cloud Operator’ to be ready_ appears
When attempting to deploy an AKS on Azure Stack HCI cluster on an Azure VM, the installation was stuck at _Waiting for pod 'Cloud Operator' to be ready..._, and then failed and timed out after two hours. Attempts to troubleshoot by checking the gateway and DNS server showed they were working appropriately. Checks to see if there was an IP or MAC address conflict showed none were found. When viewing the logs, it showed that the VIP pool had not reached the logs. There was a restriction on pulling the container image using `sudo docker pull ecpacr.azurecr.io/kube-vip:0.3.4` that returned a Transport Layer Security (TLS) timeout instead of _unauthorized_. 

To resolve this issue, run the following steps:

1. Start to deploy your cluster.
2. When deployed, connect to management cluster VM through SSH as shown below:

   ```
   ssh -i (Get-MocConfig)['sshPrivateKey'] clouduser@<IP Address>
   ```

3. Change the maximum transmission unit (MTU) setting. Don't hesitate to make the change because if you make the change too late, then the deployment fails. Modifying the MTU setting helps unblock the container image pull.

   ```
   sudo ifconfig eth0 mtu 1300
   ```

4. To view the status of your containers, run the following command:
   ```
   sudo docker ps -a
   ```

After performing these steps, the container image pull should be unblocked.

## When running the Set-AksHciRegistration command, the error _Unable to check registered Resource Providers_ appears

After running [Set-AksHciRegistration](./reference/ps/set-akshciregistration.md) in an AKS on Azure Stack HCI installation, an _Unable to check registered Resource Providers_ error is displayed. This error indicates the Kubernetes Resource Providers are not registered for the tenant that is currently logged in.

To resolve this issue, run either the Azure CLI or the PowerShell steps below:

```azurecli
az provider register --namespace Microsoft.Kubernetes
az provider register --namespace Microsoft.KubernetesConfiguration
```

```powershell
Register-AzResourceProvider -ProviderNamespace Microsoft.Kubernetes
Register-AzResourceProvider -ProviderNamespace Microsoft.KubernetesConfiguration
```

The registration takes approximately 10 minutes to complete. To monitor the registration process, use the following commands.

```azurecli
az provider show -n Microsoft.Kubernetes -o table
az provider show -n Microsoft.KubernetesConfiguration -o table
```

```powershell
Get-AzResourceProvider -ProviderNamespace Microsoft.Kubernetes
Get-AzResourceProvider -ProviderNamespace Microsoft.KubernetesConfiguration
```

## Install-AksHci failed with the error _Install-Moc failed. Logs are available C:\Users\xxx\AppData\Local\Temp\v0eoltcc.a10_

When configuring an AKS on Azure Stack HCI environment, running [Install-AksHci](./reference/ps/install-akshci.md) resulted in the error _Install-Moc failed. Logs are available C:\Users\xxx\AppData\Local\Temp\v0eoltcc.a10_.

To get more information on the error, run `$error[0].Exception.InnerException`. 

## When creating a new workload cluster, the error _Error: rpc error: code = DeadlineExceeded desc = context deadline exceeded_ occurs

This is a known issue with the AKS on Azure Stack HCI July Update (version 1.0.2.10723). The error _Error: rpc error: code = DeadlineExceeded desc = context deadline exceeded_ occurs because the CloudAgent service times out during distribution of virtual machines across multiple cluster shared volumes in the system.

## Next steps

- [Known issues](./known-issues.md)
- [Troubleshoot Windows Admin Center](./troubleshoot-windows-admin-center.md)
- [Troubleshooting Kubernetes clusters](https://kubernetes.io/docs/tasks/debug-application-cluster/troubleshooting/)
- [Connect with SSH to Windows or Linux worker nodes](./ssh-connection.md)