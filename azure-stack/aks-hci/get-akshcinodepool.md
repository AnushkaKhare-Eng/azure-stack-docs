---
title: Get-AksHciNodePool
author: jessicaguan
description: The Get-AksHciNodePool PowerShell command lists the node pools in a Kubernetes cluster.
ms.topic: reference
ms.date: 7/20/2021
ms.author: jeguan
---

# Get-AksHciNodePool

## Synopsis
Lists the node pools in a Kubernetes cluster.

## Syntax

```powershell
Get-AksHciNodePool -clusterName <String>
                  [-name <String>]
```

## Description
Lists the node pools in a Kubernetes cluster.

## Examples

### List all node pools in a Kubernetes cluster
```powershell
PS C:\> Get-AksHCiNodePool -clusterName mycluster
```

### List a specific node pool in a Kubernetes cluster
```powershell
PS C:\> Get-AksHciNodePool -clusterName mycluster -name nodepool1
```

## Parameters

### -clusterName
The name of your cluster.

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -name
The name of the node pool.

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```
