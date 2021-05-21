---
title: Install-AksHciCsiNfs
author: jessicaguan
description: The Install-AksHciCsiNfs PowerShell command Installs CSI NFS Plugin to a cluster
ms.topic: reference
ms.date: 4/13/2021
ms.author: jeguan
---

# Install-AksHciCsiNfs

## Synopsis
Installs the CSI NFS plug-in to a cluster.

## Syntax

```powershell
Install-AksHciCsiNfs -clusterName <String>                       
```

## Description
Installs CSI NFS Plugin to a cluster.

## Examples

### Example

```PowerShell
Install-AksHciCsiNfs -clusterName mycluster
```

## Parameters

### -clusterName
The alphanumeric name of your Kubernetes cluster.

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
