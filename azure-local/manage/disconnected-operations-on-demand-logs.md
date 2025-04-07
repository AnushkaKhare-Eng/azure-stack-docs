---
title: Collect logs on-demand with Azure Local disconnected operations (preview)
description: Use PowerShell module to collect logs on demand for Azure Local disconnected operations (preview).
ms.topic: how-to
author: ronmiab
ms.author: robess
ms.date: 03/19/2025
---

# Collect logs on demand with Azure Local disconnected operations PowerShell Module

::: moniker range=">=azloc-24112"

[!INCLUDE [applies-to](../includes/release-2411-1-later.md)]

This document helps you connect with support and provide logs for troubleshooting issues when Azure Local is operating in a disconnected mode.

[!INCLUDE [IMPORTANT](../includes/disconnected-operations-preview.md)]

## Overview

On-demand log collection lets you collect logs from Azure Local disconnected operations for troubleshooting purposes. This feature is useful when you need to provide logs to Microsoft support for troubleshooting issues.

## Supported scenarios

The following on-demand scenarios are supported for log collection:

| Scenarios for log collection             | How to collect logs                    |
|------------------------------------------|----------------------------------------|
| [Use on-demand direct log collection](#log-collection-when-connected-to-azure-with-an-accessible-management-endpoint) when an on-premises device with Azure Local disconnected operations is connected to Azure and the management endpoint for disconnected operations is accessible. | Trigger log collection with cmdlet `Invoke-ApplianceLogCollection`.<br></br>**Prerequisite**: Setup observability configuration using the cmdlet: `Set-ApplianceObservabilityConfiguration`. |
| [Use on-demand indirect log collection](#log-collection-for-a-disconnected-environment-and-accessible-management-endpoint) when an on-premises device using Azure Local disconnected operations doesn't have a connection to Azure, but the management endpoint for disconnected operations is accessible. | Trigger log collection with cmdlet `Invoke-ApplianceLogCollectionAndSaveToShareFolder`.<br></br> After the `Invoke-ApplianceLogCollectionAndSaveToShareFolder` step, use the `Send-DiagnosticData` cmdlet to upload the copied data logs from the file share to Microsoft.  |
| [Use on-demand fallback log collection](#log-collection-with-an-inaccessible-management-endpoint) when the management endpoint for disconnected operations isn't accessible or the integrated runtime disconnected operations with Azure Local virtual machine (VM) is down. | Logs are collected after shutting down the disconnected operations with Azure Local VM, mounting and unlocking virtual hard disks (VHDs) and copying logs using the `Copy-DiagnosticData` cmdlet from mounted VHDs into a local, user-defined location.<br></br> Use the `Send-DiagnosticData` cmdlet to manually send diagnostic data to Microsoft. |

## Trigger on-demand log collection

To trigger on demand log collection in Azure Local disconnected operations, use the following cmdlets with the parameters FromDate and ToDate in PowerShell:

- `Invoke-ApplianceLogCollection`
- `Invoke-ApplianceLogCollectionAndSaveToShareFolder`
- `Get-ApplianceLogCollectionHistory`
- `Get-ApplianceLogCollectionJobStatus`

> [!NOTE]
> Run these commands on the host that can access the management endpoint.

## Triage Azure Local issues

Use the following cmdlets and references to triage Azure Local issues.

- `AzsSupportDataBundle`. For more information, see [Azure Local Support Diagnostic Tool](/azure/azure-local/manage/support-tools).
- `Send-AzStackHciDiagnosticData`. For more information, see [Get support for Azure Local deployment issues](/azure/azure-local/manage/get-support-for-deployment-issues).
- `Get-SDDCDiagnosticInfo` and upload it to customer service and support (CSS) data transfer manager (DTM) share. For more information, see [Collect diagnostic data for clusters](/azure/azure-local/manage/collect-diagnostic-data).

## Prerequisites

There are a few prerequisites you need to perform log collection in a connected disconnected operations scenario.

- Use [Deploy disconnected operations for Azure Local (Preview)](disconnected-operations-deploy.md) to set up the following Azure resources:
  - A resource group in Azure used for the appliance.
  - A Service Principal (SPN) with contributor rights on the resource group.
    - Copy the `AppId` and `Password` form the output. Use them as **ServicePrincipalId** and **ServicePrincipalSecret** during observability setup.

- Install the operations module if it's not installed. Use the `Import-Module` cmdlet and modify the path to match your folder structure.

    Here's an example output:

    ```console
    PS C:\Users\administrator.s46r2004\Documents> Import-Module "Q:\AzureLocalVHD\OperationsModule\Azure.Local.DisconnectedOperations.psd1" -Force  
    VERBOSE: [2025-03-26 19:49:12Z][Test-RunningRequirements] PSVersionTable:  
      
    Name                           Value  
    ----                           -----  
    PSVersion                      5.1.26100.2161  
    PSEdition                      Desktop  
    PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}  
    BuildVersion                   10.0.26100.2161  
    CLRVersion                     4.0.30319.42000  
    WSManStackVersion              3.0  
    PSRemotingProtocolVersion      2.3  
    SerializationVersion           1.1.0.1  
      
    VERBOSE: See Readme.md for directions on how to use this module.
    ```

- Management endpoint
  - Identify your management endpoint IP address.
  - Identify the management client certificate used to authenticate with the Azure Local disconnected operations management endpoint.
  - Set up the management endpoint client context. Run this script:

    ```azurecli
    $certPasswordPlainText = "***"
    $certPassword = ConvertTo-SecureString $certPasswordPlainText -AsPlainText -Force
    $context = Set-DisconnectedOperationsClientContext -ManagementEndpointClientCertificatePath "<Management Endpoint Client Cert Path>" -ManagementEndpointClientCertificatePassword $certPassword -ManagementEndpointIpAddress "<Management Endpoint IP address>"
    ```

    Here's example output:

    ```console
    $recoveryKeys = Get-ApplianceBitlockerRecoveryKeys $context # context can be omitted if context is set.
    $recoveryKeys
    ```

- Retrieve BitLocker keys:
  
    ```azurecli
    $recoveryKeys = Get-ApplianceBitlockerRecoveryKeys $context # context can be omitted if context is set.
    $recoveryKeys
    ```

    Here's the example output:

    ```console
    PS C:\Users\administrator.s46r2004\Documents> $recoverykeySet = Get-ApplianceBitlockerRecoveryKeys  
    >> $recoverykeySet | ConvertTo-JSON > c:\recoveryKey.json  
    VERBOSE: [2025-03-26 21:57:01Z][Get-ApplianceBitlockerRecoveryKeys] [START] Get bitlocker recovery keys.
    VERBOSE: [2025-03-26 21:57:01Z][Invoke-ScriptsWithRetry] Executing 'Script Block' with timeout 300 seconds ...
    VERBOSE: [2025-03-26 21:57:01Z][Invoke-ScriptsWithRetry] [CHECK][Attempt 0] for task 'Script Block' ...
    VERBOSE: [2025-03-26 21:57:01Z][Invoke-ScriptsWithRetry] Task 'Script Block' succeeded.
    VERBOSE: [2025-03-26 21:57:01Z][Get-ApplianceBitlockerRecoveryKeys] [END] Get bitlocker recovery keys.
    PS C:\Users\administrator.s46r2004\Documents> Get-content c:\recoveryKey.json
    ```

> [!NOTE]
> On each deployment, the management IP address, management endpoint client certificate, and certificate password are different. Make sure you know the correct values for your deployment.

## Log collection when connected to Azure with an accessible management endpoint

Before you can start log collection and get the Stamp ID, you must have the observability configuration with Azure Local disconnected operations module setup.

1. To check that the observability configuration is set up. Use the Get-`ApplianceObservabilityConfiguration` or `Get-Appliancehealthstate` cmdlet:

    Here's an example output:

    ```console
    PS C:\Users\administrator.s46r2004\Documents> Get-ApplianceHealthState  
    VERBOSE: [2025-03-26 22:43:43Z][Invoke-ScriptsWithRetry] Executing 'Get health state ...' with timeout 300 seconds ...  
    VERBOSE: [2025-03-26 22:43:43Z][Invoke-ScriptsWithRetry] [CHECK][Attempt 0] for task 'Get health state ...' ...  
    VERBOSE: [2025-03-26 22:43:44Z][Invoke-ScriptsWithRetry] Task 'Get health state ...' succeeded.  
      
    SystemReady ReadinessStatusDetails ErrorMessages  
    ----------- ---------------------- -------------  
    False       @{Services=79; Diagnostics=100; Identity=100; Networking=100} @{Services=System.Object[]; Diagnostics=System.Object[]; Identity=System.Object[]; Networking=System.Object[]}  
    ```

    > [!NOTE]
    > If the observability configuration is set up, skip step 2 and proceed to step 3.

2. If the observability configuration isn't set up, run the following script:

    ```azurecli
    $observabilityConfiguration = New-ApplianceObservabilityConfiguration `
      -ResourceGroupName "WinfieldPreview" `
      -TenantId "<Tenant Id>" `
      -Location "eastus" `
      -SubscriptionId "<Subscription Id>" `
      -ServicePrincipalId "<Service Principal Id of the one you prepared for log collection>" `
      -ServicePrincipalSecret (ConvertTO-SecureString "Service Principal secret of the one for log collection" -AsPlainText -Force")
    Set-ApplianceObservabilityConfiguration -ObservabilityConfiguration $observabilityConfiguration
    ```

    Here's an example output:

    ```console
    PS C:\Users\administrator.s46r2004\Documents> $observabilityConfiguration = New-ApplianceObservabilityConfiguration `  
    >> -ResourceGroupName "WinfieldPreviewShip" `  
    >> -ServerId "<Server Id>" `  
    >> -Location "eastus" `  
    >> -SubscriptionId "<Subscription Id>" `  
    >> -ClientId "<Client ID>" `  
    >> -ServicePrincipalSecret (ConvertTo-SecureString "SY5T0_9@_eFlLoSdB1uck_1v_0S0#Mf1O#A0o" -AsPlainText -Force)  
    PS C:\Users\administrator.s46r2004\Documents> Set-ApplianceObservabilityConfiguration -ObservabilityConfiguration $observabilityConfiguration  
    VERBOSE: [2025-03-26 22:34:56Z][ConfigureObservability] [START] Set observability configuration  
    VERBOSE: [2025-03-26 22:34:56Z][Invoke-ScriptsWithRetry] Executing 'Setting up diagnostics ...' with timeout 300 seconds ...  
    VERBOSE: [2025-03-26 22:34:56Z][Invoke-ScriptsWithRetry] [CHECK][Attempt 0] for task 'Setting up diagnostics ...' ...  
    VERBOSE: [2025-03-26 22:34:56Z][ScriptBlock] [INFO] Result of observability configuration: ...  
    VERBOSE: [2025-03-26 22:34:56Z][Invoke-ScriptsWithRetry] Task 'Setting up diagnostics ...' succeeded.  
    VERBOSE: [2025-03-26 22:34:56Z][Invoke-ScriptsWithRetry] Executing 'Wait Diagnostics ready' with timeout 1200 seconds ...  
    VERBOSE: [2025-03-26 22:34:56Z][Invoke-ScriptsWithRetry] [CHECK][Attempt 0] for task 'Wait Diagnostics ready' ...  
    VERBOSE: [2025-03-26 22:34:56Z][Invoke-ScriptsWithRetry] Executing 'Get health state ...' with timeout 300 seconds ...  
    VERBOSE: [2025-03-26 22:34:56Z][Invoke-ScriptsWithRetry] [CHECK][Attempt 0] for task 'Get health state ...' ...  
    VERBOSE: [2025-03-26 22:34:56Z][Invoke-ScriptsWithRetry] Task 'Get health state ...' succeeded.  
    VERBOSE: [2025-03-26 22:34:56Z][ScriptBlock] System Readiness information:  
    SystemReady ReadinessStatusDetails ErrorMessages  
    ----------- ---------------------- -------------  
    False       @{Services=22; Diagnostics=0; Identity=100; Networking=100} @{Services=System.Object[]; Diagnostics=System.Object[]; Identity=System.Object[]; Networking=System.Object[]}  
    ```

3. Trigger log collection.Run the `Invoke-applianceLogCollection` cmdlet.

    ```azurecli
    $fromDate = (Get-Date).AddMinutes(-30)
    $toDate = (Get-Date)
    $operationId = Invoke-ApplianceLogCollection -FromDate $fromDate -ToDate $toDate
    ```

4. Check the status of the log collection job. Use the `Get-ApplianceLogCollectionJobStatus` or `Get-ApplianceLogCollectionHistory` cmdlet.

    ```azurecli
    Get-ApplianceLogCollectionJobStatus -OperationId $OperationId
    ```

    ```azurecli
    Get-ApplianceLogCollectionHistory -FromDate ((Get-Date).AddHours(-3))
    ```

5. Get the stamp ID

    ```azurecli
    $stampId = (Get-ApplianceInstanceConfiguration).StampId
    ```

    Here's an example output:

    ```console
    PS C:\Users\administrator.s46r2004\Documents> $stampId = (Get-ApplianceInstanceConfiguration).StampId  
    VERBOSE: [2025-03-27 19:56:41Z][Invoke-ScriptsWithRetry] Executing 'Retrieving system configuration ...' with timeout 300 seconds ...  
    VERBOSE: [2025-03-27 19:56:41Z][Invoke-ScriptsWithRetry] [CHECK][Attempt 0] for task 'Retrieving system configuration ...' ...  
    VERBOSE: [2025-03-27 19:56:41Z][ScriptBlock] Getting system configuration from https://100.69.172.253:9443/sysconfig/SystemConfiguration  
    VERBOSE: [2025-03-27 19:56:42Z][Invoke-ScriptsWithRetry] Task 'Retrieving system configuration ...' succeeded.  
    PS C:\Users\administrator.s46r2004\Documents> $stampId <Stamp ID>
    ```

## Log collection for a disconnected environment and accessible management endpoint

For this scenario, the management application programming interface (API) is used to copy logs from disconnected operations to a shared folder. Logs are then analyzed locally or manually uploaded to Microsoft via the `Send-DiagnsticsData` cmdlet.

You can trigger the log collection for your disconnected environment using the `Invoke-ApplianceLogCollectionAndSaveToShareFolder`.

1. Run the `Invoke-ApplianceLogCollectionAndSaveToShareFolder` cmdlet.

    ```azurecli
    $fromDate = (Get-Date).AddMinutes(-30)
    $toDate = (Get-Date)
    $operationId = Invoke-ApplianceLogCollectionAndSaveToShareFolder -FromDate $fromDate -ToDate $toDate `
    -LogOutputShareFolderPath "<File Share Path>" -ShareFolderUsername "ShareFolderUser" -ShareFolderPassword (ConvertTo-SecureString "<Share Folder Password>" -AsPlainText -Force)
    ```

2. Copy logs from your disconnected environment to the share folder.

    ```azurecli
    $fromDate = (Get-Date).ToUniversalTime().AddHours(-1)
    $toDate = (Get-Date).ToUniversalTime()
    
    $onDemandRequestBody = @{
        FromDate = $fromDate.ToString("o")
        ToDate = $toDate.ToString("o")
        SaveToPath = "<share folder path>" # \\10.11.206.2\irvmlogs\winfieldlogs
        UserPassword = "****"
        UserName = "<share folder user name>" # 10.11.206.2\administrator
    } | ConvertTo-JSON
    
    $mgmtClientCert = "***" # The client certificate to access management endpoint
    $mgmtIpAddress = "***"
    
    $OperationId = Invoke-RestMethod -Certificate $mgmtClientCert -Method "PUT" -URI "https://$($mgmtIpAddress):9443/logs/logCollectionIndirectJob" -Content "application/json" -Body $onDemandRequestBody -Verbose
    ```

3. Check the status of the log collection job. Use the `Get-ApplianceLogCollectionJobStatus` or `Get-ApplianceLogCollectionHistory` cmdlet to retrieve the current log collection job status.

    ```azurecli
    Get-ApplianceLogCollectionJobStatus -OperationId $OperationId
    ```

    ```azurecli
    Get-ApplianceLogCollectionHistory -FromDate ((Get-Date).AddHours(-3))
    ```

4. Send diagnostic data to Microsoft.
    - First, use the `Send-DiagnosticsData` cmdlet to manually send diagnostics data to Microsoft. For more information, see [Send-DiagnosticsData](disconnected-operations-fallback.md#send-diagnosticdata).

    - Then, unzip all files into the share folder.

    ```azurecli
    $logShareFolderPath = "***"
    $unzipLogPath = "***"
    $files = Get-ChildItem -Path $logShareFolderPath | Where-Object { $_.Extension -like "*.zip*" }
    foreach ($file in $files) {
        Expand-Achive -Path $file.FullName -Destination $unzipLogPath -Verbose
    }
    ```

5. Get the stamp ID

    ```azurecli
    $stampId = (Get-ApplianceInstanceConfiguration).StampId
    ```

## Log collection with an inaccessible management endpoint

Use fallback log collection to collect and send logs when the disconnected operations with Azure Local VM is down, the management endpoint isn't accessible, and standard log collection can't be invoked.

There are three methods in this scenario:

**Copy-DiagnosticData**: Used to copy logs from the disconnected operations with Azure Local VM to a local folder.
**Send-DiagnosticData**: Used to send logs to Microsoft for analysis.
**Get-observabilityStampId**: Used to get the stamp ID.

For more information, see [Use appliance fallback log collection](disconnected-operations-fallback.md).

::: moniker-end

::: moniker range="<=azloc-24111"

This feature is available only in Azure Local 2411.2.

::: moniker-end