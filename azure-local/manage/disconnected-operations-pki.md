---
title: Understand Public Key Infrastructure (PKI) requirements for disconnected operations on Azure Local (preview)
description: Learn about the public key infrastructure (PKI) requirements for disconnected operations on Azure Local and how to create the necessary certificates to secure the endpoints provided by the disconnected operations appliance (preview).
ms.topic: concept-article
author: ronmiab
ms.author: robess
ms.date: 04/22/2025
---

# Public Key Infrastructure (PKI) for disconnected operations on Azure Local (preview)

::: moniker range=">=azloc-24112"

[!INCLUDE [applies-to](../includes/release-2411-1-later.md)]

This document describes your local public key infrastructure (PKI) requirements and shows you how to create the necessary certificates to secure the endpoints provided by the disconnected operations appliance.

[!INCLUDE [IMPORTANT](../includes/disconnected-operations-preview.md)]

## About PKI

PKI for disconnected operations is essential for securing the endpoints provided by the disconnected operations appliance. You create and manage digital certificates to ensure secure communication and data transfer within your Azure Local environment.

## PKI requirements

Public certificate authorities (CA) or enterprise certificate authorities must issue certificates. Ensure that your certificates are part of the Microsoft Trusted Root Program. For more information, see [List of Participants - Microsoft Trusted Root Program](/security/trusted-root/participants-list).  

Mandatory certificates are grouped by area with the appropriate subject alternate names (SAN). Before you create the certificates, make sure you understand the following requirements:

- The use of self-signed certificates isn’t supported. We recommend you use certificates issued by an enterprise CA.
- Disconnected operations requires 26 external certificates for the endpoints it exposes.
- Individual certificates should be generated for each endpoint and copied into the corresponding directory/folder structure. These Certificates are mandatory for disconnected operations deployment.
- All certificates must have Subject and SAN defined, as required by most browsers.
- All certificates should share the same trust chain and should at least have a two-year expiration from the day of the deployment.
- All root certificates should be exported in Base64 encoded format. The resulting file typically has a .cer, .crt, or .pem extension.

### Ingress endpoints

This table lists the mandatory certificates required for disconnected operations on Azure Local.

| Service | Required certificate subject and subject alternative names (SAN) |
|-------------------|----------------------|  
| Azure Container Registry | *.edgeacr.fqdn |
| Appliances | dp.appliances.fqdn <br></br> adminmanagement.fqdn |
| Azure Data Policy | data.policy.fqdn |
| Arc configuration data plane |dp.kubernetesconfiguration.fqdn |
| Arc for Server Agent data service | agentserviceapi.fqdn |
| Arc for server | his.fqdn |
| Arc guest notification service | guestnotificationservice.fqdn |
| Arc metrics | metricsingestiongateway.monitoring.fqdn |
| Arc monitor agent | amcs.monitoring.fqdn |
| Azure Arc resource bridge data plane | dp.appliances.fqdn |
| Azure Resource Manager | armmanagement.fqdn |
| Azure Arc-enabled Kubernetes | autonomous.dp.kubernetesconfiguration.fqdn |
| Azure queue storage | *.queue.fqdn |
| Azure Resource Manager Public | armmanagement.fqdn |
| Azure Table storage | *.table.fqdn |
| Azure Blob storage | *.blob.fqdn |
| Front end appliances | frontend.appliances.fqdn |
| Front end monitoring | pqsqueryfrontend.monitoring.fqdn |
| Graph | graph.fqdn |
| Azure Key Vault | *.vault.<'fqdn'> (wildcard Secure Sockets Layer (SSL) certificate) |
| Kubernetes configuration | dp.kubernetesconfiguration.fqdn |
| Licensing | licensing.aszrp.fqdn <br></br> dp.aszrp.fqdn <br></br> lbc.fqdn |
| Managed Arc proxy services (MAPS) Azure Kubernetes Service (AKS) | *.k8sconnect.fqdn |
| Public extension host | *.hosting.fqdn (wildcard SSL certificate) |
| Public portal     | portal.fqdn <br></br> hosting.fqdn <br></br> portalcontroller.fqdn <br></br> catalogapi.fqdn |
| Secure token service | login.fqdn |
| Service bus | *.servicebus.fqdn |

