---
title: Azure Container Registries supported commands 
description: Review the list of supported commands for CLI and PowerShell for Azure Container Registries on Azure Stack Hub.
author: mattbriggs
ms.topic: reference
ms.date: 08/20/2021
ms.author: mabrigg
ms.reviewer: chasat
ms.lastreviewed: 08/20/2021

# Intent: As an Azure Stack user, I want to XXX so I can XXX.
# Keyword: XXX

---

# Azure Container Registry supported commands on Azure Stack Hub

The Azure Container Registry (ACS) on Azure Stack support a subset of the global Azure
features. You can find out more information in the [Overview](container-registry-overview.md).

## Supported Azure Container Registry CLI Commands

| Command | Description |
|---|---|
| az acr check-health | Gets health information on the environment and optionally a target registry. |
| az acr check-name | Checks if an Azure Container Registry name is valid and available for use. |
| az acr create | Create an Azure Container Registry. |
| az acr credential renew | Regenerate login credentials for an Azure Container Registry. |
| az acr credential show | Get the login credentials for an Azure Container Registry. |
| az acr delete | Deletes an Azure Container Registry. |
| az acr import | Imports an image to an Azure Container Registry from another Container Registry. Import removes the need to docker pull, docker tag, docker push. |
| az acr list | Lists all the container registries under the current subscription. |
| az acr login | Log in to an Azure Container Registry through the Docker CLI. |
| az acr repository delete | Delete a repository or image in an Azure Container Registry. |
| az acr repository list | List repositories in an Azure Container Registry. |
| az acr repository show | Get the attributes of a repository or image in an Azure Container Registry. |
| az acr repository show-manifests | Show manifests of a repository in an Azure Container Registry. |
| az acr repository show-tags | Show tags for a repository in an Azure Container Registry. |
| az acr repository untag | Untag an image in an Azure Container Registry. |
| az acr repository update | Update the attributes of a repository or image in an Azure Container Registry. |
| az acr show | Get the details of an Azure Container Registry. |
| az acr show-usage | Get the storage usage for an Azure Container Registry. |
| az acr update | Update an Azure Container Registry. |
| az acr webhook create | Create a webhook for an Azure Container Registry. |
| az acr webhook delete | Delete a webhook from an Azure Container Registry. |
| az acr webhook get-config | Get the service URI and custom headers for the webhook. |
| az acr webhook list | List all of the webhooks for an Azure Container Registry. |
| az acr webhook list-events | List recent events for a webhook. |
| az acr webhook ping | Trigger a ping event for a webhook. |
| az acr webhook show | Get the details of a webhook. |
| az acr webhook update | Update a webhook. |

### Unsupported optional parameters

Some supported commands have optional parameters that are not supported on Azure Stack Hub. The unsupported parameters are listed below.

**az acr create**
 - `--allow-trusted-services`  
 - `--default-action`  
 - `--identity`  
 - `--key-encryption-key`  
 - `--public-network-enabled`  
 - `--workspace`  
 - `--zone-redundancy`  

**az acr update**
 - `--allow-trusted-services`
 - `--anonymous-pull-enabled`
 - `--data-endpoint-enabled`
 - `--default-action`
 - `--public-network-enabled`
 - `--sku`

## Supported Azure Container Registry PowerShell commands for public preview

