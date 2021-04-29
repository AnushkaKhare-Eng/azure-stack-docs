---
title: Get-AksHciConfig
author: jessicaguan
description: The Get-AksHciConfig PowerShell command lists the current configuration settings for the Azure Kubernetes Service host.
ms.topic: reference
ms.date: 2/12/2021
ms.author: jeguan
---

# Get-AksHciConfig

## Synopsis
List the current configuration settings for the Azure Kubernetes Service host.

## Syntax

```powershell
Get-AksHciConfig | ConvertTo-Json
```

## Description
List the current configuration settings for the Azure Kubernetes Service host. If the `ConvertTo-Json` cmdlet is not included when running `Get-AksHciConfig`, the values will not be returned and look like the following.

Name                           Value
----                           -----
MOC                            {catalog, ipaddressprefix, cloudServiceCidr, deploymentType...}
AksHci                         {manifestCache, enableDiagnosticData, installationPackageDir, workingDir...}
Kva                            {catalog, vlanid, vnetvippoolend, ring...}

When `ConvertTo-Json` is included, the configuration values will be returned as JSON objects.

## Examples

### Example 
```powershell
PS C:\> Get-AksHciConfig | ConvertTo-Json
```


