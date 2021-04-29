---
title: Add-AksHciNode
author: jessicaguan
description: The Add-AksHciNode PowerShell command adds a new physical node to a deployment.
ms.topic: reference
ms.date: 4/16/2021
ms.author: jeguan
---

# Add-AksHciNode

## Synopsis
Add a new physical node to a deployment.

## Syntax

```powershell
Add-AksHciNode -nodeName <String>
```

## Description
Add a new physical node to your deployment.

## Examples

### Add a new physical node
```powershell
PS C:\> Add-AksHciNode -nodeName newnode
```

## Parameters

### -nodeName
The name of your cluster.

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