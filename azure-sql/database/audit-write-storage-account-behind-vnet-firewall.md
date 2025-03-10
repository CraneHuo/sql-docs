---
title: Audit to storage account behind VNet and firewall
description: Configure auditing to write database events on a storage account behind virtual network and firewall
author: sravanisaluru
ms.author: srsaluru
ms.reviewer: wiassaf, vanto, mathoma
ms.date: "03/23/2022"
ms.service: sql-database
ms.subservice: security
ms.topic: how-to
ms.custom: azure-synapse, subject-rbac-steps, devx-track-arm-template, devx-track-azurepowershell
---
# Write audit to a storage account behind VNet and firewall
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]


Auditing for [Azure SQL Database](sql-database-paas-overview.md) and [Azure Synapse Analytics](/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is) supports writing database events to an [Azure Storage account](/azure/storage/common/storage-account-overview) behind a virtual network and firewall.

This article explains two ways to configure Azure SQL Database and Azure storage account for this option. The first uses the Azure portal, the second uses REST.

## Background

[Azure Virtual Network (VNet)](/azure/virtual-network/virtual-networks-overview) is the fundamental building block for your private network in Azure. VNet enables many types of Azure resources, such as Azure Virtual Machines (VM), to securely communicate with each other, the internet, and on-premises networks. VNet is similar to a traditional network in your own data center, but brings with it additional benefits of Azure infrastructure such as scale, availability, and isolation.

To learn more about the VNet concepts, Best practices and many more, see [What is Azure Virtual Network](/azure/virtual-network/virtual-networks-overview).

To learn more about how to create a virtual network, see [Quickstart: Create a virtual network using the Azure portal](/azure/virtual-network/quick-create-portal).

## Prerequisites

For audit to write to a storage account behind a VNet or firewall, the following prerequisites are required:

