---
title: Join the Azure Stack HCI preview channel
description: How to join the Azure Stack HCI preview channel and upgrade your cluster by using Windows PowerShell or Windows Admin Center.
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 05/25/2021
---

# Join the Azure Stack HCI preview channel

> Applies to: Azure Stack HCI, version 21H2 Preview

The Azure Stack HCI release preview channel is an opt-in program that lets customers install the next version of the operating system before it's officially released. It's intended for customers who want to evaluate new features, system architects who want to build a solution before conducting a broader deployment, or anyone who wants to see what's next for Azure Stack HCI. There are no program requirements or commitments. Preview builds are only available via Windows Update using Windows Admin Center or PowerShell.

   > [!WARNING]
   > Don't use preview builds in production. Preview builds contain experimental pre-release software made available for evaluating and testing only. You might experience crashes, security vulnerabilities, or data loss. Be sure to back up any important virtual machines (VMs) before upgrading your cluster. Once you install a build from the preview channel, the only way to go back is a clean install.

## What's new in the preview channel

Azure Stack HCI, version 21H2 Preview contains the following new features:

- [Multi-cluster monitoring with Azure portal](monitor-azure-portal.md)
- [Use GPUs with clustered VMs](use-gpu-with-clustered-vm.md)
- [Dynamic CPU compatibility mode](processor-compatibility-mode.md)
- [Storage thin provisioning](thin-provisioning.md)
- [Network ATC](../deploy/network-atc.md)

## How to join the preview channel

You can install preview builds on an Azure Stack HCI cluster by using either Windows Admin Center or PowerShell. Make sure that all servers in the cluster are online, and that the cluster is [registered with Azure](../deploy/register-with-azure.md). The upgrade process will take between one and two hours.

   > [!IMPORTANT]
   > Before upgrading to Azure Stack HCI, version 21H2 Preview, be sure to apply the [May 20, 2021 preview update (KB5003237)](https://support.microsoft.com/en-us/topic/may-20-2021-preview-update-kb5003237-0c870dc9-a599-4a69-b0d2-2e635c6c219c) to Azure Stack HCI, version 20H2 via Windows Update. See [Update Azure Stack HCI clusters](update-cluster.md).

### Install the version 21H2 Preview build using Windows Admin Center

To upgrade a cluster using Windows Admin Center:

1. Make sure you have the latest version of Windows Admin Center installed on a management PC or server.

2. Connect to the Azure Stack HCI cluster you wish to upgrade, and select **Settings** from the bottom-left corner of the screen. Select **Join the preview channel**, then **Get started**.

   :::image type="content" source="media/preview-channel/join-preview-channel.png" alt-text="Select join the preview channel, then Get started" lightbox="media/preview-channel/join-preview-channel.png":::

3. You'll be reminded that preview builds are provided as-is, and are not eligible for production-level support. Select the **I understand** checkbox, then click **Join the preview channel**. This may take a few minutes to complete.

4. You should see a confirmation that you've successfully joined the preview channel and that the cluster is now ready to flight preview builds.

   :::image type="content" source="media/preview-channel/joined-preview-channel.png" alt-text="Your cluster is now ready to flight preview builds" lightbox="media/preview-channel/joined-preview-channel.png":::

   > [!NOTE]
   > If any of the servers in the cluster say **Not configured** for preview builds, try repeating the process.

5. Select **Updates** from the **Tools** pane at the left. Feature updates will be displayed.

   :::image type="content" source="media/preview-channel/feature-updates.png" alt-text="Feature updates will be displayed" lightbox="media/preview-channel/feature-updates.png":::

6. Select **Install**. A readiness check will be displayed.

   :::image type="content" source="media/preview-channel/readiness-check.png" alt-text="A readiness check will be displayed" lightbox="media/preview-channel/readiness-check.png":::

7. When the readiness check is complete, you're ready to install the updates. Review the updates listed, and select **Install** to start the update.

   :::image type="content" source="media/preview-channel/install-updates.png" alt-text="Review the updates and install them" lightbox="media/preview-channel/install-updates.png":::

8. You'll be able to see the installation progress as in the screenshot below. Because you're updating the operating system with new features, the updates may take a while to complete. You can continue to use Windows Admin Center for other operations during the update process.

   :::image type="content" source="media/preview-channel/updates-in-progress.png" alt-text="You'll be able to see the installation progress as updates are installed" lightbox="media/preview-channel/updates-in-progress.png":::

### Install the version 21H2 Preview build using PowerShell

To upgrade a cluster using PowerShell:

1.	Run the following cmdlets on every server in the cluster:

    ```PowerShell
    Set-WSManQuickConfig
    Enable-PSRemoting
    Set-NetFirewallRule -Group "@firewallapi.dll,-36751" -Profile Domain -Enabled true
    ```

2.	To test whether the cluster is properly set up to apply software updates using Cluster-Aware Updating (CAU), run the `Test-CauSetup` cmdlet, which will notify you of any warnings or errors:

    ```PowerShell
    Test-CauSetup -ClusterName Cluster1
    ```

3.  Validate the cluster's hardware and settings by running the `Test-Cluster` cmdlet:

    ```PowerShell
    Test-Cluster -ClusterName Cluster1
    ```

4.	Run the following cmdlet with no parameters on every server in the cluster:
    
    ```PowerShell
    Set-PreviewChannel
    ```

    This will configure your server to receive builds sent to the **ReleasePreview-External** audience. If your server was not already configured for flight signing, you'll need to restart after opting in. The module's output will tell you if you need to restart.

5.	Check for the feature update to Azure Stack HCI, version 21H2 Preview:

    ```PowerShell
    Invoke-CauScan -ClusterName <ClusterName> -CauPluginName "Microsoft.RollingUpgradePlugin" -CauPluginArguments @{'WuConnected'='true';} -Verbose | fl *
    ```

    Inspect the output of the above cmdlet and verify that each server is offered the same Feature Update, which should be the case.

6.	You'll need a separate Azure Stack HCI server outside the cluster to run the `Invoke-CauRun` cmdlet from. It can even be a VM on which you temporarily installed Azure Stack HCI. **Important: The system on which you run `Invoke-CauRun` must be running Azure Stack HCI, version 20H2 with the [May 20, 2021 preview update (KB5003237)](https://support.microsoft.com/en-us/topic/may-20-2021-preview-update-kb5003237-0c870dc9-a599-4a69-b0d2-2e635c6c219c) installed**.

    ```PowerShell
    Invoke-CauRun -ClusterName <ClusterName> -CauPluginName "Microsoft.RollingUpgradePlugin" -CauPluginArguments @{'WuConnected'='true';} -Verbose -EnableFirewallRules -Force
    ```

7. Check for any further updates and install them. See [Install updates with PowerShell](update-cluster.md#install-updates-with-powershell).

## Post upgrade steps

Once the upgrade process is complete, you may want to update the cluster functional level, update the storage pool version, or optionally upgrade VM configuration levels.

## Next steps

For more information, see also:

- [Update Azure Stack HCI clusters](update-cluster.md)
