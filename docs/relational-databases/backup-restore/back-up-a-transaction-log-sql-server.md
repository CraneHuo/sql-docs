---
title: "Back up a transaction log"
description: This article describes how to back up a transaction log in SQL Server by using SQL Server Management Studio, Transact-SQL, or PowerShell.
author: MashaMSFT
ms.author: mathoma
ms.reviewer: randolphwest
ms.date: 11/21/2023
ms.service: sql
ms.subservice: backup-restore
ms.topic: conceptual
helpviewer_keywords:
  - "transaction log backups [SQL Server], SQL Server Management Studio"
  - "backups [SQL Server], creating"
  - "backing up transaction logs [SQL Server], SQL Server Management Studio"
---
# Back up a transaction log

[!INCLUDE [SQL Server](../../includes/applies-to-version/sqlserver.md)]

This article describes how to back up a transaction log in [!INCLUDE [ssnoversion](../../includes/ssnoversion-md.md)] by using [!INCLUDE [ssManStudioFull](../../includes/ssmanstudiofull-md.md)], [!INCLUDE [name-sos-short](../../includes/name-sos-short.md)], [!INCLUDE [tsql](../../includes/tsql-md.md)], or PowerShell.

## Limitations

The `BACKUP` statement isn't allowed in an explicit or [implicit](../../t-sql/statements/set-implicit-transactions-transact-sql.md) transaction. An explicit transaction is one in which you explicitly define both the start and end of the transaction.

Transaction log backups of the `master` system database aren't supported.

## Recommendations

If a database uses either the full or bulk-logged [recovery model](recovery-models-sql-server.md), you must back up the transaction log regularly enough to protect your data, and to prevent the [transaction log from filling](../logs/troubleshoot-a-full-transaction-log-sql-server-error-9002.md). This truncates the log and supports restoring the database to a specific point in time.

By default, every successful backup operation adds an entry in the [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] error log and in the system event log. If you back up the log frequently, these success messages accumulate quickly, resulting in huge error logs, making finding other messages difficult. In such cases you can suppress these log entries by using trace flag 3226, if none of your scripts depend on those entries, see [Trace Flags (Transact-SQL)](../../t-sql/database-console-commands/dbcc-traceon-trace-flags-transact-sql.md).

## Permissions

The `BACKUP DATABASE` and `BACKUP LOG` permissions needed are granted by default to members of the **sysadmin** fixed server role, and the **db_owner** and **db_backupoperator** fixed database roles. Check for the correct permissions before you begin.

Ownership and permission problems on the backup device's physical file can interfere with a backup operation. [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] must be able to read and write to the device; the account under which the [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] service runs must have write permissions. However, [sp_addumpdevice](../system-stored-procedures/sp-addumpdevice-transact-sql.md), which adds an entry for a backup device in the system tables, doesn't check file access permissions. Permissions problems on the backup device's physical file aren't obvious to you until you attempt to access the [physical resource](backup-devices-sql-server.md) when you try to back up or restore. So again, check permissions before you begin.

## Use SQL Server Management Studio

> [!NOTE]  
> The steps in this section also apply to [!INCLUDE [name-sos-short](../../includes/name-sos-short.md)].

1. After connecting to the appropriate instance of the [!INCLUDE [ssDEnoversion](../../includes/ssdenoversion-md.md)], in Object Explorer, select the server name to expand the server tree.

1. Expand **Databases**, and, depending on the database, either select a user database or expand **System Databases** and select a system database.

1. Right-click the database, point to **Tasks**, and then select **Back Up**. The **Back Up Database** dialog box appears.

1. In the **Database** list box, verify the database name. You can optionally select a different database from the list.

1. Verify that the recovery model is either **FULL** or **BULK_LOGGED**.

1. In the **Backup type** list box, select **Transaction Log**.

1. *(optional)* Select **Copy Only Backup** to create a copy-only backup. A *copy-only backup* is a [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] backup that is independent of the sequence of conventional [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] backups, see [Copy-only backups](copy-only-backups-sql-server.md).

   > [!NOTE]  
   > When the **Differential** option is selected, you can't create a copy-only backup.

1. Either accept the default backup set name suggested in the **Name** text box, or enter a different name for the backup set.

1. *(optional)* In the **Description** text box, enter a description of the backup set.

1. Specify when the backup set will expire:

   - To have the backup set expire after a specific number of days, select **After** (the default option), and enter the number of days after set creation that the set will expire. This value can be from 0 to 99999 days; a value of 0 days means that the backup set never expires.

     The default value is set in the **Default backup media retention (in days)** option of the **Server Properties** dialog box (**Database Settings** page). To access this dialog box, right-click the server name in Object Explorer and select properties; then select the **Database Settings** page.

   - To have the backup set expire on a specific date, select **On**, and enter the date on which the set will expire.

1. Choose the type of backup destination by selecting **Disk**, **URL**, or **Tape**. To select the paths of up to 64 disk or tape drives containing a single media set, select **Add**. The selected paths are displayed in the **Backup to** list box.

   To remove a backup destination, select it and select **Remove**. To view the contents of a backup destination, select it and select **Contents**.

1. To view or select the advanced options, select **Options** in the **Select a page** pane.

