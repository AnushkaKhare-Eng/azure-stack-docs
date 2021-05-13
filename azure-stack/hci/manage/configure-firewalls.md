---
title: Configure firewalls for Azure Stack HCI
description: This topic provides guidance on how to configure firewalls for the Azure Stack HCI operating system.
author: JohnCobb1
ms.author: v-johcob
ms.topic: how-to
ms.date: 05/13/2021
---

# Configure firewalls for Azure Stack HCI

>Applies to: Azure Stack HCI, version 20H2

This topic provides guidance on how to configure firewalls for the Azure Stack HCI operating system. It includes connectivity requirements, and explains how service tags group IP addresses in Azure that the operating system needs to access. The topic also provides steps to update Microsoft Defender Firewall.

## Azure connectivity requirements
Azure Stack HCI needs to periodically connect to Azure. Access is limited to only:
- Well-known Azure IPs
- Outbound direction
- Port 443 (HTTPS)

For more information, see the "Azure Stack HCI connectivity" section of the [Azure Stack HCI FAQ](../faq.yml)

This topic describes how to optionally use a highly locked-down firewall configuration to block all traffic to all destinations except those included on your allowlist.

   >[!IMPORTANT]
   > If outbound connectivity is restricted by your external corporate firewall or proxy server, ensure that the URLs listed in the table below are not blocked. For related information, see the "Networking configuration" section of [Overview of Azure Arc enabled servers agent](/azure/azure-arc/servers/agent-overview#networking-configuration).

As shown below, Azure Stack HCI accesses Azure using more than one firewall potentially.

:::image type="content" source="./media/configure-firewalls/firewalls-diagram.png" alt-text="Diagram shows Azure Stack HCI accessing service tag endpoints through Port 443 (HTTPS) of firewalls." lightbox="./media/configure-firewalls/firewalls-diagram.png":::

## Microsoft Update connectivity requirements
If there is a corporate firewall between the operating system and the internet, you might have to configure that firewall to ensure the operating system can obtain updates. To obtain updates from Microsoft Update, the operating system uses port 443 for the HTTPS protocol. Although most corporate firewalls allow this type of traffic, some companies restrict internet access due to their security policies. If your company restricts access, you'll need to obtain authorization to allow internet access to the following URLs:

- http\://windowsupdate.microsoft.com
- http\://\*.windowsupdate.microsoft.com
- https\://\*.windowsupdate.microsoft.com
- http\://\*.update.microsoft.com
- https\://\*.update.microsoft.com
- http\://\*.windowsupdate.com
- http\://download.windowsupdate.com
- https\://download.microsoft.com
- http\://\*.download.windowsupdate.com
- http\://wustat.windows.com
- http\://ntservicepack.microsoft.com
- http\://go.microsoft.com
- http\://dl.delivery.mp.microsoft.com
- https\://dl.delivery.mp.microsoft.com

## Working with service tags
A *service tag* represents a group of IP addresses from a given Azure service. Microsoft manages the IP addresses included in the service tag, and automatically updates the service tag as IP addresses change to keep updates to a minimum. To learn more, see [Virtual network service tags](/azure/virtual-network/service-tags-overview).

## Required endpoint daily access (after Azure registration)
Azure maintains well-known IP addresses for Azure services that are organized using service tags. Azure publishes a weekly JSON file of all the IP addresses for every service. The IP addresses don’t change often, but they do change a few times per year. The following table shows the service tag endpoints that the operating system needs to access.

| Description                   | Service tag for IP range  | URL                                                                       | Azure China URL                         |
| :-----------------------------| :-----------------------  | :------------------------------------------------------------------------ | :-------------------------------------- |
| Azure Active Directory        | AzureActiveDirectory      | `https://login.microsoftonline.com`<br> `https://graph.microsoft.com`<br> `https://graph.windows.net`     | `https://login.partner.microsoftonline.cn`<br> `https://microsoftgraph.chinacloudapi.cn`<br> `https://graph.chinacloudapi.cn` |
| Azure Resource Manager        | AzureResourceManager      | `https://management.azure.com`                                            | `https://management.chinacloudapi.cn` |
| Azure Stack HCI Cloud Service | AzureFrontDoor.Frontend<br> AzureCloud.ChinaEast2 (Azure China) | `https://azurestackhci.azurefd.net` | `https://dp.stackhci.azure.cn` |
| Azure Arc                     | AzureArcInfrastructure<br> AzureTrafficManager | Depends on the functionality you want to use:<br> Hybrid Identity Service: `*.his.arc.azure.com`<br> Guest Configuration: `*.guestconfiguration.azure.com`<br> **Note:** Expect more URLs as we enable more functionality. | Coming soon. |

## Update Microsoft Defender Firewall
This section shows how to configure Microsoft Defender Firewall to allow IP addresses associated with a service tag to connect with the operating system:

1. Download the JSON file from the following resource to the target computer running the operating system: [Azure IP Ranges and Service Tags – Public Cloud](https://www.microsoft.com/download/details.aspx?id=56519).

1. Use the following PowerShell command to open the JSON file:

    ```powershell
    $json = Get-Content -Path .\ServiceTags_Public_20201012.json | ConvertFrom-Json
    ```

1. Get the list of IP address ranges for a given service tag, such as the “AzureResourceManager” service tag:

    ```powershell
    $IpList = ($json.values | where Name -Eq "AzureResourceManager").properties.addressPrefixes
    ```

1. Import the list of IP addresses to your external corporate firewall, if you're using an allowlist with it.

1. Create a firewall rule for each server in the cluster to allow outbound 443 (HTTPS) traffic to the list of IP address ranges:

    ```powershell
    New-NetFirewallRule -DisplayName "Allow Azure Resource Manager" -RemoteAddress $IpList -Direction Outbound -LocalPort 443 -Protocol TCP -Action Allow -Profile Any -Enabled True
    ```

## Additional endpoint for one-time Azure registration
During the Azure registration process, when you run either `Register-AzStackHCI` or use Windows Admin Center, the cmdlet tries to contact the PowerShell Gallery to verify that you have the latest version of required PowerShell modules, such as Az and AzureAD.

Although the PowerShell Gallery is hosted on Azure, currently there isn't a service tag for it. If you can't run the `Register-AzStackHCI` cmdlet from a server node because of no internet access, we recommend downloading the modules to your management computer, and then manually transferring them to the server node where you want to run the cmdlet.

## Set up a proxy server
To set up a proxy server for Azure Stack HCI, run the following PowerShell command as an administrator on each server in the cluster:

```powershell
Set-WinInetProxy -ProxySettingsPerUser 0 -ProxyServer webproxy1.com:9090
```

Use the `ProxySettingsPerUser 0` flag to make the proxy configuration server-wide instead of per user, which is the default.

To remove the proxy configuration, run the PowerShell command `Set-WinInetProxy` without arguments.

Download the WinInetProxy.psm1 script at: [PowerShell Gallery | WinInetProxy.psm1 0.1.0](https://www.powershellgallery.com/packages/WinInetProxy/0.1.0/Content/WinInetProxy.psm1).

   >[!NOTE]
   > Using the **Proxy** setting in Windows Admin Center redirects all Windows Admin Center outbound traffic (for example, download extensions, connecting to Azure and so on).

## Network port requirements
Ensure that the proper network ports are open between all server nodes both within a site and between sites (for stretched clusters). You'll need appropriate firewall and router rules to allow ICMP, SMB (port 445, plus port 5445 for SMB Direct), and WS-MAN (port 5985) bi-directional traffic between all servers in the cluster.

When using the Cluster Creation wizard in Windows Admin Center to create the cluster, the wizard automatically opens the appropriate firewall ports on each server in the cluster for Failover Clustering, Hyper-V, and Storage Replica. If you're using a different firewall on each server, open the following ports:

### Failover Clustering ports
- ICMPv4 and ICMPv6
- TCP port 445
- RPC Dynamic Ports
- TCP port 135
- TCP port 137
- TCP port 3343
- UDP port 3343

### Hyper-V ports
- TCP port 135
- TCP port 80 (HTTP connectivity)
- TCP port 443 (HTTPS connectivity)
- TCP port 6600
- TCP port 2179
- RPC Dynamic Ports
- RPC Endpoint Mapper
- TCP port 445

### Storage Replica ports (stretched cluster)
- TCP port 445
- TCP 5445 (if using iWarp RDMA)
- TCP port 5985
- ICMPv4 and ICMPv6 (if using the `Test-SRTopology` PowerShell cmdlet)

## Next steps
For more information, see also:
- The connectivity section of the [Azure Stack HCI FAQ](../faq.yml)