### Management endpoints

The management endpoint requires two certificates and they must be put in the same folder, *ManagementEndpointCerts*. The certificates are:

| Management endpoint certificate  | Required certificate subject  |
|----------------------|------------------|
| Server  | Management endpoint IP address: $ManagementIngressIpAddress. <br></br> If the management endpoint IP is **192.168.100.25**, then the server certificate’s subject name must match exactly. For example, **Subject = 192.168.100.25**|
| Client  | Use a certificate subject that helps you distinguish it from others. Any string is acceptable. <br></br> For example, **Subject = ManagementEndpointClientAuth**.  |

## Create certificates to secure endpoints

On the host machine or Active Directory virtual machine (VM), follow the steps in this section to create certificates for the ingress traffic and external endpoints of the disconnected operations appliance. Make sure you modify for each of the 26 certificates.

You need these certificates when deploying the disconnected operations appliance. Additionally, you need the public key for your local infrastructure to provide a secure trust chain.

> [!NOTE]
> **IngressEndpointCerts** is the folder where all 26 certificate files should be stored. **IngressEndpointPassword** is a secure string with the certificate password.


1. Connect to the CA.
1. Create a folder named **IngressEndpointsCerts**. Use this folder to store all certificates.
1. Create a Certificate Signing Request (CSR).

    ```PowerShell
    [CmdletBinding()]   
    param (   
        [Parameter(Mandatory = $true)]   
        [string]   
        $certSubject = "CN=*.autonomous.cloud.private",   
        [Parameter(Mandatory = $true)]   
        [string]   
        $extCertFilePath    
    )   

1. Define parameters for creating the CSR.

    ```PowerShell 
    #$certSubject = "CN=*.contoso-disconnected.com"  
    #$certSubject = "CN=*.autonomous.cloud.private"  
    $dns = $certSubject.Split('=')[1]
    $filePrefix = $dns.Replace('*.','')
    $certFilePath = Split-Path -Path $extCertFilePath
    mkdir $certFilePath -Force
    rm "$certFilePath\$filePrefix.*" -Force
    $csrPath = Join-Path -Path $certFilePath -ChildPath "$filePrefix.csr"
    $infPath = Join-Path -Path $certFilePath -ChildPath "$filePrefix.inf"
    ```

1. Create the .inf file.

    ```PowerShell
    @"
    [NewRequest]
    Subject = "$certSubject"
    KeySpec = 1
    KeyLength = 2048
    Exportable = TRUE
    MachineKeySet = TRUE
    SMIME = FALSE
    PrivateKeyArchive = FALSE
    UserProtected = FALSE
    UseExistingKeySet = FALSE
    ProviderName = "Microsoft RSA SChannel Cryptographic Provider"
    ProviderType = 12
    RequestType = PKCS10
    KeyUsage = 0xa0
    HashAlgorithm = sha256

    [Extensions]
    2.5.29.17 = "{text}"
    _continue_ = "DNS=$dns"
    "@ | Out-File -FilePath $infPath
    ```

1. [Create the CSR](/azure-stack/operator/azure-stack-get-pki-certs?view=azs-2408&tabs=omit-cn&pivots=csr-type-new-deployment&preserve-view=true) for each certificate.

    ```powershell
    certreq -new $infPath $csrPath
    Write-Verbose "CSR created and saved to $csrPath" -Verbose
    ```

1. Define parameters to submit the CSR

    ```PowerShell
    $certPath = Join-Path $certFilePath -ChildPath "$filePrefix.cer"   
    $caName = "contoso-disonnected.com\contoso-disconnected.com" # Modify for FQDN   
    ```

1. Submit the CSR to the Enterprise CA.

    ```powershell
    certreq -submit -attrib "CertificateTemplate:WebServer" -config $caName $csrPath $certPath

    Write-Verbose "Certificate request submitted. Certificate saved to $certPath" -Verbose
    ```

1. Accept the certificate and install it.

    ```powershell
    $certReqOutput = certreq.exe -accept $certPath
    ```

1. Parse the thumbprint and export the certificate.

    ```powershell
    $match = $certReqOutput -match 'Thumbprint:\s*([a-fA-F0-9]+)'

    if ($null -ne $match) {
        $thumbprint = (($match[0]).Split(':')[1]).Trim()
        Write-Verbose "Thumbprint: $thumbprint" -Verbose
    } else {
        Write-Verbose "Thumbprint not found" -Verbose
        #return;
    }
    ```

1. Save the .pfx file in the **IngressEndpointsCerts** folder. The IngressEndpointPassword is a secure string with the certificate password.

    ```powershell
    $cert = Get-Item -Path "Cert:\LocalMachine\My\$thumbprint"
    $cert | Export-PfxCertificate -FilePath $extCertFilePath -Password (ConvertTo-SecureString -String "…" -Force -AsPlainText)
    Write-Verbose "Certificate for $certSubject and private key exported to $extCertFilePath" -Verbose
    ```

1. Repeat steps 3-11 for each certificate.

1. Copy the original certificate (26 .pfx files) obtained from your CA to the directory structure represented in IngressEndpointCerts.

Here's an example script to create the certificates for the management endpoint:

```powershell
Populate cert chain info 
# Resolve certificates from endpoint

