---
title: Prepare Windows nodes for group Managed Service Account support
description: Learn how to configure group Managed Service Accounts for containers on Windows nodes
author: v-susbo
ms.topic: how-to
ms.date: 11/30/2020
ms.author: v-susbo
---

# Prepare Windows nodes for group Managed Service Account support

> Applies to: AKS on Azure Stack HCI, AKS runtime on Windows Server 2019 Datacenter

Group Managed Service Accounts are a specific type of Active Directory account that provides:
- Automatic password management
- Simplified service principal name (SPN) management
- The ability to delegate the management to other administrators across multiple servers 

To configure group Managed Service Accounts (gMSA) for pods and containers that will run on your Windows nodes, you first have to join your Windows nodes to an Active Directory domain.

To enable group Managed Service Account support, your Kubernetes cluster name has to be fewer than four characters. It needs to be fewer than four characters because the maximum supported length for a domain joined server name is 15 characters. The AKS on Azure Stack HCI Kubernetes cluster name convention for a worker node adds a few pre-defined characters to a node name.

## Join the worker nodes to a domain

To join your Windows worker nodes to a domain, log in to a Windows worker node by running `kubectl get` and noting the `EXTERNAL-IP` value.

```
kubectl get nodes -o wide
```  

You can then SSH into the node using `ssh Administrator@ip`. For more information on how to log in using SSH, see [Troubleshoot Windows worker nodes using SSH](troubleshoot.md#troubleshoot-windows-worker-nodes).

After you've successfully logged in to your Windows worker node using SSH, run the following command to join the node to a domain. Then, you need to reboot the Windows worker node. 

```
netdom.exe join %computername% /domain:DomainName /UserD:DomainName\UserName /PasswordD:Password
```  

Once all Windows worker nodes have been joined to a domain, follow the steps detailed at [configuring gMSA](https://kubernetes.io/docs/tasks/configure-pod-container/configure-gmsa/). Those steps will help you apply the Kubernetes gMSA custom resource definitions and webhooks on your Kubernetes cluster.

For more information on Windows container with gMSA, see [Windows containers and gMSA](/virtualization/windowscontainers/manage-containers/manage-serviceaccounts). 

## Next steps

* [Use Azure Monitor to monitor your cluster and application](/azure/azure-monitor/insights/container-insights-enable-arc-enabled-clusters).
* [Use persistent volume on a Kubernetes cluster](persistent-volume.md).