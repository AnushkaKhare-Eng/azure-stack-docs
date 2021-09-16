---
title: Create and delete pods in Azure Kubernetes Service on Azure Stack HCI
description: Learn how to create and delete pods in Azure Kubernetes Service (AKS) on Azure Stack HCI.
author: EkeleAsonye
ms.topic: how-to
ms.date: 09/08/2021
ms.author: v-susbo
---

# Create and delete pods

Kubernetes uses pods to run an instance of your application, and a pod represents a single instance of an application. Each pod has one or more containers deployed together on a single host and is the smallest unit of execution in Kubernetes. An internal IP address and port are assigned to a pod through which containers within the pod can share a common storage and network. Like a service, volume, and namespace, a pod is a basic Kubernetes object. Pods run on nodes and have a definite lifecycle where they run until their container(s) is removed.

## Create a pod

Before creating a pod, you must [set up an AKS host and create AKS on Azure Stack HCI clusters using Windows PowerShell](./kubernetes-walkthrough-powershell.md). You can also use Windows Admin Center to [set up the host](./setup.md) and [create the clusters](./create-kubernetes-cluster.md).

To make sure you are connected to the Kubernetes cluster, run the following command:

```powershell
kubectl get nodes
```

To create a pod, run the following command. In this example, a pod is created using an nginx image.  

```powershell
kubectl run nginx –image=nginx --restart=Never
```

When you set the parameter `–restart=Never`, Kubernetes creates a single pod, instead of creating a deployment.

To see the status of your pod, run the following command:

```powershell
kubectl get pods
```

To view the entire configuration of the pod, run the following command:

```powershell
kubectl describe pod nginx
```

### Secure traffic between pods

An important aspect in Kubernetes is to ensure consistent policy enforcement across multiple clusters. To learn more about verifying connectivity between the pods and then applying a basic network policy for pod traffic, see [Secure traffic between pods using network policies](./calico-networking-policy.md).

## Delete a pod

To delete a pod you created, run the following command:

```powershell
kubectl delete pod
```

## Example pod configuration

The following YAML example describes the features of a pod, and shows how it's the smallest unit of Kubernetes that can be defined, deployed, and managed.

```yml
apiVersion: v1 
kind: Pod 
metadata: 
      labels: 
         app: nginx 
      name: nginx 
      namespace: calico-demo 
spec: 
      containers: 
      - name: nginx 
         image: nginx:1.8 
         ports: 
         - containerPort: 80 
      nodeSelector: 
          beta.kubernetes.io/os: linux
```

## Next steps

[Create a deployment](create-deployments.md)
