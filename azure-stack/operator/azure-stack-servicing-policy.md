---
title: Azure Stack Hub servicing policy
titleSuffix: Azure Stack Hub
description: Learn about the Azure Stack Hub servicing policy and how to keep an integrated system in a supported state.
author: sethmanheim

ms.topic: article
ms.date: 07/12/2021
ms.author: sethm
ms.reviewer: niy
ms.lastreviewed: 03/18/2020

# Intent: As an Azure Stack operator, I want to learn about servicing policy and how to keep an integrated system supported.
# Keyword: servicing policy azure stack

---


# Azure Stack Hub servicing policy

This article describes the servicing policy for Azure Stack Hub integrated systems and what you must do to keep your system in a supported state.

## Download update packages for integrated systems

Microsoft releases both full update packages and hotfix packages to address specific issues.

Full update packages are hosted in a secure Azure endpoint. You can download them manually using the [Azure Stack Hub Updates downloader tool](https://aka.ms/azurestackupdatedownload). If your scale unit is connected, the update appears automatically in the administrator portal as **Update available**. For more information about each release, you can click any release from the [Update package release cadence](#update-package-release-cadence) section of this article.

Hotfix update packages are hosted in the same secure Azure endpoint. You can download them using the embedded links in each of the respective hotfix KB articles; for example, [Azure Stack Hub Hotfix 1.1809.12.114](https://support.microsoft.com/help/4481548/azure-stack-hotfix-1-1809-12-114). Similar to the full, monthly update packages, Azure Stack Hub operators can download the .xml and .zip files and import them using the procedure in [Apply updates in Azure Stack Hub](azure-stack-apply-updates.md). Azure Stack Hub operators with connected scale units will see the hotfixes automatically appear in the administrator portal with the message **Update available**.

If your scale unit isn't connected and you want to be notified about each hotfix release, subscribe to the [RSS feed](https://azurestackhubdocs.azurewebsites.net/xml/hotfixes.rss) to be notified about each hotfix release.

## Update package types

There are two types of update packages for integrated systems:

- **Microsoft software updates**. Microsoft is responsible for the end-to-end servicing lifecycle for the Microsoft software update packages. These packages can include the latest Windows Server security updates, non-security updates, and Azure Stack Hub feature updates. You can download theses update packages directly from Microsoft.

- **OEM hardware vendor-provided updates**. Azure Stack Hub hardware partners are responsible for the end-to-end servicing lifecycle (including guidance) for the hardware-related firmware and driver update packages. In addition, Azure Stack Hub hardware partners own and maintain guidance for all software and hardware on the hardware lifecycle host. The OEM hardware vendor hosts these update packages on their own download site.

## Update package release cadence

Microsoft expects to release software update packages multiple times throughout the year.

OEM hardware vendors release their updates on an as-needed basis. Check with your OEM for the latest updates to hardware.

Find documentation on how to plan for and manage updates, and how to determine your current version in [Manage updates overview](azure-stack-updates.md).

For information about a specific update, including how to download it, see the release notes for that update:

- [Azure Stack Hub 2102 update](./release-notes.md?preserve-view=true&view=azs-2102)
- [Azure Stack Hub 2008 update](./release-notes.md?preserve-view=true&view=azs-2008)
- [Azure Stack Hub 2005 update](./release-notes.md?preserve-view=true&view=azs-2005)

## Hotfixes

Occasionally, Microsoft provides hotfixes for Azure Stack Hub that address a specific issue that's often preventative or time-sensitive. Each hotfix is released with a corresponding Microsoft Knowledge Base (KB) article that details the issues addressed in that hotfix.

Hotfixes are downloaded and installed just like the regular full update packages for Azure Stack Hub. However, unlike a full update, hotfixes can install in minutes. We recommend Azure Stack Hub operators set maintenance windows when installing hotfixes. Hotfixes update the version of your Azure Stack Hub cloud so you can easily determine if the hotfix has been applied. A separate hotfix is provided for each version of Azure Stack Hub that's still in support. **Each hotfix for a specific iteration is cumulative and includes the previous hotfixes for that same version.** You can read more about the applicability of a specific hotfix in the corresponding KB article. See the release notes links in the previous section.

Before you update to the new major version, apply the latest hotfix in the **current** major version.

Starting with build 2005, when you update to a **new** major version (for example, 1.2005.x to 1.2008.x), the latest hotfixes (if any are available at the time of package download) in the new major version are installed automatically. Your 2008 installation is then current with all hotfixes. From that point forward, if a hotfix is released for 2008, you should install it.

For information about currently available hotfixes, [see the release notes](release-notes.md) "Hotfixes" section for that update.

## Keep your system under support

For your Azure Stack Hub instance to remain in a supported state, the instance must run the most recently released update version or run either of the two preceding update versions.

You must also have an active support agreement with the hardware partner that manufactured the system. Microsoft is not able to support you without a hardware support agreement in place.

Hotfixes aren't considered major update versions. If your Azure Stack Hub instance is behind by more than two updates, it's considered out of compliance. You must update to at least the minimum supported version to receive support.

For example, if the most recently available update version is 1904, and the previous two update packages were versions 1903 and 1902, both 1902 and 1903 remain in support. However, 1901 is out of support. The policy holds true when there's no release for a month or two. For example, if the current release is 1807 and there was no 1806 release, the previous two update packages of 1805 and 1804 remain in support.

Microsoft software update packages are non-cumulative and require the previous update package or hotfix as a prerequisite. If you decide to defer one or more updates, consider the overall runtime if you want to get to the latest version.

### Resource provider version support

For Azure Stack Hub resource providers, it's important to note that only the most recently released version of a given resource provider that is compatible with your supported version of Azure Stack Hub is supported, even though you may be using an older version of Azure Stack Hub that is still within the support window.

For more information about resource provider compatibility, see the release notes for that specific resource provider.

## Get support

Azure Stack Hub follows the same support process as Azure. Enterprise customers can follow the process described in [How to create an Azure support request](/azure/azure-supportability/how-to-create-azure-support-request). If you're a customer of a Cloud Solution Provider (CSP), contact your CSP for support. For more information, see the [Azure Support FAQs](https://azure.microsoft.com/support/faq/).

For help with troubleshooting update issues, see [Best practices for troubleshooting Azure Stack Hub patch and update issues](azure-stack-troubleshooting.md).

## Next steps

- [Manage updates in Azure Stack Hub](azure-stack-updates.md)
- [Best practices for troubleshooting Azure Stack Hub patch and update issues](azure-stack-troubleshooting.md)