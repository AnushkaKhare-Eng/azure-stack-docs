---
title: Create volumes in Azure Stack HCI
description: How to create volumes in Azure Stack HCI using Windows Admin Center and PowerShell.
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.date: 02/04/2021
---

# Create volumes in Azure Stack HCI

> Applies to: Azure Stack HCI, version 20H2

This topic describes how to create volumes on an Azure Stack HCI cluster by using Windows Admin Center and Windows PowerShell, how to work with files on the volumes, and how to enable data deduplication and compression on volumes. To learn how to create volumes and set up replication for stretched clusters, see [Create stretched volumes](create-stretched-volumes.md).

## Create a three-way mirror volume

To create a three-way mirror volume using Windows Admin Center:

1. In Windows Admin Center, connect to a cluster, and then select **Volumes** from the **Tools** pane.
2. On the **Volumes** page, select the **Inventory** tab, and then select **Create volume**.
3. In the **Create volume** pane, enter a name for the volume, and leave **Resiliency** as **Three-way mirror**.
4. In **Size on HDD**, specify the size of the volume. For example, 5 TB (terabytes).
5. Select **Create**.

Depending on the size, creating the volume can take a few minutes. Notifications in the upper-right will let you know when the volume is created. The new volume appears in the Inventory list.

Watch a quick video on how to create a three-way mirror volume.

