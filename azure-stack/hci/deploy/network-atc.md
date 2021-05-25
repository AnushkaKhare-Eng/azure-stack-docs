---
title: Simplify host networking with Network ATC
description: This topic covers how to simplify host networking using Network ATC for Azure Stack HCI.
author: v-dasis
ms.topic: how-to
ms.date: 05/25/2021
ms.author: v-dasis
ms.reviewer: JasonGerend
---

# Simplify host networking with Network ATC

> Applies to: Azure Stack HCI, version 21H2 Preview

This article guides you through the key functions of using Network ATC, which simplifies the deployment and network configuration management for Azure Stack HCI nodes. Network ATC provides an intent-based approach to host network deployment. By specifying one or more intents (management, compute, or storage) for a network adapter, Network ATC can automate the deployment of the intended configuration.

If you have feedback or encounter any issues, review the Requirements and best practices section, check the Network ATC event log, and work with your Microsoft support team.

## Overview

Deployment and operation of Azure Stack HCI networking can be a complex and error-prone process. Due to the configuration flexibility provided with the host networking stack, there are many moving parts that can be easily misconfigured or overlooked. Staying up to date with the latest best practices is also a challenge as improvements are continuously made to the underlying technologies. Additionally, configuration consistency across HCI cluster nodes is important as it leads to a more reliable experience.

Network ATC can help:

- Reduce host networking deployment time and complexity
- Reduce host networking configuration errors
- Deploy the latest Microsoft validated and supported best practices
- Eliminate configuration drift

## Definitions

Network ATC introduces some new terminology:

**Intent**: An intent is a definition of how you intend to use the physical adapters in your system. An intent has a handle (friendly name), identifies one or more physical adapters, and includes one or more intent types.

An individual physical adapter can only be included in one intent. By default, an adapter does not have an intent (there is no special status or property given to adapters that don’t have an intent). You can have multiple intents; the number of intents you have will be limited by the number of adapters in your system.

**Intent type**: Every intent requires one or more intent types. The currently supported intent types are:

- Management - adapters are used for management access to nodes
- Compute - adapters are used to connect virtual machine (VM) traffic to the physical network
- Storage - adapters are used for SMB traffic including Storage Spaces Direct

Any combination of the intent types can be specified for any specific single intent. However, certain intent types can only be specified in one intent:

- Management: Can be defined in a maximum of one intent
- Compute: Unlimited
- Storage: Can be defined in a maximum of one intent

**Intent mode**: An intent can be specified at a standalone level or at a cluster level. Modes are system-wide; you can't have a network intent that is standalone and another that is clustered on the same host system. Clustered mode is the focus on this preview.

- *Standalone mode*: Intents are expressed and managed independently for each host. This mode allows you to test an intent before implementing it across a cluster. Once a host is clustered, any standalone intents are ignored. Standalone intents can only be copied to a cluster from a node that is not a member of that cluster or from one cluster to another cluster.

- *Cluster mode*: Intents are applied to all cluster nodes. This is the recommended deployment mode and is required when a server is a member of a failover cluster.

**Override**: By default, Network ATC deploys the most common configuration, asking for the smallest amount of user input. Overrides allow you to deviate from the Network ATC defaults. For example, you may choose to modify the VLANs used for storage adapters.

You can only modify some properties when you initially create an intent. You can't modify the feature itself after you've deployed it. For example, a virtual switch does not allow modification of SR-IOV after it has been deployed. Network ATC does not modify this existing behavior.

## Requirements and best practices

The following are requirements and best practices for using the Network ATC feature in Azure Stack HCI:

- Supported on Azure Stack HCI, version 21H2 Preview release or later.

- All servers in the cluster must be running Azure Stack HCI, version 21H2 Preview.

- Must use two or more physical host systems that are Azure Stack HCI certified.