# https://gist.github.com/keystroke/643bcbc449b3081544c7e6db7db0bba8

#This script uses endpoints that needs to be modified

$oidcCertChain = Get-CertificateChainFromEndpoint https://adfs.contoso.com 
$ldapCertchain =  Get-CertificateChainFromEndpoint https://adfs2.contoso.com 

#First endpoint https://adfs.contoso.com 
$certificates = Get-CertificateChainFromEndpoint https://adfs.contoso.com #Please modify 

# Save certificates to specified folder 
$targetDirectory = 'C:\WinfieldExternalIdentityCertificates1' #Please modify 
foreach ($certificate in $certificates) 
{ 
    $bytes = $certificate.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Cert) 
    $bytes | Set-Content -Path "$targetDirectory\$($certificate.Thumbprint).cer" -Encoding Byte 
} 
 Prepping the parameters 

# Read certificate information for external identity configuration 
$targetDirectory = 'C:\WinfieldExternalIdentityCertificates1' 
$oidcCertChainInfo = @() 
foreach ($file in (Get-ChildItem $targetDirectory)) 
{ 
    $certificate = [System.Security.Cryptography.X509Certificates.X509Certificate2]::new($file.FullName) 
    $bytes = $certificate.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Cert) 
    $b64 = [system.convert]::ToBase64String($bytes) 
    $oidcCertChainInfo += $b64 
} 
//Repeat the same steps for the next endpoint 
//Second endpoint https://adfs2.contoso.com ( Please modify) 
$certificates = Get-CertificateChainFromEndpoint https://adfs2.contoso.com //Please modify 

# Save certificates to specified folder 
$targetDirectory = 'C:\WinfieldExternalIdentityCertificates2' //Please modify 
foreach ($certificate in $certificates) 
{ 
    $bytes = $certificate.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Cert) 
    $bytes | Set-Content -Path "$targetDirectory\$($certificate.Thumbprint).cer" -Encoding Byte 
} 
 Prepping the parameters 

# Read certificate information for external identity configuration 
$targetDirectory = 'C:\WinfieldExternalIdentityCertificates2' 
$ ldapCertchainInfo = @() 
foreach ($file in (Get-ChildItem $targetDirectory)) 
{ 
    $certificate = [System.Security.Cryptography.X509Certificates.X509Certificate2]::new($file.FullName) 
    $bytes = $certificate.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Cert) 
    $b64 = [system.convert]::ToBase64String($bytes) 
    $ ldapCertchainInfo += $b64 
} 
```

## Related content

- [Plan hardware for Azure local with disconnected operations](disconnected-operations-overview.md#preview-participation-criteria).

- [Plan and understand identity](disconnected-operations-identity.md).

- [Plan and understand networking](disconnected-operations-network.md).

- [Set up disconnected operations](disconnected-operations-set-up.md).

::: moniker-end

::: moniker range="<=azloc-24111"

This feature is available only in Azure Local 2411.2.

::: moniker-end