> [!VIDEO https://www.youtube-nocookie.com/embed/o66etKq70N8]

## Create a mirror-accelerated parity volume

Mirror-accelerated parity (MAP) reduces the footprint of the volume on the HDD. For example, a three-way mirror volume would mean that for every 10 terabytes of size, you will need 30 terabytes as footprint. To reduce the overhead in footprint, create a volume with mirror-accelerated parity. This reduces the footprint from 30 terabytes to just 22 terabytes, even with only 4 servers, by mirroring the most active 20 percent of data, and using parity, which is more space efficient, to store the rest. You can adjust this ratio of parity and mirror to make the performance versus capacity tradeoff that's right for your workload. For example, 90 percent parity and 10 percent mirror yields less performance but streamlines the footprint even further.

  >[!NOTE]
  >Mirror-accelerated parity volumes require Resilient File System (ReFS).

To create a volume with mirror-accelerated parity in Windows Admin Center:

1. In Windows Admin Center, connect to a cluster, and then select **Volumes** from the **Tools** pane.
2. On the Volumes page, select the **Inventory** tab, and then select **Create volume**.
3. In the **Create volume** pane, enter a name for the volume.
4. In **Resiliency**, select **Mirror-accelerated parity**.
5. In **Parity percentage**, select the percentage of parity.
6. Select **Create**.

Watch a quick video on how to create a mirror-accelerated parity volume.

> [!VIDEO https://www.youtube-nocookie.com/embed/R72QHudqWpE]

## Open volume and add files

To open a volume and add files to the volume in Windows Admin Center:

1. In Windows Admin Center, connect to a cluster, and then select **Volumes** from the **Tools** pane.
2. On the **Volumes** page, select the **Inventory** tab.
2. In the list of volumes, select the name of the volume that you want to open.

    On the volume details page, you can see the path to the volume.

4. At the top of the page, select **Open**. This launches the **Files** tool in Windows Admin Center.
5. Navigate to the path of the volume. Here you can browse the files in the volume.
6. Select **Upload**, and then select a file to upload.
7. Use the browser **Back** button to go back to the **Tools** pane in Windows Admin Center.

Watch a quick video on how to open a volume and add files.

> [!VIDEO https://www.youtube-nocookie.com/embed/j59z7ulohs4]

## Turn on deduplication and compression

Deduplication and compression is managed per volume. Deduplication and compression uses a post-processing model, which means that you won't see savings until it runs. When it does, it'll work over all files, even those that were there from before.

To learn more, see [Enable volume encryption, deduplication, and compression](volume-encryption-deduplication.md)

## Create volumes using Windows PowerShell

First, launch Windows PowerShell from the Windows start menu. We recommend using the **New-Volume** cmdlet to create volumes for Azure Stack HCI. It provides the fastest and most straightforward experience. This single cmdlet automatically creates the virtual disk, partitions and formats it, creates the volume with matching name, and adds it to cluster shared volumes – all in one easy step.

The **New-Volume** cmdlet has four parameters you'll always need to provide:

- **FriendlyName:** Any string you want, for example *"Volume1"*
- **FileSystem:** Either **CSVFS_ReFS** (recommended for all volumes; required for mirror-accelerated parity volumes) or **CSVFS_NTFS**
- **StoragePoolFriendlyName:** The name of your storage pool, for example *"S2D on ClusterName"*
- **Size:** The size of the volume, for example *"10TB"*

   > [!NOTE]
   > Windows, including PowerShell, counts using binary (base-2) numbers, whereas drives are often labeled using decimal (base-10) numbers. This explains why a "one terabyte" drive, defined as 1,000,000,000,000 bytes, appears in Windows as about "909 GB". This is expected. When creating volumes using **New-Volume**, you should specify the **Size** parameter in binary (base-2) numbers. For example, specifying "909GB" or "0.909495TB" will create a volume of approximately 1,000,000,000,000 bytes.

### Example: With 2 or 3 servers

To make things easier, if your deployment has only two servers, Storage Spaces Direct will automatically use two-way mirroring for resiliency. If your deployment has only three servers, it will automatically use three-way mirroring.

```PowerShell
New-Volume -FriendlyName "Volume1" -FileSystem CSVFS_ReFS -StoragePoolFriendlyName S2D* -Size 1TB
```

### Example: With 4+ servers

If you have four or more servers, you can use the optional **ResiliencySettingName** parameter to choose your resiliency type.

-	**ResiliencySettingName:** Either **Mirror** or **Parity**.

In the following example, *"Volume2"* uses three-way mirroring and *"Volume3"* uses dual parity (often called "erasure coding").

```PowerShell
New-Volume -FriendlyName "Volume2" -FileSystem CSVFS_ReFS -StoragePoolFriendlyName S2D* -Size 1TB -ResiliencySettingName Mirror
New-Volume -FriendlyName "Volume3" -FileSystem CSVFS_ReFS -StoragePoolFriendlyName S2D* -Size 1TB -ResiliencySettingName Parity
```

## Using storage tiers

In deployments with three types of drives, one volume can span the SSD and HDD tiers to reside partially on each. Likewise, in deployments with four or more servers, one volume can mix mirroring and dual parity to reside partially on each.

To help you create such volumes, Azure Stack HCI provides default tier templates called **MirrorOn*MediaType*** and **NestedMirrorOn*MediaType*** (for performance), and **ParityOn*MediaType*** and **NestedParityOn*MediaType*** (for capacity), where *MediaType* is HDD or SSD. The templates represent storage tiers based on media types and encapsulate definitions for three-way mirroring on the faster capacity drives (if applicable), and dual parity on the slower capacity drives (if applicable).

You can see them by running the **Get-StorageTier** cmdlet on any server in the cluster.

```PowerShell
Get-StorageTier | Select FriendlyName, ResiliencySettingName, PhysicalDiskRedundancy
```

For example, if you have a two-node cluster with only HDD, your output might look something like this:

```PowerShell
FriendlyName      ResiliencySettingName PhysicalDiskRedundancy
------------      --------------------- ----------------------
NestedParityOnHDD Parity                                     1
Capacity          Mirror                                     1
NestedMirrorOnHDD Mirror                                     3
MirrorOnHDD       Mirror                                     1
```

To create tiered volumes, reference these tier templates using the **StorageTierFriendlyNames** and **StorageTierSizes** parameters of the **New-Volume** cmdlet. For example, the following cmdlet creates one volume which mixes three-way mirroring and dual parity in 30:70 proportions.

```PowerShell
New-Volume -FriendlyName "Volume1" -FileSystem CSVFS_ReFS -StoragePoolFriendlyName S2D* -StorageTierFriendlyNames MirrorOnHDD, Capacity -StorageTierSizes 300GB, 700GB
```

Repeat as needed to create more than one volume.

### Nested resiliency volumes

Nested resiliency only applies to two-server clusters; you can't use nested resiliency if your cluster has three or more servers. Nested resiliency enables a two-server cluster to withstand multiple hardware failures at the same time without loss of storage availability, allowing users, apps, and virtual machines to continue to run without disruption. To learn more, see [Plan volumes: choosing the resiliency type](../concepts/plan-volumes.md#choosing-the-resiliency-type).

#### Create nested storage tiers

To create a NestedMirror tier:

```PowerShell
New-StorageTier -StoragePoolFriendlyName S2D* -FriendlyName NestedMirror -ResiliencySettingName Mirror -NumberOfDataCopies 4 -MediaType HDD -CimSession 2nodecluster
```

To create a NestedParity tier:

```PowerShell
New-StorageTier -StoragePoolFriendlyName S2D* -FriendlyName NestedParity -ResiliencySettingName Parity -NumberOfDataCopies 2 -PhysicalDiskRedundancy 1 -NumberOfGroups 1 -FaultDomainAwareness StorageScaleUnit -ColumnIsolation PhysicalDisk -MediaType HDD -CimSession 2nodecluster
```

#### Create nested volumes

To create a NestedMirror volume:

```PowerShell
New-Volume -StoragePoolFriendlyName S2D* -FriendlyName MyMirrorNestedVolume -StorageTierFriendlyNames NestedMirror -StorageTierSizes 500GB -CimSession 2nodecluster
```

To create a NestedParity volume:

```PowerShell
New-Volume -StoragePoolFriendlyName S2D* -FriendlyName MyParityNestedVolume -StorageTierFriendlyNames NestedMirror,NestedParity -StorageTierSizes 200GB, 1TB -CimSession 2nodecluster
```

### Storage tier summary table

The following tables summarize the storage tiers that are/can be created in Azure Stack HCI.

**NumberOfNodes: 2**

| FriendlyName      | MediaType | ResiliencySettingName | NumberOfDataCopies | PhysicalDiskRedundancy | NumberOfGroups | FaultDomainAwareness | ColumnIsolation | Note         |
| ----------------- | :-------: | :-------------------: | :----------------: | :--------------------: |:--------------:| :------------------: | :-------------: | :----------: |
| MirrorOnHDD       | HDD       | Mirror                | 2                  | 1                      | 1              | StorageScaleUnit     | PhysicalDisk    | auto created |
| MirrorOnSSD       | SSD       | Mirror                | 2                  | 1                      | 1              | StorageScaleUnit     | PhysicalDisk    | auto created |
| MirrorOnSCM       | SCM       | Mirror                | 2                  | 1                      | 1              | StorageScaleUnit     | PhysicalDisk    | auto created |
| NestedMirrorOnHDD | HDD       | Mirror                | 4                  | 3                      | 1              | StorageScaleUnit     | PhysicalDisk    | manual       |
| NestedMirrorOnSSD | SSD       | Mirror                | 4                  | 3                      | 1              | StorageScaleUnit     | PhysicalDisk    | manual       |
| NestedMirrorOnSCM | SCM       | Mirror                | 4                  | 3                      | 1              | StorageScaleUnit     | PhysicalDisk    | manual       |
| NestedParityOnHDD | HDD       | Parity                | 2                  | 1                      | 1              | StorageScaleUnit     | PhysicalDisk    | manual       |
| NestedParityOnSSD | SSD       | Parity                | 2                  | 1                      | 1              | StorageScaleUnit     | PhysicalDisk    | manual       |
| NestedParityOnSCM | SCM       | Parity                | 2                  | 1                      | 1              | StorageScaleUnit     | PhysicalDisk    | manual       |

**NumberOfNodes: 3**

| FriendlyName      | MediaType | ResiliencySettingName | NumberOfDataCopies | PhysicalDiskRedundancy | NumberOfGroups | FaultDomainAwareness | ColumnIsolation | Note         |
| ----------------- | :-------: | :-------------------: | :----------------: | :--------------------: |:--------------:| :------------------: | :-------------: | :----------: |
| MirrorOnHDD       | HDD       | Mirror                | 3                  | 2                      | 1              | StorageScaleUnit     | PhysicalDisk    | auto created |
| MirrorOnSSD       | SSD       | Mirror                | 3                  | 2                      | 1              | StorageScaleUnit     | PhysicalDisk    | auto created |
| MirrorOnSCM       | SCM       | Mirror                | 3                  | 2                      | 1              | StorageScaleUnit     | PhysicalDisk    | auto created |

**NumberOfNodes: 4+**

| FriendlyName      | MediaType | ResiliencySettingName | NumberOfDataCopies | PhysicalDiskRedundancy | NumberOfGroups | FaultDomainAwareness | ColumnIsolation | Note         |
| ----------------- | :-------: | :-------------------: | :----------------: | :--------------------: |:--------------:| :------------------: | :-------------: | :----------: |
| MirrorOnHDD       | HDD       | Mirror                | 3                  | 2                      | 1              | StorageScaleUnit     | PhysicalDisk    | auto created |
| MirrorOnSSD       | SSD       | Mirror                | 3                  | 2                      | 1              | StorageScaleUnit     | PhysicalDisk    | auto created |
| MirrorOnSCM       | SCM       | Mirror                | 3                  | 2                      | 1              | StorageScaleUnit     | PhysicalDisk    | auto created |
| ParityOnHDD       | HDD       | Parity                | 1                  | 2                      | Auto           | StorageScaleUnit     | StorageScaleUnit| auto created |
| ParityOnSSD       | SSD       | Parity                | 1                  | 2                      | Auto           | StorageScaleUnit     | StorageScaleUnit| auto created |
| ParityOnSCM       | SCM       | Parity                | 1                  | 2                      | Auto           | StorageScaleUnit     | StorageScaleUnit| auto created |

## Next steps

For related topics and other storage management tasks, see also:

- [Storage Spaces Direct overview](/windows-server/storage/storage-spaces/storage-spaces-direct-overview)
- [Plan volumes](../concepts/plan-volumes.md)
- [Extend volumes](extend-volumes.md)
- [Delete volumes](delete-volumes.md)