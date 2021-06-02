---
title: Monitor Azure Stack HCI clusters from Azure portal
description: How to enable Logs and Monitoring capabilities to monitor Azure Stack HCI clusters from Azure portal.
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 05/26/2021
---

# Monitor Azure Stack HCI clusters from Azure portal

> Applies to: Azure Stack HCI, version 21H2 Preview

This topic explains how to enable Logs and Monitoring capabilities to monitor Azure Stack HCI clusters. You can [access these features in preview](https://aka.ms/hci-insights) from Azure portal.

   > [!IMPORTANT]
   > These features will be available to try in public preview after June 15, 2021.

If you haven't already, be sure to [Register your cluster with Azure](../deploy/register-with-azure.md). After you've enabled Logs and Monitoring, you can use [Azure Stack HCI Insights](azure-stack-hci-insights.md) to monitor cluster health, performance, and usage.

   > [!IMPORTANT]
   > Monitoring an Azure Stack HCI cluster from Azure portal requires every server in the cluster to be Azure Arc-enabled. If you registered your cluster on or after May 25, 2021, this happens by default. Otherwise, you'll need to [enable Azure Arc integration](../deploy/register-with-azure.md#enabling-azure-arc-integration).

## Logs capability (preview)

After you register your cluster and Arc-enable the servers, you'll see the following in Azure portal:

- An Azure Stack HCI resource in the specified resource group
- **Server - Azure Arc** resources for every server in the cluster in the `<clustername>ArcInstanceResourceGroup`
- Nodes with **Server-Azure Arc** resource link on the Azure Stack HCI resource page under the **Nodes** tab

Now that your cluster nodes are Arc-enabled, navigate to your Azure Stack HCI cluster resource page. Under the **Capabilities** tab you will see the option to enable Logs, which should say **Not configured**.

:::image type="content" source="media/monitor-azure-portal/logs-capability.png" alt-text="Select the Logs capability under the Capabilities tab" lightbox="media/monitor-azure-portal/logs-capability.png":::

This capability is an Arc for Servers extension that simplifies installing the Microsoft Monitoring Agent. Because you're using the Arc for Servers extension to enable this workflow, if you ever add additional servers to your cluster, they will automatically have the Microsoft Monitoring Agent installed on them as well.

   > [!NOTE]
   > The Microsoft Monitoring Agent for Windows communicates outbound to the Azure Monitor service over TCP port 443. If the servers connect through a firewall or proxy server to communicate over the internet, review [these requirements](/azure/azure-monitor/agents/log-analytics-agent#network-requirements) to understand the network configuration required.

### Configure the Log Analytics Agent extension

To configure the Log Analytics Agent extension:

1. Under the **Capabilities** tab, select **Logs.**
2. Select **Use existing** to use the existing workspace for your subscription.
3. Click **Add** at the bottom of the page.

   :::image type="content" source="media/monitor-azure-portal/enable-log-analytics.png" alt-text="Enable Log Analytics on Azure portal" lightbox="media/monitor-azure-portal/enable-log-analytics.png":::

4. When the configuration is finished, **Logs** will appear as **Configured** under the **Capabilities** tab.
5. Select **Settings > Extensions** from the toolbar on the left. You should see that each of your servers has successfully installed the Microsoft Monitoring Agent.

You have now successfully installed the log analytics extension.

### Disable Log Analytics

If you’d like to disable the Logs capability, you'll need to remove the Microsoft Monitoring Agent from the Extensions settings. Note that this does not delete the Log Analytics workspace in Azure or any of the data that resides in it, so you'll have to do that manually.

To remove the Microsoft Monitoring Agent from every server in the cluster, follow these steps:

1. Select **Settings > Extensions** from the toolbar on the left.
2. Select the **MicrosoftMonitoringAgent** checkbox.
3. Click **Remove**, and then **Yes**.

## Monitoring capability (preview)

Now that you've set up a Log Analytics workspace, you can enable monitoring. Once monitoring is enabled, the data generated from your on-premises Azure Stack HCI cluster will then be collected in a Log Analytics workspace in Azure. Within that workspace, you can collect data about the health of your cluster. By default, monitoring collects the following logs every hour:

- SDDC Management (Microsoft-Windows-SDDC-Management/Operational; Event ID: 3000, 3001, 3002, 3003, 3004)

To change the frequency of log collection, refer to [Event Log Channel](azure-stack-hci-insights.md#event-log-channel).

### Enable monitoring visualizations

Enabling monitoring will turn on monitoring for all Azure Stack HCI clusters currently associated with the Log Analytics workspace. You will be billed based on the amount of data ingested and the data retention settings of your Log Analytics workspace.

To enable this capability from Azure portal, follow these steps:

1. Under the **Capabilities** tab, select **Monitoring**, then **Enable**.
1. Monitoring should now show as **Configured** under the **Capabilities** tab.

The **Microsoft-windows-sddc-management/operational** Windows event channel will be added to your Log Analytics workspace. By collecting these logs, we'll be able to show the health status of the individual servers, drives, volumes, and VMs.

After you enable monitoring, it can take up to an hour to collect the data. When the process is finished, you'll be able go to the **Monitoring** tab and see a rich visualization of the health of your cluster, as in the screenshot below.

:::image type="content" source="media/monitor-azure-portal/monitoring-visualization.png" alt-text="Enabling monitoring displays a rich visualization of the health of your cluster from Azure portal" lightbox="media/monitor-azure-portal/monitoring-visualization.png":::

You'll see tiles for the health status of your overall cluster along with key subcomponents. The first tile shows any health faults that the Health Service has thrown on your cluster. The other three tiles show you the health status of your drives, VMs, and volumes so that you can easily discern what's going on with the internals of your HCI cluster. You'll also see charts for CPU, memory, and storage usage. These charts are populated using the SDDC Management logs that are collected every hour by default. This view will allow you to check up on your HCI cluster through the Azure portal without having to connect to it directly.

### Disable monitoring visualizations

To disable monitoring, follow these steps:

1. Select **Monitoring** under the **Capabilities** tab.
2. Select **Disable Monitoring**.

When you disable the monitoring feature, the Health Service and SDDC Management logs are no longer collected; however, existing data is not deleted. If you'd like to delete that data, go into your Log Analytics workspace and delete the data manually.

## Azure Monitor pricing

Because pricing can vary due to multiple factors, such as the region of Azure you're using, visit the [Azure Monitor pricing calculator](https://azure.microsoft.com/pricing/details/monitor/) for the most up-to-date pricing calculations.

As described earlier, when you enable monitoring visualization, logs are collected from:

- SDDC Management (Microsoft-Windows-SDDC-Management/Operational; Event ID: 3000, 3001, 3002, 3003, 3004)

Azure Monitor has pay-as-you-go pricing, and the first GB per billing account per month is free.

| **Clusters in standalone Log Analytics workspace** | **GB ingested per month** | **Estimated cost** |
|:---------------------------------------------------|:--------------------------|:-------------------|
| Two-node cluster                                   | ~1 MB per hour            | Free               |
| Four-node cluster                                  | ~1 MB per hour            | Free               |
| Eight-node cluster                                 | ~1 MB per hour            | Free               |

The table below shows a rough pricing estimate for Azure Stack HCI clusters of different sizes.

| **Clusters in same Log Analytics Workspace** | **GB ingested per month** | **Estimated cost**            |
|:---------------------------------------------|:--------------------------|:------------------------------|
| Small deployment (3 two-node clusters)       | ~3 GB                     | Free                          |
| Medium deployment (10 four-node clusters)    | ~10 GB                    | $2.76 per GB after first 5 GB |
| Large deployment (25 four-node clusters)     | ~25 GB                    | $2.76 per GB after first 5 GB |

Every GB of data ingested into your Log Analytics workspace can be retained at no charge for up to 31 days. Data retained beyond the first 31 days will be charged $0.12 per GB per month.

## Next steps

For related information, see also:

- [Azure Stack HCI Insights](azure-stack-hci-insights.md)
- [Monitor Azure Stack HCI clusters from Windows Admin Center](monitor-cluster.md)
