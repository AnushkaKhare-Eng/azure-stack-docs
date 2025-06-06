---
title: Monitor Disconnected Operations for Azure Local (preview)
description: Learn how to monitor Disconnected operations for Azure Local (preview).
author: ronmiab
ms.topic: concept-article
ms.date: 05/22/2025
ms.author: robess
ms.reviewer: robess
ai-usage: ai-assisted
---

# Monitor disconnected operations for Azure Local (preview)

::: moniker range=">=azloc-24112"

[!INCLUDE [applies-to](../includes/hci-applies-to-23h2.md)]

This article explains how to monitor infrastructure and workloads running on Azure Local with disconnected operations by integrating with external monitoring solutions. You can use Microsoft, non-Microsoft, and open-source monitoring systems for specific resource types.

For more information on disconnected operations, see [Disconnected operations for Azure Local (preview)](./disconnected-operations-overview.md).

[!INCLUDE [IMPORTANT](../includes/disconnected-operations-preview.md)]

## Why monitor disconnected operations?

Monitoring is essential for ensuring the reliability, performance, and security of IT systems that support modern business operations. By continuously collecting and analyzing telemetry data — such as CPU usage, disk usage and utilization, memory consumption, network traffic, and error rates organizations gain real-time visibility into the health of their environments, whether on-premises, in the cloud, or across hybrid deployments.

Monitored data can be stored on-premises or in the cloud, depending on the monitoring solution used. This flexibility allows you to choose the best approach for your organizations specific needs and requirements.

## Benefits of monitoring

Monitoring disconnected operations for Azure Local provides several benefits, including:

- Ensure the availability and performance of critical systems.
- Detect and diagnose issues in real-time.
- Analyze performance trends and patterns.
- Optimize resource utilization.
- Ensure compliance with security and regulatory requirements.
- Provide insights for capacity planning and resource allocation.

For information on the views that provide insights, visualization, and analysis of your monitored telemetry data, see [View types in Operations Manager](/system-center/scom/manage-console-view-types?view=sc-om-2025&preserve-view=true).

## What can be monitored?

The following components of Azure Local with disconnected operations can be monitored using external solutions:
- [Azure Local](#monitor-azure-local-infrastructure) (infrastructure) and the [disconnected operations appliance](#monitor-azure-local-infrastructure) (local Azure portal and Arc services).
- [Virtual machines](#monitor-virtual-machines) (VMs).
- [Azure Kubernetes Service (AKS)](#monitor-azure-kubernetes-service-clusters) clusters.

## Prerequisites

Install and deploy System Center Operations Manager (SCOM), see [System Center Operations Manager](/system-center/scom/system-requirements?view=sc-om-2025&preserve-view=true).

## Monitor Azure Local infrastructure

The disconnected operations appliance provides the local Azure portal and Arc services to deploy and manage Azure Local instances. Monitoring the appliance ensures the local portal and services remain available. Use System Center Operations Manager and the disconnected operations management pack for this purpose.

### System Center Operations Manager

When you deploy Azure Local using the disconnected operations feature, you can integrate with external solutions such as System Center Operations Manager to monitor the instance and nodes.

1. Install the Operations Manager agent on each node. Follow the steps in [Install Windows Agent Manually Using MOMAgent.msi](/system-center/scom/manage-deploy-windows-agent-manually?view=sc-om-2025#deploy-the-operations-manager-agent-with-the-agent-setup-wizard&preserve-view=true). Choose the installation method that best fits your environment.

1. Import these management packs:

    - [Windows Server Operating System 2016 and above for Base OS](https://aka.ms/AAvqh49).

    - [Microsoft System Center Management Pack for Windows Server Cluster 2016 and above for Cluster](https://aka.ms/AAvqwlr).

    - [Microsoft System Center 2019 Management Pack for Hyper-V](https://aka.ms/AAvqh4i).

    - [AzS HCI S2D MP for Storage Spaces Direct (S2D)](https://aka.ms/AAvqwo9).

For more information, see [Operations Manager](/system-center/scom/welcome?view=sc-om-2025&preserve-view=true).

### Disconnected operations management pack

The disconnected operations management pack for System Center Operations Manager provides monitoring capabilities for Azure Local instances and nodes deployed with the disconnected operations feature. The management pack is designed to help you monitor the health and performance of your Azure Local infrastructure in a disconnected environment.

Capabilities of the disconnected operations management pack include:

- Single disconnected operations deployment management.

- Support for Active Directory Federation Services (AD FS).

- Health and metrics dashboards.

- Preconfigured alert rules based on metrics for issue detection and operator action highlighting, including certificate expiration 
warnings.

- Notification and reporting support.

To download the System Center Management Pack and user guide, see [Microsoft System Center Management Pack for Azure Local with disconnected operations](https://aka.ms/disconnected-operations-scom-mp).

For monitoring failover clusters, see [Monitoring Failover Cluster with Operations Manager](/system-center/scom/manage-monitor-clusters-overview).

## Monitor virtual machines

Monitor virtual machines (VMs) on Azure Local with disconnected operations using System Center Operations Manager or third-party/open-source solutions. Install the appropriate agents in each VM.

For more information, see [Operations Manager](/system-center/scom/welcome?view=sc-om-2025&preserve-view=true).

## Monitor Azure Kubernetes Service clusters

Monitor Azure Kubernetes Service (AKS) clusters and container applications on Azure Local with disconnected operations using third-party or open-source solutions. The following solutions are commonly used for monitoring AKS clusters:

- **Prometheus**: An open-source monitoring and alerting toolkit designed for reliability and scalability. It collects metrics from configured targets at specified intervals, evaluates rule expressions, and can trigger alerts if certain conditions are met.

    For more information on Prometheus, see [Overview Prometheus](https://prometheus.io/docs/introduction/overview/).

- **Grafana**: An open-source analytics and monitoring platform that integrates with Prometheus and other data sources to visualize metrics and logs. It provides a rich set of features for creating dashboards, alerts, and reports.

    For more information on Grafana, see [Grafana OSS](https://grafana.com/oss/grafana/).

Download these solutions from their repositories and install them on an AKS cluster running on Azure Local, or deploy them on a Kubernetes cluster running outside Azure Local.

::: moniker-end

::: moniker range="<=azloc-24111"

This feature is available only in Azure Local 2411.2.

::: moniker-end
