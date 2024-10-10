--- 
title: Configure Arc proxy manually for Azure gateway on Azure Local, version 2408 and 2408.1 (preview)
description: Learn how to configure Arc proxy manually for Azure gateway on Azure Local, version 2408 and 2408.1 (preview). 
author: alkohli
ms.topic: how-to
ms.date: 10/07/2024
ms.author: alkohli
ms.service: azure-stack-hci
---

# Configure Arc proxy manually for Azure gateway on Azure Local (preview)

Applies to: Azure Local, version 23H2, release 2408.1 and 2408

After creating the Arc gateway resource in your Azure subscription, you can enable the new Arc gateway preview features. This article details how to manually configure the Arc proxy before Arc registration.

[!INCLUDE [important](../../hci/includes/hci-preview.md)]

## Prerequisites

Make sure the following prerequisites are met before proceeding:

- You’ve access to an Azure Local, version 23H2 system.

- An Arc gateway resource created in the same subscription as used to deploy Azure Local. For more information, see [Create the Arc gateway resource in Azure](deployment-azure-arc-gateway-overview.md#create-the-arc-gateway-resource-in-azure).

> [!Warning]
> Only the standard ISO OS image available at https://aka.ms/PVenEREWEEW should be used to test the Arc gateway public preview on Azure Local, version 2408. Do not use the ISO image available in Azure portal.

## Step 1: Manually configure the proxy on each node

If you need to configure the Arc proxy on your Azure Local machines before starting the Arc registration process, follow the instructions at [Configure proxy settings for Azure Local, version 23H2](../manage/configure-proxy-settings-23h2.md).

Ensure that you configure the proxy and the bypass list for all the nodes on your cluster.

## Step 2: Get the ArcGatewayID  

You need the proxy and the Arc gateway ID (ArcGatewayID) from Azure to run the registration script on Azure Local machines. To get the ArcGatewayID, run the following `az connectedmachine gateway list` command from any computer that is not an Azure Local machine.

Here's an example:

```azurecli
PS C:\> az connectedmachine gateway list 

This command is in preview and under development. Reference and support levels: https://aka.ms/CLI_refstatus 

[ 
  { 
    "allowedFeatures": [ 
      "*" 
    ], 
    "gatewayEndpoint": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx.gw.arc.azure.com", 
    "gatewayId": "xxxxxxx-xxxx-xxx-xxxx-xxxxxxxxx", 
    "gatewayType": "Public", 
    "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx/resourceGroups/yourresourcegroup/providers/Microsoft.HybridCompute/gateways/yourArcgateway", 
    "location": "eastus", 
    "name": " yourArcgateway", 
    "provisioningState": "Succeeded", 
    "resourceGroup": "yourresourcegroup", 
    "type": "Microsoft.HybridCompute/gateways" 
  } 
] 
```

## Step 3: Register new nodes in Azure Arc

You run the initialization script by passing the ArcGatewayID parameter and the proxy server parameters. Here's an example of how you should change the `Invoke-AzStackHciArcInitialization` parameters on the initialization script:

```azurecli
#Install required PowerShell modules in your node for registration 

Install-Module Az.Accounts -RequiredVersion 2.13.2 

Install-Module Az.Resources -RequiredVersion 6.12.0 

Install-Module Az.ConnectedMachine -RequiredVersion 0.5.2 

#Install Arc registration script from PSGallery  

Install-Module AzsHCI.ARCinstaller 

#Define the subscription where you want to register your machine as Arc device 

$Subscription = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx" 

#Define the resource group where you want to register your machine as Arc device 

$RG = "yourresourcegroupname" 

#Define the tenant you will use to register your machine as Arc device 

$Tenant = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx" 

#Define Proxy Server if necessary 

$ProxyServer = "http://x.x.x.x:port" 

#Define the Arc gateway resource ID from Azure 

$ArcgwId = "/subscriptions/xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx /resourceGroups/ yourresourcegroupname /providers/Microsoft.HybridCompute/gateways/yourarcgatewayname" 

#Connect to your Azure account and Subscription 

Connect-AzAccount -SubscriptionId $Subscription -TenantId $Tenant -DeviceCode 

#Get the Access Token and Account ID for the registration 

$ARMtoken = (Get-AzAccessToken).Token 

#Get the Account ID for the registration 

$id = (Get-AzContext).Account.Id 

#Invoke the registration script with Proxy and ArcgatewayID 

Invoke-AzStackHciArcInitialization -SubscriptionID $Subscription -ResourceGroup $RG -TenantID $Tenant -Region australiaeast -Cloud "AzureCloud" -ArmAccessToken $ARMtoken -AccountID $id -Proxy $ProxyServer -ArcGatewayID $ArcgwId 
```

## Step 4: Start Azure Local cloud deployment

Once the Azure Local machines are registered in Azure Arc and all the extensions are installed, you can start deployment from Azure portal or using the ARM templates that are documented in these articles:

- [Deploy an Azure Local instance using the Azure portal](deploy-via-portal.md).

- [Azure Resource Manager template deployment for Azure Local, version 23H2](deployment-azure-resource-manager-template.md).

## Step 5: Verify that the setup succeeded

Once the deployment validation starts, you can connect to the first Azure Local machine from your cluster and open the Arc gateway log to monitor which endpoints are redirected to the Arc gateway and which ones continue using your firewall or proxy.

You can find the Arc gateway log at: *c:\programdata\AzureConnectedMAchineAgent\Log\arcproxy.log*.

:::image type="content" source="./media/deployment-connect-nodes-to-arc-gateway/arc-gateway-log.png" alt-text="Screenshot that shows the Arc gateway log using manual method." lightbox="./media/deployment-connect-nodes-to-arc-gateway/arc-gateway-log.png":::

To check the Arc agent configuration and verify that it is using the Arc gateway, run the following command: `c:\program files\AzureConnectedMachineAgent>.\azcmagent show`

The result should show the following values:

- **Agent version** is **1.45**.

- **Agent Status** is **Connected**.

- **Using HTTPS Proxy**  is empty when Arc gateway isn't in use. It should show as `http://localhost:40343` when the Arc gateway is enabled.

- **Upstream Proxy** shows your enterprise proxy server and port.

- **Azure Arc Proxy** shows as **stopped** when Arc gateway isn't in use, and **running** when the Arc gateway is enabled.

The Arc agent without the Arc gateway:

:::image type="content" source="./media/deployment-connect-nodes-to-arc-gateway/arc-agent-without-gateway.png" alt-text="Screenshot that shows the Arc agent without gateway using manual method." lightbox="./media/deployment-connect-nodes-to-arc-gateway/arc-agent-without-gateway.png":::

The Arc agent using the Arc gateway:

:::image type="content" source="./media/deployment-connect-nodes-to-arc-gateway/arc-agent-with-gateway.png" alt-text="Screenshot that shows the Arc agent with gateway using manual method." lightbox="./media/deployment-connect-nodes-to-arc-gateway/arc-agent-with-gateway.png":::

Additionally, to verify that the setup successful, you can run the following command: `c:\program files\AzureConnectedMachineAgent>.\azcmagent check`.

The response should indicate that `connection.type` is set to `gateway`, and the **Reachable** column should indicate **true** for all URLs, as shown:

The Arc agent without the Arc gateway:

:::image type="content" source="./media/deployment-connect-nodes-to-arc-gateway/arc-agent-without-gateway-2.png" alt-text="Screenshot that shows the Arc agent without gateway 2 using manual method." lightbox="./media/deployment-connect-nodes-to-arc-gateway/arc-agent-without-gateway-2.png":::

The Arc agent with the Arc gateway enabled:

:::image type="content" source="./media/deployment-connect-nodes-to-arc-gateway/arc-agent-with-gateway-2.png" alt-text="Screenshot that shows the Arc agent with gateway 2 using manual method." lightbox="./media/deployment-connect-nodes-to-arc-gateway/arc-agent-without-gateway-2.png":::

You can also audit your gateway traffic by viewing the gateway router logs.  

To view gateway router logs on Windows, run the `azcmagent logs` command in PowerShell. In the resulting .zip file, the logs are located in the *C:\ProgramData\Microsoft\ArcGatewayRouter* folder.

## Next steps

- [Get support for deployment issues](../manage/get-support-for-deployment-issues.md)
- [Get support for Azure Local](../manage/get-support.md)
