---
title: Get-AksHciStorageContainer
author: jessicaguan
description: The Get-AksHciStorageContainer PowerShell command returns the Storage Container name and location for the specified Storage Container
ms.topic: reference
ms.date: 3/3/2021
ms.author: jeguan
---

# Get-AksHciContainerStorage

## Synopsis
Get the information of the specified storage container.

## Syntax

```powershell
Get-AksHciStorageContainer [-name <String>]
```

## Description
Get the information of the specified Storage Container. If the Storage Container is not specified, the command will return all the Storage Containers.

## Examples

### Example
```powershell
PS C:\> Get-AksHciStorageContainer -name mystoragecontainer
```

## Parameters

### -name
The name of the new Storage Container. 

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
