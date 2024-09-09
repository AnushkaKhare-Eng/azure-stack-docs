---
title: App Service on Azure Stack Hub 2024 R1 release notes 
description: Learn about what's new in the 2024 R1 release for App Service on Azure Stack Hub, the known issues, and where to download the update.
author: anwestg
ms.topic: article
ms.date: 12/06/2024
ms.author: anwestg
ms.reviewer:

---

# App Service on Azure Stack Hub 2024 R1 release notes

These release notes describe the improvements and fixes in Azure App Service on Azure Stack Hub 2024 R1 release notes and any known issues. Known issues are divided into issues directly related to the deployment, update process, and issues with the build (post-installation).

[!INCLUDE [Azure Stack Hub update reminder](../includes/app-service-hub-update-banner.md)]

## Build reference

The App Service on Azure Stack Hub 2024 R1 build number is **102.0.x.x**

## What's new?

Azure App Service on Azure Stack Hub 2024 R1 brings new updates to Azure Stack Hub.

- All roles are now powered by Windows Server 2022 Datacenter.
- As of the Azure App Service on Azure Stack Hub 2022 H1 update, the letter **K** is now a reserved SKU letter. If you have a custom SKU defined that uses the letter K, contact support to assist with resolving this situation prior to upgrade.

## Prerequisites

Refer to the [Before You Get Started documentation](azure-stack-app-service-before-you-get-started.md) before beginning deployment.

Before you begin the upgrade of Azure App Service on Azure Stack to 2024 R1:

- Ensure your **Azure Stack Hub** is updated to **1.2406.0.8** or **1.2311.1.22**.

- Ensure all roles are Ready in the Azure App Service Administration in the Azure Stack Hub Admin Portal.

- Backup App Service Secrets using the App Service Administration in the Azure Stack Hub Admin Portal.

- Back up the App Service and SQL Server Master Databases:
  - AppService_Hosting;
  - AppService_Metering;
  - Master

- Back up the Tenant App content file share.

  > [!Important]
  > Cloud operators are responsible for the maintenance and operation of the File Server and SQL Server.  The resource provider does not manage these resources.  The cloud operator is responsible for backing up the App Service databases and tenant content file share.

- Syndicate the **Custom Script Extension** version **1.9.3** from the Marketplace.

## Updates

Azure App Service on Azure Stack Update 2024 R1 includes the following improvements and fixes:

- Updates to **App Service Tenant, Admin, Functions portals and Kudu tools**. Consistent with Azure Stack Portal SDK version.

- Updates **Azure Functions runtime** to **v1.0.21719**.

- Updates to core service to improve reliability and error messaging enabling easier diagnosis of common issues.

- **Updates to the following application frameworks and tools**:
  - 2024-09 Cumulative Update for .NET Framework 3.5 and 4.8 for Microsoft server operating system version 21H2 for x64 (KBxxxxx).
  - ASP.NET Core 
    - 8.0.7
    - 6.0.24
  - Eclipse Temurin OpenJDK 8
    - 8u332
    - 8u345
    - 8u362
    - 8u392
  - Microsoft OpenJDK 11
    - 11.0.15.10
    - 11.0.16.1
    - 11.0.18.10
    - 11.0.21.9
  - Microsoft OpenJDK 17
    - 17.0.3.7
    - 17.0.4.1
    - 17.0.6.10
    - 17.0.9.8
  - Microsoft OpenJDK 21
    - 21.0.1.12
  - MSBuild 
    - 16.11.2
    - 17.9.0
  - MSDeploy 3.5.120717.134
  - NodeJS
    - 14.20.0
    - 16.16.0
    - 18.12.1
    - 18.19.1
    - 20.9.0
  - npm
    - 6.14.16
    - 7.14.17
    - 8.19.2
    - 10.1.0
    - 10.2.4
  - Tomcat
    - 8.5.79
    - 8.5.82
    - 8.5.85
    - 9.0.63
    - 9.0.65
    - 9.0.71
    - 9.0.83
    - 10.0.21
    - 10.0.23
    - 10.0.27
    - 10.1.5
    - 10.1.16
  - Updated Kudu to 102.10502.001.


- **Updates to underlying operating system of all roles**:
  - TBD
  - Defender Definition TBD

- **Cumulative Updates for Windows Server are now applied to Controller roles as part of deployment and upgrade**.

## Issues fixed in this release

- 

## Pre-Update steps

- As of Azure App Service on Azure Stack Hub 2022 H1 Update, the letter K is now a reserved SKU Letter, if you have a custom SKU defined utilizing the letter K, contact support to assist resolving this situation prior to upgrade.

