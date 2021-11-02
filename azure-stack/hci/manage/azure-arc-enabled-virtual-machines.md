---
title: VM provisioning through Azure portal on Azure Stack HCI 
description: How to set up Azure Arc enabled virtual machines to provide cloud-based provisioning and management for on-premises VMs running on Azure Stack HCI
author: khdownie
ms.author: v-kedow
ms.topic: how-to
ms.service: azure-stack
ms.subservice: azure-stack-hci
ms.date: 11/02/2021
---

# VM provisioning through Azure portal on Azure Stack HCI (preview)

> Applies to: Azure Stack HCI, version 21H2

Azure Stack HCI, version 21H2 customers can use Azure portal to provision and manage on-premises Windows and Linux virtual machines (VMs) running on Azure Stack HCI clusters. With [Azure Arc](https://azure.microsoft.com/services/azure-arc/), IT administrators can delegate permissions and roles to app owners and DevOps teams to enable self-service VM management for their Azure Stack HCI clusters through the Azure cloud control plane. Using [Azure Resource Manager](/azure/azure-resource-manager/management/overview) templates, VM provisioning can be easily automated in a secure cloud environment.

## Benefits of Azure Arc-enabled Azure Stack HCI

By using Azure portal to provision and manage on-premises and cloud VMs, users get a consistent experience. Users can access their VMs only, not the host fabric, enabling role-based access control and self-service.

## Next steps

To sign up for this capability in preview, fill out [this form](https://aka.ms/joinEAP).
