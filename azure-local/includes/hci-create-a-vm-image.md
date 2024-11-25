---
author: ronmiab
ms.author: robess
ms.service: azure-stack-hci
ms.topic: include
ms.date: 11/25/2024
---

Follow these steps on your Azure Local to create the VM image from the VHDX that you created earlier.

Use the Azure CLI to create the VM image:

1. Run PowerShell as an administrator.

1. Sign in. Run the following cmdlet:

    ```azurecli
    az login
    ```

1. Set your subscription. Run the following cmdlet:

    ```azurecli
    az account set --subscription <Subscription ID>
    ```

1. Set parameters for your subscription, resource group, custom location, location, OS type for the image, name of the image, and the path where the image is located. Replace the parameters in `< >` with the appropriate values.

    ```azurecli
    $subscription = "<Subscription ID>"
    $resource_group = "<Resource group>"
    $customLocation = "<Custom location>"
    $location = "<Location for your Azure Local>"
    $OsType = "<OS of source image>"
    ```

    Parameters are described in the following table.

    | Parameter      | Description                                                                                |
    |----------------|--------------------------------------------------------------------------------------------|
    | `subscription`   | Subscription associated with your Azure Local instance.        |
    | `resource_group` | Resource group for the Azure Local instance that you associate with this image.        |
    | `location`       | Location for your Azure Local instance. For example, the location could be `eastus` or `westreurope`. |
    | `os-Type`         | Operating system associated with the source image. This system can be Windows or Linux.           |

1. Use the VHDX of the VM to create a gallery image. Use this VM image to create Azure Arc virtual machines on Azure Local.

    Make sure to copy the VHDX in user storage in the cluster shared volume of Azure Local. For example, the path could look like `C:\ClusterStorage\UserStorage_1\linuxvhdx`.

    ```powershell
    $imagePath = "Path to user storage in CSV" 

    $imageName = "mylinuxvmimg" 

    az stack-hci-vm image create --subscription $subscription -g $resource_group --custom-location $customLocation --location $location --image-path $imagePath --name $imageName --debug --os-type 'Linux' 
    ```

1. Verify that the image is created.