- Adapters managed by Network ATC must be symmetric (of the same make, model, speed, and configuration) and available on each cluster node. For more information on adapter symmetry, see [Switch Embedded Teaming (SET)](../concepts/host-network-requirements.md#switch-embedded-teaming-set)

- Each symmetric physical adapter must use the same name on each cluster node.

- Ensure each network adapter has an "Up" status, as verified by the PowerShell `Get-NetAdapter` cmdlet.

- Remove any prior ATC intent on the adapter.

- Must install the following Windows features on each node:

  - Network ATC
  - Data Center Bridging (DCB)
  - Failover Clustering
  - Hyper-V

- Best practice: Insert each adapter in the same PCI slot(s) in each host. This leads to ease in automated naming conventions by imaging systems.

- Best practice: Configure the physical network (switches) prior to Network ATC including VLANs, MTU, and DCB configuration. See [Physical Network Requirements](../concepts/physical-network-requirements.md) for more information.

You can use the following cmdlet to install the required Windows features:

```powershell
Install-WindowsFeature -Name NetworkATC, 'Data-Center-Bridging', 'Failover-Clustering', 'Hyper-V' -IncludeManagementTools
```

> [!NOTE]
> Network ATC does not require a system reboot if the other Windows features have already been installed.

## Using Windows PowerShell

There are several new PowerShell commands included with the Network ATC service. Run the`Get-Command -ModuleName NetworkATC` cmdlet to identify them. Be sure and always run PowerShell as an administrator.

Typically, only a few of the Network ATC cmdlets are needed. Here is a brief overview of the cmdlets before you start:

|PowerShell command|Description|
|--|--|
|Add-NetIntent|Creates and submits an intent to Network ATC|
|Set-NetIntent|Modifies an existing intent|
|Get-NetIntent|Gets a list of intent requests from the target|
|Get-NetIntentStatus|Gets the status of intent requests from the target|
|New-NetIntentOverrides|Specifies overrides to the default configuration|
|Remove-NetIntent|Removes an intent from the local node or cluster. This does not destroy the configuration invoked by Network ATC.
|Set-NetIntentRetryState|After three attempts, Network ATC will stop attempting to implement the intent (`Get-NetIntentStatus` = 'Failed'). This command instructs Network ATC to try again after you've resolved the issue.|

## Example network intents

Network ATC modifies how you deploy host networking, not what you deploy. Network ATC can implement multiple scenarios so long as each scenario is supported by Microsoft. Here are some examples of common deployment options, and the PowerShell commands needed. These are not the only combinations available but they should give you an idea of the possibilities.

For simplicity we only demonstrate two physical adapters per SET team, however it is possible to add more. Refer to [Plan Host Networking](../concepts/host-network-requirements.md) for more information.

### Fully converged intent

For this intent, Network ATC deploys and manages the compute, storage, and management networks across all cluster nodes.

:::image type="content" source="media/network-atc/network-atc-2-full-converge.png" alt-text="Fully converged network intent"  lightbox="media/network-atc/network-atc-2-full-converge.png":::

```powershell
Add-NetIntent -Name ConvergedIntent -Management -Compute -Storage -ClusterName HCI01 -AdapterName pNIC01, pNIC02
```

### Converged compute and storage intent; separate management intent

Network ATC manages two intents across cluster nodes. Management uses pNIC01, and pNIC02; Compute and storage are on different adapters.

:::image type="content" source="media/network-atc/network-atc-3-separate-management-compute-storage.png" alt-text="Storage and compute converged network intent"  lightbox="media/network-atc/network-atc-3-separate-management-compute-storage.png":::

```powershell
Add-NetIntent -Name Mgmt -Management -ClusterName HCI01 -AdapterName pNIC01, pNIC02
Add-NetIntent -Name Compute_Storage -Compute -Storage -ClusterName HCI01 -AdapterName pNIC03, pNIC04
```

### Fully disaggregated intent

Fir this intent, Network ATC manages compute, storage, and management networks on different adapters across all cluster nodes.

:::image type="content" source="media/network-atc/network-atc-4-fully-disaggregated.png" alt-text="Fully disaggregated network intent"  lightbox="media/network-atc/network-atc-4-fully-disaggregated.png":::

```powershell
Add-NetIntent -Name Mgmt -Management -ClusterName HCI01 -AdapterName pNIC01, pNIC02
Add-NetIntent -Name Compute -Compute -ClusterName HCI01 -AdapterName pNIC03, pNIC04
Add-NetIntent -Name Storage -Storage -ClusterName HCI01 -AdapterName pNIC05, pNIC06
```

### Storage-only intent

For this intent, Network ATC manages storage only. Management and compute adapters will not be managed by Network ATC.

:::image type="content" source="media/network-atc/network-atc-5-fully-disaggregated-storage-only.png" alt-text="Storage only network intent"  lightbox="media/network-atc/network-atc-5-fully-disaggregated-storage-only.png":::

```powershell
Add-NetIntent -Name Storage -Storage -ClusterName HCI01 -AdapterName pNIC05, pNIC06
```

### Compute and management intent

For this intent, Network ATC manages compute and management networks, but not storage.

:::image type="content" source="media/network-atc/network-atc-6-disaggregated-management-compute.png" alt-text="Management and compute network intent"  lightbox="media/network-atc/network-atc-6-disaggregated-management-compute.png":::

```powershell
Add-NetIntent -Name Management_Compute -Management -Compute -ClusterName HCI01 -AdapterName pNIC01, pNIC02
```

### Multiple compute (switch) intent

For this intent, Network ATC manages multiple compute switches.

:::image type="content" source="media/network-atc/network-atc-7-multiple-compute.png" alt-text="Multiple switches network intent"  lightbox="media/network-atc/network-atc-7-multiple-compute.png":::

```powershell
Add-NetIntent -Name Compute1 -Compute -ClusterName HCI01 -AdapterName pNIC03, pNIC04
Add-NetIntent -Name Compute2 -Compute -ClusterName HCI01 -AdapterName pNIC05, pNIC06
```

## Activity overview

The following activities represent common usage of Network ATC.

You can specify any combination of the following types of intent:

- Compute – adapters will be used to connect virtual machines traffic to the physical network
- Storage – adapters will be used for SMB traffic including Storage Spaces Direct
- Management – adapters will be used for management access to nodes. This intent is not covered in this article, but feel free to explore.

This article covers the following activities:

- Activity 1: Configure a cluster

- Activity 2: Configure an override

- Activity 3: Validate automatic remediation

- Activity 4: Remove an intent


## Activity 1: Configure a cluster

In this activity, we use Network ATC to maintain a consistent configuration across all cluster nodes. This is beneficial for several reasons including improved reliability of the cluster. With Network ATC, the cluster is considered the configuration boundary. That is, all nodes in the cluster share the same configuration (symmetric intent).

> [!IMPORTANT]
> If a node is clustered, you must use a clustered intent. Standalone intents are ignored.

### Task 1: Create a cluster

Create a failover cluster of one or more nodes. The cluster can include any number of supported Azure Stack HCI nodes. We also demonstrate adding other nodes to the cluster later.

> [!NOTE]
> You can add all nodes at one time using the `New-Cluster` cmdlet, then add the intent to all nodes. Alternatively, you can incrementally add nodes to the cluster. Network ATC will manage the new nodes automatically.

1. Create the cluster on the first node. A simple example is shown as follows:

    ```powershell
    New-Cluster -Name HCI01
    ```

1. Use the following example cmdlets to verify that the cluster was created and the nodes in the cluster.

    ```powershell
    Get-Cluster
    Get-ClusterNode
    ```

### Task 2: Create a cluster intent

In this task, we will create a Network ATC intent that specifies the compute and storage intent types with no overrides.

1. On one of the cluster nodes, run `Get-NetAdapter` to review the physical adapters. Ensure that each node in the cluster has the same named physical adapters.

    ```powershell
    Get-NetAdapter -Name pNIC01, pNIC02 -CimSession (Get-ClusterNode).Name | Select Name, PSComputerName
    ```

1. Run the following command to add the storage and compute intent types to pNIC01 and pNIC02. Note that we specify the `-ClusterName` parameter.

    ```powershell
    Add-NetIntent -Name Cluster_ComputeStorage -Compute -Storage -ClusterName HCI01 -AdapterName pNIC01, pNIC02
    ```

    The command should immediately return after some initial verification. The cmdlet checks that each node in the cluster has:

    - the adapters specified
    - adapters report status 'Up'
    - adapters ready to be teamed to create the specified vSwitch

1. Run the `Get-NetIntent` cmdlet to see the cluster intent. If you have more than one intent, you can specify the `Name` parameter to see details of only a specific intent.

    ```powershell
    Get-NetIntent -ClusterName HCI01
    ```

1. To see the provisioning status of the intent, run the `Get-NetIntentStatus` command:

    ```powershell
    Get-NetIntentStatus -ClusterName HCI01 -Name Cluster_ComputeStorage
    ```

    Note the status parameter that shows Provisioning, Validating, Success, Failure.

1. Status should display success in a few minutes. If this doesn't occur or you see a Status parameter failure, check the event viewer for issues.

    ```powershell
    Get-NetIntentStatus -ClusterName HCI01 -Name Cluster_ComputeStorage
    ```
 
1. Check that the configuration has been applied to all cluster nodes. For this example, check that the VMSwitch was deployed on each node in the cluster and that host virtual NICs were created for storage. For more validation examples, see the [Network ATC demo](https://youtu.be/Z8UO6EGnh0k).

    ```powershell
    Get-VMSwitch -CimSession (Get-ClusterNode).Name | Select Name, ComputerName
    ```

> [!NOTE]
> At this time, Network ATC does not configure IP addresses for any of its managed adapters. Once `Get-NetIntentStatus` reports status completed, you should add IP addresses to the adapters.

### Task 3: Add a new node to the cluster

You can freely add nodes to the cluster. Network ATC ensures that each node in the cluster receives the same intent, improving the reliability of the cluster (the new node must meet the requirements mentioned earlier in this article).

In this task, you will add additional nodes to the cluster and observe how Network ATC enforces a consistent configuration across all nodes in the cluster.

1. Use the `Add-ClusterNode` cmdlet to add the additional (unconfigured) nodes to the cluster. You only need management access to the cluster at this time. Each node in the cluster should have all pNICs named the same.

    ```powershell
    Add-ClusterNode -Cluster HCI01
    Get-ClusterNode
    ```

1. Check the status across all cluster nodes using the `-ClusterName` parameter.

    ```powershell
    Get-NetIntentStatus -ClusterName HCI01
    ```

    > [!NOTE]
    > If pNICs do not exist on one of the additional nodes, `Get-NetIntentStatus` will report the error 'PhysicalAdapterNotFound', which easily identifies the provisioning issue.

1. Check the provisioning status of all nodes using `Get-NetIntentStatus`. The cmdlet reports the configuration for both nodes. Note that this may take a similar amount of time to provision as the original node.

    ```powershell
    Get-NetIntentStatus -ClusterName HCI01
    ```

1. You can experiment by adding several nodes to the cluster at once.


## Activity 2: Configure an override

In this activity, we will modify the default configuration and verify Network ATC makes the necessary changes.

> [!IMPORTANT]
> Network ATC implements the Microsoft tested, **Best Practice** configuration. We highly recommend that you only modify the default configuration with guidance from Microsoft Azure Stack HCI support teams.

### Task 1: Update an intent with a single override

This task will help you override the default configuration which has already been deployed. This example modifies the default bandwidth reservation for SMB Direct.

> [!IMPORTANT]
> The `Set-NetIntent` cmdlet is used to update an already deployed intent. Use the `Add-NetIntent` cmdlet to add an override at initial deployment time

1. Get a list of possible override cmdlets. We use wildcards to see the options available:

    ```powershell
    Get-Command -Noun NetIntent*Over* -Module NetworkATC
    ```

1. Create an override object for the DCB Quality of Service (QoS) configuration:

    ```powershell
    $QosOverride = New-NetIntentQosPolicyOverrides
    $QosOverride
    ```

1. Modify the bandwidth percentage for SMB Direct:

    ```powershell
    $QosOverride.BandwidthPercentage_SMB = 25
    $QosOverride
    ```

    > [!NOTE]
    >It is expected that no values appear for any property you don’t override.

1. Submit the intent request specifying the override:

    ```powershell
    Set-NetIntent -Name Cluster_ComputeStorage -QosPolicyOverrides $QosOverride
    ```

1. Wait for the provisioning status to complete:

    ```powershell
    Get-NetIntentStatus -Name Cluster_ComputeStorage | Format-Table IntentName, Host, ProvisioningStatus, ConfigurationStatus
    ```

1. Check that the override has been properly set on all cluster nodes. In the example, the SMB_Direct traffic class was overridden with a bandwidth percentage of 25%:

    ```powershell
    Get-NetQosTrafficClass -Cimsession (Get-ClusterNode).Name | Select PSComputerName, Name, Priority, Bandwidth
    ```

### Task 2: Update an intent with multiple overrides

This task will help you override the default configuration which has already been deployed. This example modifies the default bandwidth reservation for SMB Direct and the maximum transmission unit (MTU) of the adapters.

1. Create an override object. In this example, we create two objects - one for QoS properties and one for a physical adapter property.

    ```powershell
    $QosOverride = New-IntentQosPolicyOverrides
    $AdapterOverride = New-NetIntentAdapterPropertyOverrides
    $QosOverride
    $AdapterOverride
    ```

1. Modify the SMB bandwidth percentage:

    ```powershell
    $QosOverride.BandwidthPercentage_SMB = 60
    $QosOverride
    ```

1. Modify the MTU size (JumboPacket) value:

    ```powershell
    $AdapterOverride.JumboPacket = 9014
    ```

1. Use the `Set-NetIntent` command to update the intent and specify the overrides objects previously created.

    Use the appropriate parameter based on the type of override you're specifying. In the example below, we use the `AdapterPropertyOverrides` parameter for the `$AdapterOverride` object that was created with `New-NetIntentAdapterPropertyOverrides` cmdlet whereas the `QosPolicyOverrides` parameter is used with the `$QosOverride` object created from `New-NetIntenQosPolicyOverrides` cmdlet.

    ```powershell
    Set-NetIntent -ClusterName HCI01 -Name Cluster_ComputeStorage -AdapterPropertyOverrides $AdapterOverride -QosPolicyOverride $QosOverride
    ```

1. First, notice that the status for all nodes in the cluster has changed to ProvisioningUpdate and Progress is on 1 of 2. The progress property is similar to a configuration watermark in that you have a new submission to the Network ATC service that must be enacted.

    ```powershell
    Get-NetIntentStatus -ClusterName HCI01
    ```

1. Wait for the provisioning status to complete:

    ```powershell
    Get-NetIntentStatus -ClusterName HCI01
    ```

1. Check that traffic class was overridden with a bandwidth percentage of 60%.

    ```powershell
    Get-NetQosTrafficClass -Cimsession (Get-ClusterNode).Name | Select PSComputerName, Name, Priority, Bandwidth 
    ```

1. Check that the adapters MTU (JumboPacket) value was modified and that the host virtual NICs created for storage also have been modified.

    ```powershell
    Get-NetAdapterAdvancedProperty -Name pNIC01, pNIC02, vSMB* -RegistryKeyword *JumboPacket -Cimsession (Get-ClusterNode).Name
    ```

    > [!IMPORTANT]
    > Ensure you modify the cmdlet above to include the adapter names in the intent specified

## Activity 3: Validate automatic remediation

Network ATC will ensure that the deployed configuration stays the same across all cluster nodes. In this activity, we will modify one the configuration (without an override) emulating an accidental configuration change and observe how Network ATC improves the reliability of the system by remediating the misconfigured property. 

>[!NOTE]
> ATC will automatically remediate all of the configuration it manages.

1. Check the adapter's existing MTU (JumboPacket) value:

    ```powershell
    Get-NetAdapterAdvancedProperty -Name pNIC01, pNIC02, vSMB* -RegistryKeyword *JumboPacket -Cimsession (Get-ClusterNode).Name
    ```

1. Modify one of the physical adapter's MTU without specifying an override. This emulates an accidental change or "configuration drift" which must be remediated.

    ```powershell
    Set-NetAdapterAdvancedProperty -Name pNIC01 -RegistryKeyword *JumboPacket -RegistryKeyword *JumboPacket -RegistryValue 4088
    ```

1. Verify that the adapter's existing MTU (JumboPacket) value has been modified:

    ```powershell
    Get-NetAdapterAdvancedProperty -Name pNIC01, pNIC02, vSMB* -RegistryKeyword *JumboPacket -Cimsession (Get-ClusterNode).Name
    ```

1. Now tell Network ATC to retry the configuration. This step is only performed to expedite the remediation. Network ATC will automatically remediate this configuration.

    ```powershell
    Set-NetIntentRetryState -ClusterName HCI01 -Name Cluster_ComputeStorage
    ```

1. Verify that Network ATC has completed its consistency check:

    ```powershell
    Get-NetIntentStatus -ClusterName HCI01 -Name Cluster_ComputeStorage
    ```

1. Verify that the adapter's MTU (JumboPacket) value has returned to the expected value:

    ```powershell
    Get-NetAdapterAdvancedProperty -Name pNIC01, pNIC02, vSMB* -RegistryKeyword *JumboPacket -Cimsession (Get-ClusterNode).Name
    ```

## Activity 4: Remove an intent

If you want to test various configurations on the same adapters, you may need to remove an intent. If you previously deployed a configuration on your system, you may need to reset the node so that Network ATC can deploy its configuration. To do this, copy and paste the following commands to remove all existing intents and their corresponding vSwitch:

```powershell
    $intents = Get-NetIntent
    foreach ($intent in $intents)
    {
        Remove-NetIntent -Name $intent.IntentName
        Remove-VMSwitch -Name "*$($intent.IntentName)*" -ErrorAction SilentlyContinue -Force
    }
    
    Get-NetQosTrafficClass | Remove-NetQosTrafficClass
    Get-NetQosPolicy | Remove-NetQosPolicy -Confirm:$false
    Get-NetQosFlowControl | Disable-NetQosFlowControl
```

## Post-deployment tasks

There are several tasks to complete following a Network ATC deployment. Following an ATC deployment you should:

### Add non-APIPA addresses to storage adapters

This can be accomplished using DHCP on the storage VLANs or by using the `NetIPAddress` cmdlets.

### Set SMB bandwidth limits

If live migration uses SMB Direct (RDMA), configure a bandwidth limit to ensure that live migration does not consume all the bandwidth used by Storage Spaces Direct and Failover Clustering.

### Stretched cluster configuration

Stretched cluster requires additional configuration beyond what Network ATC performs. This must be manually added following success of an intent. For stretched clusters, all nodes in the cluster must use the same intent.

## Network ATC default values

As mentioned, Network ATC deploys several default values. This section lists some of the key default values.

### Default VLANs

Network ATC uses the following default VLANs. These VLANs must be available on the physical network for proper operation.

|Adapter Intent|Default Value|
|--|--|
|Management|Network ATC does not modify the configured VLAN for management adapters|
|Storage Adapter 1|711|
|Storage Adapter 2|712|
|Storage Adapter 3|713|
|Storage Adapter 4|714|
|Storage Adapter 5|715|
|Storage Adapter 6|716|
|Storage Adapter 7|717|
|Storage Adapter 8|718|
|Future Use|719|

Consider the following Network ATC command:

```powershell
Add-NetIntent -Name Cluster_ComputeStorage -Storage -ClusterName HCI01 -AdapterName pNIC01, pNIC02, pNIC03, pNIC04
```

Network ATC will configure the physical NIC (or virtual NIC if required) to use VLANs 711, 712, 713, and 714 respectively.

### Default Data Center Bridging (DCB) configuration

Network ATC establishes the following priorities and bandwidth reservations. This configuration should also be configured on the physical network.

|Policy|Use|Default Priority|Default Bandwidth Reservation|
|--|--|--|--|
|Cluster|Cluster Heartbeat reservation|7|2% if the adapter(s) are <= 10 Gbps
1% if the adapter(s) are > 10 Gbps|
|SMB_Direct|RDMA Storage Traffic|3|50%|
|Default|All other traffic types|0|Remainder|

## Next steps

Learn more about [Stretched clusters](../concepts/stretched-clusters.md).
