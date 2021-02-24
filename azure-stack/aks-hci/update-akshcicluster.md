---
title: Update-AksHciCluster
author: jessicaguan
description: The Update-AksHciCluster PowerShell command updates a managed Kubernetes cluster to a newer Kubernetes or OS version.
ms.topic: reference
ms.date: 2/12/2021
ms.author: jeguan
---

# Update-AksHciCluster

## Synopsis
Update a managed Kubernetes cluster to a newer Kubernetes or OS version.

## Syntax

```powershell
Update-AksHciCluster -name <String>
                    [-kubernetesVersion <String>]
                    [-operatingSystem]
```

## Description
Update a managed Kubernetes cluster to a newer Kubernetes or OS version.

## Examples

### Example
```powershell
PS C:\> Update-AksHciCluster -clusterName myCluster -kubernetesVersion v1.18.8 -operatingSystem
```

## Parameters

### -name
The name of your Kubernetes cluster.

```yaml
Type: System.String
Parameter Sets: (All)
Aliases:

Required: True
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

### -kubernetesVersion
The version of Kubernetes you want to upgrade to.

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

### -operatingSystem
Use this flag if you want to update to the next version of the operating system.

```yaml
Type: System.Management.Automation.SwitchParameter
Parameter Sets: (All)
Aliases:

Required: False
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

