---
title: Support tool for Azure Local
description: This article provides guidance on the Azure Local Support Diagnostic Tool for Azure Local.
author: alkohli
ms.author: alkohli
ms.topic: how-to
ms.date: 11/06/2024
---

# Use the Azure Local Support Diagnostic Tool to troubleshoot issues

This article provides information to download and use the Azure Local Support Diagnostic Tool. Use the tool to troubleshoot and diagnose complex issues on Azure Local. The tool is a set of PowerShell commands to simplify data collection, troubleshooting, and resolution of common issues.

This tool isn't a substitute for expert knowledge. If you encounter any issues, contact Microsoft Support

## Benefits

The Azure Local Support Diagnostic Tool simplifies support and troubleshooting using simple commands to identify issues without expert product knowledge.

The tool provides:

- **Easy installation and updates**: Install and update natively using PowerShell Gallery, without extra requirements.

- **Diagnostic checks**: Provides diagnostic checks based on common issues, incidents, and telemetry data.

- **Automatic data collection**: Automatically collects important data to provide to Microsoft Customer Support.

- **Regular updates**: Updates with new checks and useful commands to manage, troubleshoot, and diagnose issues on Azure Local.

## Prerequisites

Before you use the PowerShell module, make sure to:

- Download the Azure Local Support Diagnostic Tool from the [PowerShell Gallery](https://www.powershellgallery.com/packages?q=hci).

- Import the module into an elevated PowerShell window using an account with administrator privileges on the local system.

- Install the module on each node of the Azure Local system.

## Install and use the Azure Local Support Diagnostic Tool

Run the following commands in PowerShell:

To install the tool, run the following command:

```powershell
Install-Module –Name Microsoft.AzureStack.HCI.CSSTools
```

To list all available diagnostic checks, run the following command:

```powershell
Invoke-AzsSupportDiagnosticCheck –ProductName <BaseSystem, Registration>
```

Run all diagnostic checks by pressing `CTRL+SPACE` after the parameter `ProductName`.

To collect data using one of our pre-defined collection sets, run the following command:

```powershell
New-AzsSupportDataBundle –Component <Component>
```

To check all data collection sets, press `CTRL+SPACE` after the parameter `Component`.

To collect your own dataset, run the following command:

```powershell
$ClusterCommands = @(<clusterCommand1>,<clusterCommand2>)
$nodeCommands = @(<nodeCommand1>,<nodeCommand2>)
$nodeEvents = @(<eventLogName1>,<eventLogName2>)
$nodeRegistry = @(<registryPath1>,<registryPath2>)
$nodeFolders = @(<folderPath1>,<folderPath2>)


New-AzsSupportDataBundle -ClusterCommands $clusterCommands `
-NodeCommands $nodeCommands `
-NodeEvents $nodeEvents `
-NodeRegistry $nodeRegistry `
-NodeFolders $nodeFolders `
-ComputerName @(<computerName1>,<computerName2>)
```

## Example scenario

To troubleshoot Azure Local core products, run the following command:

#### For registration issues

```powershell
Invoke-AzsSupportDiagnosticCheck -ProductName Registration
```

Here's example output for a registration issue:

```output
PS C:\temp> Invoke-AzsSupportDiagnosticCheck -ProductName Registration
Starting known issue check for Azure Stack HCI: Registration.                                                                                                       
Starting Azure Stack HCI base system validation.                                                                                                                        
Gathering information from all clustered nodes.                                                                                                                         
We are preparing to collect diagnostic information from your environment                                                                                                
We started the diagnostic data collection! This might take some time.                                                                                                   
Finished collecting diagnostic information.                                                                                                                             
====[ Validating registration state on node: HCI-N-1 ]====                                                                                                              
[Pass] [Azure Stack HCI - General registration state]                                                                                                                   
Validate that the cluster is registered
Details: Validation successfull

[Fail] [Azure Stack HCI - Azure Connection state]
Validate that the cluster is in a connected state
Details: This Azure Stack HCI node does not seem to be connected to azure. Ensure that this node is in a connected state.
Documentation: https://learn.microsoft.com/en-us/azure-stack/hci/deploy/troubleshoot-hci-registration.

[Pass] [Azure Arc Agent - Connection state]
Validate that the azure arc agent is connected
Details: Validation successfull

[Pass] [Azure Arc Agent - Service state]
Validate that all azure arc services are running
Details: Validation successfull

[Pass] [Azure Arc Agent - Heartbeat state]
Validate that the azure arc agent has sent out a heartbeat at least a day ago
Details: Validation successfull

[Pass] [Azure Stack HCI - Arc Agent onboarded]
Validate that all arc agent checks are passed
Details: Validation successfull

[Fail] [Validation summary]

Details: At least one node reported an invalid registration state.

We will collect log information from your envirorment.
Creating local storage container for diagnostic data.
Gathering cluster data ... this might take a while.
Cluster data collection complete.
We are preparing to collect diagnostic information from your environment
We started the diagnostic data collection! This might take some time.
Waiting for all diagnostic output to be generated and compressed ... this might take a while.
Finished collecting diagnostic information.
Starting copy of items ... this might take a while.
All items copied.
Successfully created archive C:\temp\6c5a4685-6e32-4b68-aeec-05475f8d6c6f\log-collection-RegistrationInformation07-22_06-03-2024.zip. Removing raw data C:\temp\6c5a4685-6e32-4b68-aeec-05475f8d6c6f\container.
Data collection done . Please upload the file to the Microsoft Workspace.
```

#### For base Azure Local system issues

```powershell
Invoke-AzsSupportDiagnosticCheck -ProductName BaseSystem
```

Here's an example of the output for base system issues:

```output
PS C:\temp> Invoke-AzsSupportDiagnosticCheck -ProductName BaseSystem
Starting known issue check for Azure Stack HCI: BaseSystem.
Gathering information from all clustered nodes.
We are preparing to collect diagnostic information from your environment
We started the diagnostic data collection! This might take some time.
Starting to validate cluster settings.
[Pass] [Failover Clustering - Cluster validation report contains no errors]
Validate that there are no critical errors in the cluster validation report
Details: Validation successfull

[Pass] [Failover Clustering - Cluster Networks have redundancy]
Validate that we have redundancy in clustered networks
Details: Validation successfull

[Pass] [Failover Clustering - Validation Summary]
Validate that there are no critical issues in our cluster validation report.
Details: Validation successfull

Collecting node data.
Finished collecting diagnostic information.
====[ Validating data from node: HCI-N-1 ]====
[Pass] [Windows Features - All windows features installed]
Verify that all features required for Azure Local are installed.
Details: Validation successfull

[Pass] [Validation summary]
Ensure that no other check has returned a failed state
Details: Validation successfull
```

Afterwards, a comprehensive overview of the different components that are required for properly connected Azure Local systems is created. Based on this overview, you can either follow troubleshooting guidance or reach out to Microsoft Support for assistance.

To collect data, refer to the following two example scenarios:

#### For automatic data collection

```powershell
New-AzsSupportDataBundle -Component OS
==== CUT ==================== CUT =======
Data collection done C:\temp\Azs.Support\XXXXXXX\SupportDataBundle-XX-XX_XX-XX-XXXX.zip . Please upload the file to the Microsoft Workspace
```

#### For manual data collection

```powershell
$ClusterCommands = @()
$nodeCommands = @('Get-AzureStackHci','Get-AzureStackHCIArcIntegration','Get-ClusteredScheduledTask | fl *','systeminfo.exe')
$nodeEvents = @('system','application','Microsoft-AzureStack-HCI/Admin')
$nodeRegistry = @('HKLM:\Cluster\ArcForServers')
$nodeFolders = @('C:\Windows\Tasks\ArcforServers\','C:\ProgramData\AzureConnectedMachineAgent\Log\')

New-AzsSupportDataBundle -ClusterCommands $clusterCommands `
-NodeCommands $nodeCommands `
-NodeEvents $nodeEvents `
-NodeRegistry $nodeRegistry `
-NodeFolders $nodeFolders `
-ComputerName (Get-ClusterNode)

==== CUT ==================== CUT =======
Data collection done C:\temp\Azs.Support\XXXXXXX\SupportDataBundle-XX-XX_XX-XX-XXXX.zip . Please upload the file to the Microsoft Workspace.
```

## Questions or Feedback?

Do you have an issue? Would like to share feedback with us about the Azure Local Support Diagnostic Tool?
We Listen! To submit feedback, use "contact owners" option inside PSGallery.

## Next steps

For related information, see also:

- [Get support for Azure Local](get-support.md).