1. Select an **Overwrite Media** option, by selecting one of the following:

   - **Back up to the existing media set**

     For this option, select either **Append to the existing backup set** or **Overwrite all existing backup sets**, see [Media Sets, Media Families, and Backup Sets (SQL Server)](media-sets-media-families-and-backup-sets-sql-server.md).

     - *(optional)* Select **Check media set name and backup set expiration** to cause the backup operation to verify the date and time at which the media set and backup set expire.

     - *(optional)* Enter a name in the **Media set name** text box. If no name is specified, a media set with a blank name is created. If you specify a media set name, the media (tape or disk) is checked to see whether the actual name matches the name you enter here.

     If you leave the media name blank and check the box to check it against the media, success equals the media name on the media also being blank.

   - **Back up to a new media set, and erase all existing backup sets**

     For this option, enter a name in the **New media set name** text box, and, optionally, describe the media set in the **New media set description** text box, see [Media Sets, Media Families, and Backup Sets (SQL Server)](media-sets-media-families-and-backup-sets-sql-server.md).

1. In the **Reliability** section, optionally, check:

   - **Verify backup when finished**.

   - **Perform checksum before writing to media** and *(optional)* **Continue on checksum error**.

     For information on checksums, see [Possible Media Errors During Backup and Restore (SQL Server)](possible-media-errors-during-backup-and-restore-sql-server.md).

1. In the **Transaction log** section:

   - For routine log backups, keep the default selection, **Truncate the transaction log by removing inactive entries**.

   - To back up the tail of the log (the active log), check **Back up the tail of the log, and leave database in the restoring state**.

     A tail-log backup is taken after a failure to back up the tail of the log in order to prevent work loss. Back up the active log (a tail-log backup) both after a failure, before beginning to restore the database, or when failing over to a secondary database. Selecting this option is equivalent to specifying the NORECOVERY option in the BACKUP LOG statement of Transact-SQL.

     For more information about tail-log backups, see [Tail-log backups (SQL Server)](tail-log-backups-sql-server.md).

1. If you're backing up to a tape drive (as specified in the **Destination** section of the **General** page), the **Unload the tape after backup** option is active. Selecting this option activates the **Rewind the tape before unloading** option.

1. By default, whether a [backup is compression](backup-compression-sql-server.md) depends on the value of the **backup-compression default** server configuration option. However, regardless of the current server-level default, you can compress a backup by checking **Compress backup**, and you can prevent compression by checking **Don't compress backup**.

   [Backup compression](backup-compression-sql-server.md) is supported on [!INCLUDE [ssEnterpriseEd10](../../includes/ssenterpriseed10-md.md)] and later versions, and [!INCLUDE [sssql16-md](../../includes/sssql16-md.md)] Standard with Service Pack 1 and later versions.

   To view the current backup compression default, see [View or Configure the backup compression default (server configuration option)](../../database-engine/configure-windows/view-or-configure-the-backup-compression-default-server-configuration-option.md).

   To encrypt the backup file, check the **Encrypt backup** check box. Select an encryption algorithm to use for encrypting the backup file and provide a Certificate or Asymmetric key. The available algorithms for encryption are:

   - AES 128
   - AES 192
   - AES 256
   - Triple DES

## Use Transact-SQL

Execute the BACKUP LOG statement to back up the transaction log, providing the following information:

- The name of the database to which the transaction log that you want to back up belongs.
- The backup device where the transaction log backup is written.

> [!IMPORTANT]  
> This example uses the [!INCLUDE [ssSampleDBobject](../../includes/sssampledbobject-md.md)] database, which uses the simple recovery model. To permit log backups, before taking a full database backup, the database was set to use the full recovery model.
>  
> For more information, see [View or change the recovery model of a database (SQL Server)](view-or-change-the-recovery-model-of-a-database-sql-server.md).

This example creates a transaction log backup for the [!INCLUDE [ssSampleDBobject](../../includes/sssampledbobject-md.md)] database to the previously created named backup device, `MyAdvWorks_FullRM_log1`.

```sql
BACKUP LOG AdventureWorks2022
   TO MyAdvWorks_FullRM_log1;
GO
```

## <a id="PowerShellProcedure"></a> Use PowerShell

Set up and use the [SQL Server PowerShell Provider](../../powershell/sql-server-powershell-provider.md). Use the **Backup-SqlDatabase** cmdlet and specify **Log** for the value of the **-BackupAction** parameter.

The following example creates a log backup of the `<myDatabase>` database to the default backup location of the server instance `Computer\Instance`.

```powershell
Backup-SqlDatabase -ServerInstance Computer\Instance -Database <myDatabase> -BackupAction Log
```

## Related tasks

- [Restore a Transaction Log Backup (SQL Server)](restore-a-transaction-log-backup-sql-server.md)
- [Restore a SQL Server Database to a Point in Time (Full Recovery Model)](restore-a-sql-server-database-to-a-point-in-time-full-recovery-model.md)
- [Troubleshoot a full transaction log (SQL Server Error 9002)](../logs/troubleshoot-a-full-transaction-log-sql-server-error-9002.md)

## Related content

- [BACKUP (Transact-SQL)](../../t-sql/statements/backup-transact-sql.md)
- [Apply Transaction Log Backups (SQL Server)](apply-transaction-log-backups-sql-server.md)
- [Maintenance plans](../maintenance-plans/maintenance-plans.md)
- [Full File Backups (SQL Server)](full-file-backups-sql-server.md)
