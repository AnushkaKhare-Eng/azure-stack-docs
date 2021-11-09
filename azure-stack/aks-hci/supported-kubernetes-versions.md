---
title: Supported Kubernetes versions in Azure Kubernetes Service on Azure Stack HCI
description: Understand the Kubernetes version support policy and lifecycle of clusters in Azure Kubernetes Service (AKS on Azure Stack HCI)
services: container-service
ms.topic: article
ms.date: 10/8/2021
author: mkostersitz
ms.author: mikek
---

# Supported Kubernetes versions in Azure Kubernetes Service (AKS on Azure Stack HCI)

The Kubernetes community releases minor versions roughly every three months. Recently, the Kubernetes community has [increased the support window for each version from 9 months to 12 months](https://kubernetes.io/blog/2020/08/31/kubernetes-1-19-feature-one-year-support/), starting with version 1.19. 

Minor version releases include new features and improvements. Patch releases are more frequent (sometimes weekly) and are intended for critical bug fixes within a minor version. Patch releases include fixes for security vulnerabilities or major bugs.

## Kubernetes versions

Kubernetes uses the standard [Semantic Versioning](https://semver.org/) versioning scheme for each version:

```
[major].[minor].[patch]

Example:
  1.17.7
  1.17.8
```

Each number in the version indicates general compatibility with the previous version:

* **Major versions** change when incompatible API updates or backwards compatibility may be broken.
* **Minor versions** change when functionality updates are made that are backwards compatible to the other minor releases.
* **Patch versions** change when backwards-compatible bug fixes are made.

Aim to run the latest patch release of the minor version you're running. For example, your production cluster is on **`1.17.7`**. **`1.17.8`** is the latest available patch version available for the *1.17* series. You should upgrade to **`1.17.8`** as soon as possible to ensure your cluster is fully patched and supported.

## Kubernetes version support policy

AKS on Azure Stack HCI defines a generally available version as a version available for download when deploying or updating AKS on Azure Stack HCI. AKS on Azure Stack HCI supports three GA minor versions of Kubernetes:

* The latest GA minor version that is released in AKS on Azure Stack HCI (which we'll refer to as N).
* Two previous minor versions.
    * Each supported minor version also supports a maximum of two (2) stable patches.

AKS on Azure Stack HCI may also support preview versions, which are explicitly labeled as such.

> [!NOTE]
> AKS on Azure Stack HCI uses safe deployment practices which involve gradual region deployment. This means it may take up to 10 business days for a new release or a new version to be available in all regions.

The supported window of Kubernetes versions on AKS on Azure Stack HCI is known as "N-2": (N (Latest release) - 2 (minor versions)).

For example, if AKS on Azure Stack HCI introduces *1.17.a* today, support is provided for the following versions:

New minor version    |    Supported Version List
-----------------    |    ----------------------
1.17.a               |    1.17.a, 1.17.b, 1.16.c, 1.16.d, 1.15.e, 1.15.f

Where ".letter" is representative of patch versions.

When a new minor version is introduced, the oldest minor version and patch releases supported are deprecated and removed. For example, the current supported version list is:

```
1.17.a
1.17.b
1.16.c
1.16.d
1.15.e
1.15.f
```

AKS on Azure Stack HCI releases 1.18.\*, removing all the 1.15.\* versions out of support in 30 days.

> [!NOTE]
> If customers are running an unsupported Kubernetes version, they will be asked to upgrade when requesting support for the cluster. Clusters running unsupported Kubernetes releases are not covered by the [AKS on Azure Stack HCI support policies](./support-policies.md).

In addition to the above, AKS on Azure Stack HCI supports a maximum of two **patch** releases of a given minor version. So given the following supported versions:

```
Current Supported Version List
------------------------------
1.17.8, 1.17.7, 1.16.10, 1.16.9
```

If AKS on Azure Stack HCI releases `1.17.9` and `1.16.11`, the oldest patch versions are deprecated and removed, and the supported version list becomes:

```
New Supported Version List
----------------------
1.17.*9*, 1.17.*8*, 1.16.*11*, 1.16.*10*
```

### Supported `kubectl` versions

You can use one minor version older or newer of `kubectl` relative to your *kube-apiserver* version, consistent with the [Kubernetes support policy for kubectl](https://kubernetes.io/docs/setup/release/version-skew-policy/#kubectl).

For example, if your *kube-apiserver* is at *1.17*, then you can use versions *1.16* to *1.18* of `kubectl` with that *kube-apiserver*.

To install or update your version of `kubectl`, run `az AKS on Azure Stack HCI install-cli`.

## Release and deprecation process

You can reference upcoming version releases and deprecations on the [AKS on Azure Stack HCI Kubernetes release calendar](#aks-on-azure-stack-hci-kubernetes-release-calendar).

For new **minor** versions of Kubernetes:

* AKS on Azure Stack HCI publishes a pre-announcement with the planned date of a new version release and respective old version deprecation in the [AKS on Azure Stack HCI release notes](https://aka.ms/aks-hci-relnotes) at least 30 days prior to removal.  
* Users have **30 days** from version removal to upgrade to a supported minor version release to continue receiving support.

For new **patch** versions of Kubernetes:

* Because of the urgent nature of patch versions, patch versions can be introduced into the service as they become available.
* In general, AKS on Azure Stack HCI does not broadly communicate the release of new patch versions. However, AKS on Azure Stack HCI constantly monitors and validates available CVE patches to support them in AKS on Azure Stack HCI in a timely manner. If a critical patch is found or user action is required, AKS on Azure Stack HCI will notify users to upgrade to the newly available patch.
* Users have **30 days** from a patch release's removal from AKS on Azure Stack HCI to upgrade into a supported patch and continue receiving support.

### Supported versions policy exceptions

AKS on Azure Stack HCI reserves the right to add or remove new/existing versions with one or more critical production-impacting bugs or security issues without advance notice.

Specific patch releases may be skipped or rollout accelerated, depending on the severity of the bug or security issue.

## Default versions

When you deploy an AKS on Azure Stack HCI cluster using Windows Admin Center or PowerShell, the cluster defaults to the newest available version of Kubernetes. For example, if AKS on Azure Stack HCI supports *1.17.a*, *1.17.b*, *1.16.c*, *1.16.d*, *1.15.e*, and *1.15.f*, the default version selected is *1.17.b*.

To find out what versions are currently available for your subscription and region, use the
[Get-AksHciKubernetesVersion](./reference/ps/get-akshcikubernetesversion.md) cmdlet. The following example lists the available Kubernetes versions:

```powershell
Get-AksHciKubernetesVersion
```

## AKS on Azure Stack HCI Kubernetes release calendar

For the past release history, see [Kubernetes](https://en.wikipedia.org/wiki/Kubernetes#History).

|  K8s version | Upstream release  | AKS on Azure Stack HCI preview  | AKS on Azure Stack HCI GA  | End of life |
|--------------|-------------------|--------------|---------|-------------|
| 1.19  | Aug-04-20  | Sep 2020   | Nov 2020  | 1.22 GA |
| 1.20  | Dec-08-20  | Jan 2021   | Mar 2021  | 1.23 GA |
| 1.21  | Apr-08-21 | May 2021   | Jul 2021  | 1.24 GA |
| 1.22  | Aug-04-21 | Oct 2021   | Nov 2021  | 1.25 GA |
| 1.23  | Dec 2021 | Jan 2022   | Feb 2022  | 1.26 GA |

## FAQ

**How does Microsoft notify me of new Kubernetes versions?**

The AKS on Azure Stack HCI team publishes pre-announcements with planned dates of the new Kubernetes versions in our documentation, our [GitHub](https://github.com/Azure/aks-hci/releases).

**How often should I expect to upgrade Kubernetes versions to stay in support?**

Starting with Kubernetes 1.19, the [open source community has expanded support to 1 year](https://kubernetes.io/blog/2020/08/31/kubernetes-1-19-feature-one-year-support/). AKS on Azure Stack HCI commits to enabling patches and support matching the upstream commitments. For AKS on Azure Stack HCI clusters on 1.19 and greater, you will be able to upgrade at a minimum of once a year to stay on a supported version. 

For versions on 1.18 or below, the window of support remains at 9 months, requiring an upgrade once every 9 months to stay on a supported version. Regularly test new versions and be prepared to upgrade to newer versions to capture the latest stable enhancements within Kubernetes.

**What happens when a user upgrades a Kubernetes cluster with a minor version that isn't supported?**

If you're on the *n-3* version or older, it means you're outside of support and will be asked to upgrade. When your upgrade from version n-3 to n-2 succeeds, you're back within our support policies. For example:

- If the oldest supported AKS on Azure Stack HCI version is *1.15.a* and you are on *1.14.b* or older, you're outside of support.
- When you successfully upgrade from *1.14.b* to *1.15.a* or higher, you're back within our support policies.

Downgrades are not supported.

**What does 'Outside of Support' mean?**

'Outside of Support' means that:
* The version you're running is outside of the supported versions list.
* You'll be asked to upgrade the cluster to a supported version when requesting support, unless you're within the 30-day grace period after version deprecation. 

Additionally, AKS on Azure Stack HCI doesn't make any runtime or other guarantees for clusters outside of the supported versions list.

**What happens when a user scales a Kubernetes cluster with a minor version that isn't supported?**

For minor versions not supported by AKS on Azure Stack HCI, scaling in or out should continue to work. Since there are no Quality of Service guarantees, we recommend upgrading to bring your cluster back into support.

**Can I skip multiple AKS on Azure Stack HCI versions during cluster upgrade?**

When you upgrade a supported AKS on Azure Stack HCI cluster, Kubernetes minor versions cannot be skipped. For example, upgrades between:
  * *1.12.x* -> *1.13.x*: allowed.
  * *1.13.x* -> *1.14.x*: allowed.
  * *1.12.x* -> *1.14.x*: not allowed.

To upgrade from *1.12.x* -> *1.14.x*:
1. Upgrade from *1.12.x* -> *1.13.x*.
2. Upgrade from *1.13.x* -> *1.14.x*.

Skipping multiple versions can only be done when upgrading from an unsupported version back into a supported version. For example, you can upgrade from an unsupported *1.10.x* to a supported *1.15.x*.

**Can I create a new 1.xx.x cluster during its 30 day support window?**

No. Once a version is deprecated/removed, you cannot create a cluster with that version. As the change rolls out, you will start to see the old version removed from your version list. This process may take up to two weeks from announcement, progressively by region.

**I am on a freshly deprecated version, can I still add new node pools? Or will I have to upgrade?**

No. You will not be allowed to add node pools of the deprecated version to your cluster. You can add node pools of a new version. However, this may require you to update the control plane first. 

## Next steps

For information on how to upgrade your cluster, see [Update the Kubernetes version of AKS clusters on Azure Stack HCI](./upgrade.md).