Review the [known issues for update](#known-issues-update) and take any action prescribed.

## Post-deployment steps

> [!IMPORTANT]
> If you have provided the App Service resource provider with a SQL Always On Instance you MUST [add the appservice_hosting and appservice_metering databases to an availability group](/sql/database-engine/availability-groups/windows/availability-group-add-a-database) and synchronize the databases to prevent any loss of service in the event of a database failover.

## Known issues (update)

- In situations where a customer has converted the appservice_hosting and appservice_metering databases to contained database, upgrade may fail if logins haven't been successfully migrated to contained users.

Customers that have converted the appservice_hosting and appservice_metering databases to contained database post deployment, and haven't successfully migrated the database logins to contained users, may experience upgrade failures.  

Customers must execute the following script against the SQL Server hosting appservice_hosting and appservice_metering before upgrading your Azure App Service on Azure Stack Hub installation to 2020 Q3.  **This script is non-destructive and will not cause downtime**.

This script must be run under the following conditions:

- By a user that has the system administrator privilege, for example the SQL SA Account;
- If using SQL Always on, ensure the script is run from the SQL instance that contains all App Service logins in the form:

    - appservice_hosting_FileServer
    - appservice_hosting_HostingAdmin
    - appservice_hosting_LoadBalancer
    - appservice_hosting_Operations
    - appservice_hosting_Publisher
    - appservice_hosting_SecurePublisher
    - appservice_hosting_WebWorkerManager
    - appservice_metering_Common
    - appservice_metering_Operations
    - All WebWorker logins - which are in the form WebWorker_\<instance ip address\>

```sql
        USE appservice_hosting
        IF EXISTS(SELECT * FROM sys.databases WHERE Name=DB_NAME() AND containment = 1)
        BEGIN
        DECLARE @username sysname ;  
        DECLARE user_cursor CURSOR  
        FOR
            SELECT dp.name
            FROM sys.database_principals AS dp  
            JOIN sys.server_principals AS sp
                ON dp.sid = sp.sid  
                WHERE dp.authentication_type = 1 AND dp.name NOT IN ('dbo','sys','guest','INFORMATION_SCHEMA');
            OPEN user_cursor  
            FETCH NEXT FROM user_cursor INTO @username  
                WHILE @@FETCH_STATUS = 0  
                BEGIN  
                    EXECUTE sp_migrate_user_to_contained
                    @username = @username,  
                    @rename = N'copy_login_name',  
                    @disablelogin = N'do_not_disable_login';  
                FETCH NEXT FROM user_cursor INTO @username  
            END  
            CLOSE user_cursor ;  
            DEALLOCATE user_cursor ;
            END
        GO

        USE appservice_metering
        IF EXISTS(SELECT * FROM sys.databases WHERE Name=DB_NAME() AND containment = 1)
        BEGIN
        DECLARE @username sysname ;  
        DECLARE user_cursor CURSOR  
        FOR
            SELECT dp.name
            FROM sys.database_principals AS dp  
            JOIN sys.server_principals AS sp
                ON dp.sid = sp.sid  
                WHERE dp.authentication_type = 1 AND dp.name NOT IN ('dbo','sys','guest','INFORMATION_SCHEMA');
            OPEN user_cursor  
            FETCH NEXT FROM user_cursor INTO @username  
                WHILE @@FETCH_STATUS = 0  
                BEGIN  
                    EXECUTE sp_migrate_user_to_contained
                    @username = @username,  
                    @rename = N'copy_login_name',  
                    @disablelogin = N'do_not_disable_login';  
                FETCH NEXT FROM user_cursor INTO @username  
            END  
            CLOSE user_cursor ;  
            DEALLOCATE user_cursor ;
            END
        GO
```

- Tenant Applications are unable to bind certificates to applications after upgrade.

  The cause of this issue is due to a missing feature on Front-Ends after upgrade to Windows Server 2022.  Operators must follow this procedure to resolve the issue.

  1. In the Azure Stack Hub admin portal, navigate to **Network Security Groups** and view the **ControllersNSG** Network Security Group.

  1. By default, remote desktop is disabled to all App Service infrastructure roles. Modify the **Inbound_Rdp_2289** rule action to **Allow** access.

  1. Navigate to the resource group containing the App Service Resource Provider deployment, by default the name is **AppService.\<region\>** and connect to **CN0-VM**.
  1. Return to the **CN0-VM** remote desktop session.
  1. In an Administrator PowerShell session run:
      
      > [!IMPORTANT] 
      > During the execution of this script there will be a pause for each instance in the Front End scaleset.  If there is a message indicating the feature is being installed,
      > that instance will be rebooted, use the pause in the script to maintain Front End availability.  Operators must ensure at least one Front End instance is "Ready" at all times
      > to ensure tenant applications can receive traffic and not experience downtime.

      ```powershell
      $c = Get-AppServiceConfig -Type Credential -CredentialName FrontEndCredential
      $spwd = ConvertTo-SecureString -String $c.Password -AsPlainText -Force
      $cred = New-Object System.Management.Automation.PsCredential ($c.UserName, $spwd)

      Get-AppServiceServer -ServerType LoadBalancer | ForEach-Object {
          $lb = $_
          $session = New-PSSession -ComputerName $lb.Name -Credential $cred

          Invoke-Command -Session $session {
            $f = Get-WindowsFeature -Name Web-CertProvider
            if (-not $f.Installed) {
                Write-Host Install feature on $env:COMPUTERNAME
                Install-WindowsFeature -Name Web-CertProvider

                Read-Host -Prompt "If installing the feature, the machine will reboot. Wait until there's enough frontend availability, then press ENTER to continue"
                Shutdown /t 5 /r /f 
            }
      }

      Remove-PSSession -Session $session     
      ```

  1. In the Azure Stack admin portal, navigate back to the **ControllersNSG** Network Security Group.

  1. Modify the **Inbound_Rdp_3389** rule to deny access. 

## Known issues (post-installation)

- Workers are unable to reach file server when App Service is deployed in an existing virtual network and the file server is only available on the private network,  as called out in the Azure App Service on Azure Stack deployment documentation.

  If you chose to deploy into an existing virtual network and an internal IP address to connect to your file server, you must add an outbound security rule, enabling SMB traffic between the worker subnet and the file server. Go to the WorkersNsg in the Admin Portal and add an outbound security rule with the following properties:
  - Source: Any
  - Source port range: *
  - Destination: IP Addresses
  - Destination IP address range: Range of IPs for your file server
  - Destination port range: 445
  - Protocol: TCP
  - Action: Allow
  - Priority: 700
  - Name: Outbound_Allow_SMB445

- To remove latency when workers are communicating with the file server we also advise adding the following rule to the Worker NSG to allow outbound LDAP and Kerberos traffic to your Active Directory Controllers if securing the file server using Active Directory, for example if you've used the Quickstart template to deploy a HA File Server and SQL Server.

  Go to the WorkersNsg in the Admin Portal and add an outbound security rule with the following properties:
  - Source: Any
  - Source port range: *
  - Destination: IP Addresses
  - Destination IP address range: Range of IPs for your AD Servers, for example with the Quickstart template 10.0.0.100, 10.0.0.101
  - Destination port range: 389,88
  - Protocol: Any
  - Action: Allow
  - Priority: 710
  - Name: Outbound_Allow_LDAP_and_Kerberos_to_Domain_Controllers

- Tenant Applications are unable to bind certificates to applications after upgrade.

  The cause of this issue is due to a missing feature on Front-Ends after upgrade to Windows Server 2022.  Operators must follow this procedure to resolve the issue.

  1. In the Azure Stack Hub admin portal, navigate to **Network Security Groups** and view the **ControllersNSG** Network Security Group.

  1. By default, remote desktop is disabled to all App Service infrastructure roles. Modify the **Inbound_Rdp_2289** rule action to **Allow** access.

  1. Navigate to the resource group containing the App Service Resource Provider deployment, by default the name is **AppService.\<region\>** and connect to **CN0-VM**.
  1. Return to the **CN0-VM** remote desktop session.
  1. In an Administrator PowerShell session run:
      
      > [!IMPORTANT] 
      > During the execution of this script there will be a pause for each instance in the Front End scaleset.  If there is a message indicating the feature is being installed,
      > that instance will be rebooted, use the pause in the script to maintain Front End availability.  Operators must ensure at least one Front End instance is "Ready" at all times
      > to ensure tenant applications can receive traffic and not experience downtime.

      ```powershell
      $c = Get-AppServiceConfig -Type Credential -CredentialName FrontEndCredential
      $spwd = ConvertTo-SecureString -String $c.Password -AsPlainText -Force
      $cred = New-Object System.Management.Automation.PsCredential ($c.UserName, $spwd)

      Get-AppServiceServer -ServerType LoadBalancer | ForEach-Object {
          $lb = $_
          $session = New-PSSession -ComputerName $lb.Name -Credential $cred

          Invoke-Command -Session $session {
            $f = Get-WindowsFeature -Name Web-CertProvider
            if (-not $f.Installed) {
                Write-Host Install feature on $env:COMPUTERNAME
                Install-WindowsFeature -Name Web-CertProvider

                Shutdown /t 5 /r /f 
            }
      }

      Remove-PSSession -Session $session

      Read-Host -Prompt "If installing the feature, the machine will reboot. Wait until there's enough frontend availability, then press ENTER to continue"
      ```

  1. In the Azure Stack admin portal, navigate back to the **ControllersNSG** Network Security Group.

  1. Modify the **Inbound_Rdp_3389** rule to deny access. 

### Known issues for Cloud Admins operating Azure App Service on Azure Stack

- Custom domains aren't supported in disconnected environments.

  App Service performs domain ownership verification against public DNS endpoints. As a result, custom domains aren't supported in disconnected scenarios.

- Virtual Network integration for Web and Function Apps is not supported.

  The ability to add virtual network integration to Web and Function apps shows in the Azure Stack Hub portal and if a tenant attempts to configure, they receive an internal server error. This feature is not supported in Azure App Service on Azure Stack Hub.

## Next steps

- For an overview of Azure App Service, see [Azure App Service on Azure Stack overview](azure-stack-app-service-overview.md).
- For more information about how to prepare to deploy App Service on Azure Stack, see [Before you get started with App Service on Azure Stack](azure-stack-app-service-before-you-get-started.md).
