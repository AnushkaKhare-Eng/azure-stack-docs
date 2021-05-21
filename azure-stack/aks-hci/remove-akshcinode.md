---
title: Remove-AksHciNode
author: jessicaguan
description: The Remove-AksHciNode PowerShell command removes a physical node from your deployment.
ms.topic: reference
ms.date: 4/16/2021
ms.author: jeguan
---

# Remove-AksHciNode

## Synopsis
Remove a physical node from your deployment.

## Syntax

```powershell
Remove-AksHciNode -nodeName <String>
```

## Description
Remove a physical node from your deployment. You can use this command in the case that a physical node fails.

## Examples

### Add a new physical node
```powershell
PS C:\> Remove-AksHciNode -nodeName newnode
```

## Parameters

### -nodeName
The name of the node in the failover cluster to be removed.

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