> [!div class="checklist"]
>
> * A general-purpose v2 storage account. If you have a general-purpose v1 or blob storage account, [upgrade to a general-purpose v2 storage account](/azure/storage/common/storage-account-upgrade). For more information, see [Types of storage accounts](/azure/storage/common/storage-account-overview#types-of-storage-accounts).
> * The premium storage with BlockBlobStorage is supported
> * The storage account must be on the same tenant and at the same location as the [logical SQL server](logical-servers.md) (it's OK to be on different subscriptions).
> * The Azure Storage account requires `Allow trusted Microsoft services to access this storage account`. Set this on the Storage Account **Firewalls and Virtual networks**.
> * You must have `Microsoft.Authorization/roleAssignments/write` permission on the selected storage account. For more information, see [Azure built-in roles](/azure/role-based-access-control/built-in-roles).

> [!NOTE]
> When Auditing to storage account is already enabled on a server / db, and if the target storage account is moved behind a firewall, we lose write access to 
the storage account and audit logs stop getting written to it.To make auditing work we have to resave the audit settings from portal. 
   

## Configure in Azure portal

Connect to [Azure portal](https://portal.azure.com) with your subscription. Navigate to the resource group and server.

1. Click on **Auditing** under the Security heading. Select **On**.

2. Select **Storage**. Select the storage account where logs will be saved. The storage account must comply with the requirements listed in [Prerequisites](#prerequisites).

3. Open **Storage details**

  > [!NOTE]
  > If the selected Storage account is behind VNet, you will see the following message:
  >
  >`You have selected a storage account that is behind a firewall or in a virtual network. Using this storage requires to enable 'Allow trusted Microsoft services to access this storage account' on the storage account and creates a server managed identity with 'storage blob data contributor' RBAC.`
  >
  >If you do not see this message, then storage account is not behind a VNet.

4. Select the number of days for the retention period. Then click **OK**. Logs older than the retention period are deleted.

5. Select **Save** on your auditing settings.

You have successfully configured audit to write to a storage account behind a VNet or firewall.

## Configure with REST commands

As an alternative to using the Azure portal, you can use REST commands to configure audit to write database events on a storage account behind a VNet and Firewall.

The sample scripts in this section require you to update the script before you run them. Replace the following values in the scripts:

|Sample value|Sample description|
|:-----|:-----|
|`<subscriptionId>`| Azure subscription ID|
|`<resource group>`| Resource group|
|`<logical SQL Server>`| Server name|
|`<administrator login>`| Administrator account |
|`<complex password>`| Complex password for the administrator account|

To configure SQL Audit to write events to a storage account behind a VNet or Firewall:

1. Register your server with Microsoft Entra ID ([formerly Azure Active Directory](/azure/active-directory/fundamentals/new-name)). Use either PowerShell or REST API.

   **PowerShell**

   ```powershell
   Connect-AzAccount
   Select-AzSubscription -SubscriptionId <subscriptionId>
   Set-AzSqlServer -ResourceGroupName <your resource group> -ServerName <azure server name> -AssignIdentity
   ```

   [**REST API**](/rest/api/sql/servers/createorupdate):

   Sample request

   ```html
   PUT https://management.azure.com/subscriptions/<subscription ID>/resourceGroups/<resource group>/providers/Microsoft.Sql/servers/<azure server name>?api-version=2015-05-01-preview
   ```

   Request body

   ```json
   {
   "identity": {
              "type": "SystemAssigned",
              },
   "properties": {
     "fullyQualifiedDomainName": "<azure server name>.database.windows.net",
     "administratorLogin": "<administrator login>",
     "administratorLoginPassword": "<complex password>",
     "version": "12.0",
     "state": "Ready"
     }
   }
   ```

1. Assign the Storage Blob Data Contributor role to the server hosting the database that you registered with Microsoft Entra ID in the previous step.

    For detailed steps, see [Assign Azure roles using the Azure portal](/azure/role-based-access-control/role-assignments-portal).

   > [!NOTE]
   > Only members with Owner privilege can perform this step. For various Azure built-in roles, refer to [Azure built-in roles](/azure/role-based-access-control/built-in-roles).

1. Configure the [server's blob auditing policy](/rest/api/sql/server%20auditing%20settings/createorupdate), without specifying a *storageAccountAccessKey*:

   Sample request

   ```html
     PUT https://management.azure.com/subscriptions/<subscription ID>/resourceGroups/<resource group>/providers/Microsoft.Sql/servers/<azure server name>/auditingSettings/default?api-version=2017-03-01-preview
   ```

   Request body

   ```json
   {
     "properties": {
      "state": "Enabled",
      "storageEndpoint": "https://<storage account>.blob.core.windows.net"
     }
   }
   ```

## Using Azure PowerShell

- [Create or Update Database Auditing Policy (Set-AzSqlDatabaseAudit)](/powershell/module/az.sql/set-azsqldatabaseaudit)
- [Create or Update Server Auditing Policy (Set-AzSqlServerAudit)](/powershell/module/az.sql/set-azsqlserveraudit)

## Using Azure Resource Manager template

You can configure auditing to write database events on a storage account behind virtual network and firewall using [Azure Resource Manager](/azure/azure-resource-manager/management/overview) template, as shown in the following example:

> [!IMPORTANT]
> In order to use storage account behind virtual network and firewall, you need to set **isStorageBehindVnet** parameter to true

- [Deploy an Azure SQL Server with Auditing enabled to write audit logs to a blob storage](https://azure.microsoft.com/resources/templates/sql-auditing-server-policy-to-blob-storage/)

> [!NOTE]
> The linked sample is on an external public repository and is provided 'as is', without warranty, and are not supported under any Microsoft support program/service.

## Next steps

* [Use PowerShell to create a virtual network service endpoint, and then a virtual network rule for Azure SQL Database.](scripts/vnet-service-endpoint-rule-powershell-create.md)
* [Virtual Network Rules: Operations with REST APIs](/rest/api/sql/virtualnetworkrules)
* [Use virtual network service endpoints and rules for servers](vnet-service-endpoint-rule-overview.md)
