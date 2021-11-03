---
title: Azure Stack Hub known issues 
description: Learn about known issues in Azure Stack Hub releases.
author: sethmanheim

ms.topic: article
ms.date: 11/02/2021
ms.author: sethm
ms.reviewer: unknown
ms.lastreviewed: 09/09/2020

# Intent: As an Azure Stack Hub user, I want to know about known issues in the latest release so that I can plan my update and be aware of any issues.
# Keyword: Notdone: keyword noun phrase

---


# Azure Stack Hub known issues

This article lists known issues in Azure Stack Hub releases. The list is updated as new issues are identified.

To access known issues for a different version, use the version selector dropdown above the table of contents on the left.

::: moniker range=">=azs-2005"
> [!IMPORTANT]  
> Review this section before applying the update.
::: moniker-end
::: moniker range="<azs-2005"
> [!IMPORTANT]  
> If your Azure Stack Hub instance is behind by more than two updates, it's considered out of compliance. You must [update to at least the minimum supported version to receive support](azure-stack-servicing-policy.md#keep-your-system-under-support). 
::: moniker-end

<!---------------------------------------------------------->
<!------------------- SUPPORTED VERSIONS ------------------->
<!---------------------------------------------------------->

::: moniker range="azs-2102"
## Update

For known Azure Stack Hub update issues, see [Troubleshooting Updates in Azure Stack Hub](azure-stack-troubleshooting.md#troubleshoot-azure-stack-hub-updates).

### Update to 2102 fails during pre-update checks for AKS/ACR

- Applicable: This issue applies to Azure Kubernetes Service (AKS) and Azure Container Registry (ACR) private preview customers who plan to upgrade to 2102 or apply any hotfixes.
- Remediation: Uninstall AKS and ACR prior to updating to 2102, or prior to applying any hotfixes after updating to 2102. Restart the update after uninstalling these services.
- Occurrence: Any stamp that has ACR or AKS installed will experience this failure.

## Portal

### Administrative subscriptions

- Applicable: This issue applies to all supported releases.
- Cause: The two administrative subscription types **Metering** and **Consumption** have been disabled and should not be used. If you have resources in them, an alert is generated until those resources are removed.
- Remediation: If you have resources running on these two subscriptions, recreate them in user subscriptions.
- Occurrence: Common

## Networking

### Virtual network gateway

#### Documentation links are Azure-specific

- Applicable: This issue applies to all supported releases.
- Cause: The documentation links in the overview page of Virtual Network gateway link to Azure-specific documentation instead of Azure Stack Hub. Use the following links for the Azure Stack Hub documentation:

  - [Gateway SKUs](../user/azure-stack-vpn-gateway-about-vpn-gateways.md#gateway-skus)
  - [Highly Available Connections](../user/azure-stack-vpn-gateway-about-vpn-gateways.md#gateway-availability)
  - [Configure BGP on Azure Stack Hub](../user/azure-stack-vpn-gateway-settings.md#gateway-requirements)
  - [ExpressRoute circuits](azure-stack-connect-expressroute.md)
  - [Specify custom IPsec/IKE policies](../user/azure-stack-vpn-gateway-settings.md#ipsecike-parameters)

### Load balancer

#### IPv6 button visible on "Add frontend IP address"

- Applicable: This issue applies to release 2008 and later.
- Cause: IPv6 button is visible on the **Add frontend IP address** option on a load balancer. These buttons are disabled and cannot be selected.
- Occurrence: Common

#### Backend and frontend ports when floating IP is enabled

- Applicable: This issue applies to all supported releases.
- Cause: Both the frontend port and backend port need to be the same in the load balancing rule when floating IP is enabled. This behavior is by design.
- Occurrence: Common

<!-- ## Compute -->

## Health and alerts

[!INCLUDE [resource providers fail in test-azurestack](../includes/known-issue-alerts-aks-acs.md)]

### No alerts in Syslog pipeline

- Applicable: This issue applies to release 2102.
- Cause: The alert module for customers depending on Syslog for alerts has been disabled in this release. For this release, the health and monitoring pipeline was modified to reduce the number of dependencies and services requirements. As a result, the new services will not emit alerts to the Syslog pipeline.
- Remediation: None.
- Occurrence: Common

<!-- ## Storage -->

<!-- ## SQL and MySQL-->

<!-- ## App Service -->

## Usage

### Wrong status on infrastructure backup

- Applicable: This issue applies to release 2102.
- Cause: The infrastructure backup job can display the wrong status (failed or successful) while the status itself is refreshed. This does not impact the consistency of the backup data, but can cause confusion if an actual failure occurred.
- Remediation: The issue will be fixed in the next hotfix for 2102.

<!-- ### Identity -->

<!-- ### Marketplace -->
::: moniker-end

::: moniker range="azs-2008"
## Update

For known Azure Stack Hub update issues, see [Troubleshooting Updates in Azure Stack Hub](azure-stack-troubleshooting.md#troubleshoot-azure-stack-hub-updates).

### Update failed to install package Microsoft.AzureStack.Compute.Installer to CA VM

- Applicable: This issue applies to all supported releases.
- Cause: During update, a process takes a lock on the new content that needs to be copied to CA VM. When the update fails, the lock is released.
- Remediation: Resume the update.
- Occurrence: Rare

## Portal

### Administrative subscriptions

- Applicable: This issue applies to all supported releases.
- Cause: The two administrative subscriptions that were introduced with version 1804 should not be used. The subscription types are **Metering** subscription, and **Consumption** subscription.
- Remediation: If you have resources running on these two subscriptions, recreate them in user subscriptions.
- Occurrence: Common

## Networking

### Network Security Groups

#### VM deployment fails due to DenyAllOutbound rule

- Applicable: This issue applies to all supported releases.
- Cause: An explicit **DenyAllOutbound** rule to the internet cannot be created in an NSG during VM creation as this will prevent the communication required for the VM deployment to complete. It will also deny the two essential IPs that are required in order to deploy VMs: DHCP IP:169.254.169.254 and DNS IP: 168.63.129.16.

- Remediation: Allow outbound traffic to the internet during VM creation, and modify the NSG to block the required traffic after VM creation is complete.
- Occurrence: Common

### Virtual Network Gateway

#### Documentation links are Azure-specific

- Applicable: This issue applies to all supported releases.
- Cause: The documentation links in the overview page of Virtual Network gateway link to Azure-specific documentation instead of Azure Stack Hub. Use the following links for the Azure Stack Hub documentation:

  - [Gateway SKUs](../user/azure-stack-vpn-gateway-about-vpn-gateways.md#gateway-skus)
  - [Highly Available Connections](../user/azure-stack-vpn-gateway-about-vpn-gateways.md#gateway-availability)
  - [Configure BGP on Azure Stack Hub](../user/azure-stack-vpn-gateway-settings.md#gateway-requirements)
  - [ExpressRoute circuits](azure-stack-connect-expressroute.md)
  - [Specify custom IPsec/IKE policies](../user/azure-stack-vpn-gateway-settings.md#ipsecike-parameters)

### Load Balancer

#### Load Balancer directing traffic to one backend VM in specific scenarios

- Applicable: This issue applies to all supported releases. 
- Cause: When enabling **Session Affinity** on a load balancer, the 2 tuple hash utilizes the PA IP (Physical Address IP) instead of the private IPs assigned to the VMs. In scenarios where traffic directed to the load balancer arrives through a VPN, or if all the client VMs (source IPs) reside on the same node and Session Affinity is enabled, all traffic is directed to one backend VM.
- Occurrence: Common

#### IPv6 button visible on "Add frontend IP address" 

- Applicable: This issue applies to the 2008 release.
- Cause: The IPv6 button is visible and enabled when creating the frontend IP configuration of a public load balancer. This is a cosmetic issue on the portal. IPv6 is not supported on Azure Stack Hub.
- Occurrence: Common

#### Backend port and frontend port need to be the same when floating IP is enabled

- Applicable: This issue applies to all releases. 
- Cause: Both the frontend port and backend port need to be the same in the load balancing rule when floating IP is enabled. This behavior is by design.
- Occurrence: Common

## Compute

### Stop or start VM

#### Stop-Deallocate VM results in MTU configuration removal

- Applicable: This issue applies to all supported releases.
- Cause: Performing **Stop-Deallocate** on a VM results in MTU configuration on the VM to be removed. This behavior is inconsistent with Azure.
- Occurrence: Common



<!-- ## Storage -->
<!-- ## SQL and MySQL-->
<!-- ## App Service -->
<!-- ## Usage -->
<!-- ### Identity -->
<!-- ### Marketplace -->
::: moniker-end

::: moniker range="azs-2005"
## Update

For known Azure Stack Hub update issues, see [Troubleshooting Updates in Azure Stack Hub](azure-stack-troubleshooting.md#troubleshoot-azure-stack-hub-updates).

### Update failed to install package Microsoft.AzureStack.Compute.Installer to CA VM

- Applicable: This issue applies to all supported releases.
- Cause: During update, a process takes a lock on the new content that needs to be copied to CA VM. When the update fails, the lock is released.
- Remediation: Resume the update.
- Occurrence: Rare

### Create failures during patch and update on 4-node Azure Stack Hub environments

- Applicable: This issue applies to all supported releases.
- Cause: Creating VMs in an availability set of 3 fault domains and creating a virtual machine scale set instance fails with a **FabricVmPlacementErrorUnsupportedFaultDomainSize** error during the update process on a 4-node Azure Stack Hub environment.
- Remediation: You can create single VMs in an availability set with 2 fault domains successfully. However, scale set instance creation is still not available during the update process on a 4-node Azure Stack Hub deployment.


## Portal

### Administrative subscriptions

- Applicable: This issue applies to all supported releases.
- Cause: The two administrative subscriptions that were introduced with version 1804 should not be used. The subscription types are **Metering** subscription, and **Consumption** subscription.
- Remediation: If you have resources running on these two subscriptions, recreate them in user subscriptions.
- Occurrence: Common

## Networking

### Network Security Groups

#### Cannot delete an NSG if NICs not attached to running VM

- Applicable: This issue applies to all supported releases.
- Cause: When disassociating an NSG and a NIC that is not attached to a running VM, the update (PUT) operation for that object fails at the network controller layer. The NSG will be updated at the network resource provider layer, but not on the network controller, so the NSG moves to a failed state.
- Remediation: Attach the NICs associated to the NSG that needs to be removed with running VMs, and disassociate the NSG or remove all the NICs that were associated with the NSG.
- Occurrence: Common

#### VM deployment fails due to DenyAllOutbound rule

- Applicable: This issue applies to all supported releases.
- Cause: An explicit **DenyAllOutbound** rule to the internet cannot be created in an NSG during VM creation as this will prevent the communication required for the VM deployment to complete.
- Remediation: Allow outbound traffic to the internet during VM creation, and modify the NSG to block the required traffic after VM creation is complete.
- Occurrence: Common

### Virtual Network Gateway

#### Documentation links are Azure-specific

- Applicable: This issue applies to all supported releases.
- Cause: The documentation links in the overview page of Virtual Network gateway link to Azure-specific documentation instead of Azure Stack Hub. Use the following links for the Azure Stack Hub documentation:

  - [Gateway SKUs](../user/azure-stack-vpn-gateway-about-vpn-gateways.md#gateway-skus)
  - [Highly Available Connections](../user/azure-stack-vpn-gateway-about-vpn-gateways.md#gateway-availability)
  - [Configure BGP on Azure Stack Hub](../user/azure-stack-vpn-gateway-settings.md#gateway-requirements)
  - [ExpressRoute circuits](azure-stack-connect-expressroute.md)
  - [Specify custom IPsec/IKE policies](../user/azure-stack-vpn-gateway-settings.md#ipsecike-parameters)
  
### Load Balancer

#### Load Balancer directing traffic to one backend VM in specific scenarios

- Applicable: This issue applies to all supported releases. 
- Cause: When enabling **Session Affinity** on a load balancer, the 2 tuple hash utilizes the PA IP (Physical Address IP) instead of the private IPs assigned to the VMs. In scenarios where traffic directed to the load balancer arrives through a VPN, or if all the client VMs (source IPs) reside on the same node and Session Affinity is enabled, all traffic is directed to one backend VM.
- Occurrence: Common

#### Public IP is in a failed state

- Applicable: This issue applies to all supported releases.
- Cause: The **IdleTimeoutInMinutes** value for a public IP that is associated to a load balancer cannot be changed. The operation puts the public IP into a failed state.
- Remediation: To bring the public IP back into a successful state, change the **IdleTimeoutInMinutes** value on the load balancer rule that references the public IP back to the original value (the default is 4 minutes).
- Occurrence: Common

## Compute

### Create or delete VM

#### Issues deploying virtual machine scale set with Standard_DS2_v2 size using the portal

- Applicable: This issue applies to the 2005 release.
- Cause: A portal bug causes scale set creation with Standard_DS2_v2 size to fail.
- Remediation: Use PowerShell or CLI to deploy this virtual machine scale set VM size.

#### NVv4 deployment fails

- Applicable: This issue applies to 2002 and later.
- Cause: When going through the VM creation experience, you will see the VM size: NV4as_v4. Customers who have the hardware required for the AMD MI25-based Azure Stack Hub GPU preview are able to have a successful VM deployment. All other customers will have a failed VM deployment with this VM size.
- Remediation: By design in preparation for the Azure Stack Hub GPU preview.

#### Subscription is at capacity for Total Regional vCPUs

- Applicable: This issue applies to all supported releases.
- Cause: When creating a new virtual machine, you may receive an error such as **This subscription is at capacity for Total Regional vCPUs on this location. This subscription is using all 50 Total Regional vCPUs available.**. This indicates that the quota for total cores available to you has been reached.
- Remediation: Ask your operator for an add-on plan with additional quota. Editing the current plan's quota will not work or reflect increased quota.
- Occurrence: Rare

### Stop or start VM

#### Stop-Deallocate VM results in MTU configuration removal

- Applicable: This issue applies to all supported releases.
- Cause: Performing **Stop-Deallocate** on a VM results in MTU configuration on the VM to be removed. This behavior is inconsistent with Azure.
- Occurrence: Common

### Configuration and setup

#### VM overview blade does not show correct computer name

- Applicable: This issue applies to all releases.
- Cause: When viewing details of a VM in the overview blade, the computer name shows as **(not available)**. This behavior is by design for VMs created from specialized disks/disk snapshots, and appears for Marketplace images as well.
- Remediation: View the **Properties** blade under **Settings**.

### Extensions

#### VM extensions fail on Ubuntu Server 20.04

- Applicable: This issue applies to **Ubuntu Server 20.04 LTS**.
- Cause: Some Linux distributions have transitioned to Python 3.8 and removed the legacy `/usr/bin/python` entrypoint for Python altogether. Linux distribution users who have transitioned to Python 3.x must ensure the legacy `/usr/bin/python` entry point exists before attempting to deploy those extensions to their VMs. Otherwise, the extension deployment might fail.
- Remediation: Follow the resolution steps in [Issues using VM extensions in Python 3-enabled Linux Azure Virtual Machines systems](/azure/virtual-machines/extensions/issues-using-vm-extensions-python-3) but skip step 2, because Azure Stack Hub does not have the **Run command** functionality.

## Storage

### Retention period reverts to 0

- Applicable: This issue applies to releases 2002 and 2005.
- Cause: If you specify a time period other than 0 in the retention period setting, it reverts to 0 (the default value of this setting) during the 2002 or 2005 update. The 0 days setting takes effect immediately after the update finishes, which causes all existing deleted storage accounts and any upcoming newly deleted storage accounts to be immediately out of retention and marked for periodic garbage collection (which runs hourly).
- Remediation: Manually specify the retention period to a correct period. Any storage account that has already been garbage collected before the new retention period is specified, is not recoverable.  

## Resource providers

### SQL/MySQL

- Applicable: This issue applies to release 2002.
- Cause: If the stamp contains SQL resource provider (RP) version 1.1.33.0 or earlier, upon update of the stamp, the blades for SQL/MySQL do not load.
- Remediation: Update the RP to version 1.1.47.0.

### App Service

- Applicable: This issue applies to release 2002.
- Cause: If the stamp contains App Service resource provider (RP) version 1.7 and older, upon update of the stamp, the blades for App Service do not load.
- Remediation: Update the RP to version 2002 Q2.

## PowerShell

[!Include[Known issue for install - one](../includes/known-issue-az-install-1.md)]

[!Include[Known issue for install - two](../includes/known-issue-az-install-2.md)]

<!-- ## Storage -->
<!-- ## SQL and MySQL-->
<!-- ## App Service -->
<!-- ## Usage -->
<!-- ### Identity -->
<!-- ### Marketplace -->
::: moniker-end

::: moniker range=">=azs-2005"
## Archive

To access archived known issues for an older version, use the version selector dropdown above the table of contents on the left, and select the version you want to see.

## Next steps

- [Review update activity checklist](release-notes-checklist.md)
- [Review list of security updates](release-notes-security-updates.md)
::: moniker-end

<!------------------------------------------------------------>
<!------------------- UNSUPPORTED VERSIONS ------------------->
<!------------------------------------------------------------>
::: moniker range="azs-2002"
## 2002 archived known issues
::: moniker-end
::: moniker range="azs-1910"
## 1910 archived known issues
::: moniker-end
::: moniker range="azs-1908"
## 1908 archived known issues
::: moniker-end
::: moniker range="azs-1907"
## 1907 archived known issues
::: moniker-end
::: moniker range="azs-1906"
## 1906 archived known issues
::: moniker-end
::: moniker range="azs-1905"
## 1905 archived known issues
::: moniker-end
::: moniker range="azs-1904"
## 1904 archived known issues
::: moniker-end
::: moniker range="azs-1903"
## 1903 archived known issues
::: moniker-end
::: moniker range="azs-1902"
## 1902 archived known issues
::: moniker-end
::: moniker range="azs-1901"
## 1901 archived known issues
::: moniker-end
::: moniker range="azs-1811"
## 1811 archived known issues
::: moniker-end
::: moniker range="azs-1809"
## 1809 archived known issues
::: moniker-end
::: moniker range="azs-1808"
## 1808 archived known issues
::: moniker-end
::: moniker range="azs-1807"
## 1807 archived known issues
::: moniker-end
::: moniker range="azs-1805"
## 1805 archived known issues
::: moniker-end
::: moniker range="azs-1804"
## 1804 archived known issues
::: moniker-end
::: moniker range="azs-1803"
## 1803 archived known issues
::: moniker-end
::: moniker range="azs-1802"
## 1802 archived known issues
::: moniker-end

::: moniker range="<azs-2005"
You can access older versions of Azure Stack Hub known issues in the table of contents on the left side, under the [Resources > Release notes archive](./relnotearchive/known-issues.md). Select the desired archived version from the version selector dropdown in the upper left. These archived articles are provided for reference purposes only and do not imply support for these versions. For information about Azure Stack Hub support, see [Azure Stack Hub servicing policy](azure-stack-servicing-policy.md). For further assistance, contact Microsoft Customer Support Services.
::: moniker-end
