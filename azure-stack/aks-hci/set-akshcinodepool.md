---
title: Set-AksHciNodePool
author: jessicaguan
description: The Set-AksHciNodePool PowerShell command scales a node pool
ms.topic: reference
ms.date: 7/20/2021
ms.author: jeguan
---

# Set-AksHciNodePool

## Synopsis
Scale a node pool within a Kubernetes cluster.

## Syntax
```powershell
Set-AksHciNodePool -clusterName <String>
                   -name <String>
                   -count <int>
```

## Description
Scale a node pool within a Kubernetes cluster.

## Example

```powershell
PS C:\> Set-AksHciNodePool -clusterName mycluster -name nodepool1 -count 3
```


### -clusterName
The name of the Kubernetes cluster.

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

### -name
The name of your node pool.

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

### -count
The number of nodes to scale to.

```yaml
Type: System.Int32
Parameter Sets: (All)
Aliases:

Required: True
Position: Named
Default value: None
Accept pipeline input: False
Accept wildcard characters: False
```