| Command | Description |
|---|---|
| [Connect-AzContainerRegistry](/powershell/module/az.containerregistry/connect-azcontainerregistry?view=azps-5.9.0)                           | Log in to an Azure container registry.                                     |
| [Get-AzContainerRegistry](/powershell/module/az.containerregistry/get-azcontainerregistry?view=azps-5.9.0)                                   | Gets a container registry.                                                |
| [Get-AzContainerRegistryCredential](/powershell/module/az.containerregistry/get-azcontainerregistrycredential?view=azps-5.9.0)               | Gets the login credentials for a container registry.                      |
| [Get-AzContainerRegistryManifest](/powershell/module/az.containerregistry/get-azcontainerregistrymanifest?view=azps-5.9.0)                   | Get or list ACR manifest.                                                 |
| [Get-AzContainerRegistryRepository](/powershell/module/az.containerregistry/get-azcontainerregistryrepository?view=azps-5.9.0)               | Get or list ACR repositories.                                             |
| [Get-AzContainerRegistryTag](/powershell/module/az.containerregistry/get-azcontainerregistrytag?view=azps-5.9.0)                             | Get or list ACR tag.                                                      |
| [Get-AzContainerRegistryUsage](/powershell/module/az.containerregistry/get-azcontainerregistryusage?view=azps-5.9.0)                         | Get Usage of an Azure container registry.                                 |
| [Get-AzContainerRegistryWebhook](/powershell/module/az.containerregistry/get-azcontainerregistrywebhook?view=azps-5.9.0)                     | Gets a container registry webhook.                                        |
| [Get-AzContainerRegistryWebhookEvent](/powershell/module/az.containerregistry/get-azcontainerregistrywebhookevent?view=azps-5.9.0)           | Gets events of a container registry webhook.                              |
| [Import-AzContainerRegistryImage](/powershell/module/az.containerregistry/import-azcontainerregistryimage?view=azps-5.9.0)                   | Import image from a global Azure registry to an Azure container registry. |
| [New-AzContainerRegistry](/powershell/module/az.containerregistry/new-azcontainerregistry?view=azps-5.9.0)                                   | Creates a container registry.                                             |
| [New-AzContainerRegistryWebhook](/powershell/module/az.containerregistry/new-azcontainerregistrywebhook?view=azps-5.9.0)                     | Creates a container registry webhook.                                     |
| [Remove-AzContainerRegistry](/powershell/module/az.containerregistry/remove-azcontainerregistry?view=azps-5.9.0)                             | Removes a container registry.                                             |
| [Remove-AzContainerRegistryManifest](/powershell/module/az.containerregistry/remove-azcontainerregistrymanifest?view=azps-5.9.0)             | Delete ACR manifest.                                                      |
| [Remove-AzContainerRegistryRepository](/powershell/module/az.containerregistry/remove-azcontainerregistryrepository?view=azps-5.9.0)         | Delete repository from ACR.                                               |
| [Remove-AzContainerRegistryTag](/powershell/module/az.containerregistry/remove-azcontainerregistrytag?view=azps-5.9.0)                       | Untag ACR tag.                                                            |
| [Remove-AzContainerRegistryWebhook](/powershell/module/az.containerregistry/remove-azcontainerregistrywebhook?view=azps-5.9.0)               | Removes a container registry webhook.                                     |
| [Test-AzContainerRegistryNameAvailability](/powershell/module/az.containerregistry/test-azcontainerregistrynameavailability?view=azps-5.9.0) | Checks the availability of a container registry name.                     |
| [Test-AzContainerRegistryWebhook](/powershell/module/az.containerregistry/test-azcontainerregistrywebhook?view=azps-5.9.0)                   | Triggers a webhook ping event.                                            |
| [Update-AzContainerRegistry](/powershell/module/az.containerregistry/update-azcontainerregistry?view=azps-5.9.0)                             | Updates a container registry.                                             |
| [Update-AzContainerRegistryCredential](/powershell/module/az.containerregistry/update-azcontainerregistrycredential?view=azps-5.9.0)         | Regenerates a login credential for a container registry.                  |
| [Update-AzContainerRegistryManifest](/powershell/module/az.containerregistry/update-azcontainerregistrymanifest?view=azps-5.9.0)             | Update ACR manifest.                                                      |
| [Update-AzContainerRegistryRepository](/powershell/module/az.containerregistry/update-azcontainerregistryrepository?view=azps-5.9.0)         | Update ACR repository.                                                    |
| [Update-AzContainerRegistryTag](/powershell/module/az.containerregistry/update-azcontainerregistrytag?view=azps-5.9.0)                       | Update ACR tag.                                                           |
| [Update-AzContainerRegistryWebhook](/powershell/module/az.containerregistry/update-azcontainerregistrywebhook?view=azps-5.9.0)               | Updates a container registry webhook.                                     |

## Next steps

Learn more about the [Azure Container Registry on Azure Stack Hub](container-registry-overview.md)