---
title: "sp_addpullsubscription_agent (Transact-SQL)"
description: Adds a new scheduled agent job used to synchronize a pull subscription to a transactional publication.
author: markingmyname
ms.author: maghan
ms.reviewer: randolphwest
ms.date: 11/02/2023
ms.service: sql
ms.subservice: replication
ms.topic: "reference"
f1_keywords:
  - "sp_addpullsubscription_agent"
  - "sp_addpullsubscription_agent_TSQL"
helpviewer_keywords:
  - "sp_addpullsubscription_agent"
dev_langs:
  - "TSQL"
---
# sp_addpullsubscription_agent (Transact-SQL)

[!INCLUDE [SQL Server SQL MI](../../includes/applies-to-version/sql-asdbmi.md)]

Adds a new scheduled agent job used to synchronize a pull subscription to a transactional publication. This stored procedure is executed at the Subscriber on the subscription database.

:::image type="icon" source="../../includes/media/topic-link-icon.svg" border="false"::: [Transact-SQL syntax conventions](../../t-sql/language-elements/transact-sql-syntax-conventions-transact-sql.md)

## Syntax

```syntaxsql
sp_addpullsubscription_agent
    [ @publisher = ] N'publisher'
    [ , [ @publisher_db = ] N'publisher_db' ]
    , [ @publication = ] N'publication'
    [ , [ @subscriber = ] N'subscriber' ]
    [ , [ @subscriber_db = ] N'subscriber_db' ]
    [ , [ @subscriber_security_mode = ] subscriber_security_mode ]
    [ , [ @subscriber_login = ] N'subscriber_login' ]
    [ , [ @subscriber_password = ] N'subscriber_password' ]
    [ , [ @distributor = ] N'distributor' ]
    [ , [ @distribution_db = ] N'distribution_db' ]
    [ , [ @distributor_security_mode = ] distributor_security_mode ]
    [ , [ @distributor_login = ] N'distributor_login' ]
    [ , [ @distributor_password = ] N'distributor_password' ]
    [ , [ @optional_command_line = ] N'optional_command_line' ]
    [ , [ @frequency_type = ] frequency_type ]
    [ , [ @frequency_interval = ] frequency_interval ]
    [ , [ @frequency_relative_interval = ] frequency_relative_interval ]
    [ , [ @frequency_recurrence_factor = ] frequency_recurrence_factor ]
    [ , [ @frequency_subday = ] frequency_subday ]
    [ , [ @frequency_subday_interval = ] frequency_subday_interval ]
    [ , [ @active_start_time_of_day = ] active_start_time_of_day ]
    [ , [ @active_end_time_of_day = ] active_end_time_of_day ]
    [ , [ @active_start_date = ] active_start_date ]
    [ , [ @active_end_date = ] active_end_date ]
    [ , [ @distribution_jobid = ] distribution_jobid OUTPUT ]
    [ , [ @encrypted_distributor_password = ] encrypted_distributor_password ]
    [ , [ @enabled_for_syncmgr = ] N'enabled_for_syncmgr' ]
    [ , [ @ftp_address = ] N'ftp_address' ]
    [ , [ @ftp_port = ] ftp_port ]
    [ , [ @ftp_login = ] N'ftp_login' ]
    [ , [ @ftp_password = ] N'ftp_password' ]
    [ , [ @alt_snapshot_folder = ] N'alt_snapshot_folder' ]
    [ , [ @working_directory = ] N'working_directory' ]
    [ , [ @use_ftp = ] N'use_ftp' ]
    [ , [ @publication_type = ] publication_type ]
    [ , [ @dts_package_name = ] N'dts_package_name' ]
    [ , [ @dts_package_password = ] N'dts_package_password' ]
    [ , [ @dts_package_location = ] N'dts_package_location' ]
    [ , [ @reserved = ] N'reserved' ]
    [ , [ @offloadagent = ] N'offloadagent' ]
    [ , [ @offloadserver = ] N'offloadserver' ]
    [ , [ @job_name = ] N'job_name' ]
    [ , [ @job_login = ] N'job_login' ]
    [ , [ @job_password = ] N'job_password' ]
[ ; ]
```

## Arguments

#### [ @publisher = ] N'*publisher*'

The name of the Publisher. *@publisher* is **sysname**, with no default.

<!--SQL Server 2019 on Linux-->
::: moniker range=">= sql-server-linux-ver15 || >= sql-server-ver15"
[!INCLUDE [custom-port](includes/custom-port.md)]
::: moniker-end

#### [ @publisher_db = ] N'*publisher_db*'

The name of the Publisher database. *@publisher_db* is **sysname**, with a default of `NULL`. *@publisher_db* is ignored by Oracle Publishers.

#### [ @publication = ] N'*publication*'

The name of the publication. *@publication* is **sysname**, with no default.

#### [ @subscriber = ] N'*subscriber*'

The name of the Subscriber instance or the name of the AG listener if the subscriber database is in an availability group.
*@subscriber* is **sysname**, with a default of `NULL`.

> [!NOTE]  
> [!INCLUDE [deprecated-parameter](../includes/deprecated-parameter.md)]

When running `sp_addpullsubscription_agent` for a subscriber that is part of an AG, set *@subscriber* to the AG listener name. If you run [!INCLUDE [sssql16-md](../../includes/sssql16-md.md)] and earlier versions, or [!INCLUDE [sssql17](../../includes/sssql17-md.md)] before CU 16, the stored procedure executes without returning an error, but the *@subscriber* parameter on the [Replication Distribution Agent](../replication/agents/replication-distribution-agent.md) doesn't reference the AG listener name; the parameter is created with the subscriber server name on which the command is executed. To amend this issue, manually update the [Distribution Agent job](../replication/agents/replication-distribution-agent.md#arguments) *@subscriber* parameter with the AG listener name value.

#### [ @subscriber_db = ] N'*subscriber_db*'

The name of the subscription database. *@subscriber_db* is **sysname**, with a default of `NULL`.

> [!NOTE]  
> [!INCLUDE [deprecated-parameter](../includes/deprecated-parameter.md)]

#### [ @subscriber_security_mode = ] *subscriber_security_mode*

The security mode to use when connecting to a Subscriber when synchronizing. *@subscriber_security_mode* is **int**, with a default of `NULL`. `0` specifies [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] Authentication. `1` specifies Windows Authentication.

> [!NOTE]  
> [!INCLUDE [deprecated-parameter](../includes/deprecated-parameter.md)] The Distribution Agent always connects to the local Subscriber using Windows Authentication. If a value other than `NULL` or `1` is specified for this parameter, a warning message is returned.

#### [ @subscriber_login = ] N'*subscriber_login*'

The Subscriber login to use when connecting to a Subscriber when synchronizing. *@subscriber_login* is **sysname**, with a default of `NULL`.

> [!NOTE]  
> [!INCLUDE [deprecated-parameter-returns-warning](../includes/deprecated-parameter-returns-warning.md)]

#### [ @subscriber_password = ] N'*subscriber_password*'

The Subscriber password. *subscriber_password* is required if *subscriber_security_mode* is set to `0`. *@subscriber_password* is **sysname**, with a default of `NULL`. If a subscriber password is used, it's automatically encrypted.

> [!NOTE]  
> [!INCLUDE [deprecated-parameter-returns-warning](../includes/deprecated-parameter-returns-warning.md)]

#### [ @distributor = ] N'*distributor*'

The name of the Distributor. *@distributor* is **sysname**, with a default of the value specified by *@publisher*.

#### [ @distribution_db = ] N'*distribution_db*'

The name of the distribution database. *@distribution_db* is **sysname**, with a default of `NULL`.

#### [ @distributor_security_mode = ] *distributor_security_mode*

The security mode to use when connecting to a Distributor when synchronizing. *@distributor_security_mode* is **int**, with a default of `1`. The following values define the security mode:

- `0` specifies [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] Authentication.
- `1` specifies Windows Authentication.
- `2` specifies Microsoft Entra password authentication starting with [!INCLUDE [sssql22-md](../../includes/sssql22-md.md)] CU 6.
- `3` specifies Microsoft Entra integrated authentication starting with [!INCLUDE [sssql22-md](../../includes/sssql22-md.md)] CU 6.
- `4` specifies Microsoft Entra token authentication starting with [!INCLUDE [sssql22-md](../../includes/sssql22-md.md)] CU 6.

> [!IMPORTANT]  
> [!INCLUDE [ssNoteWinAuthentication](../../includes/ssnotewinauthentication-md.md)]

#### [ @distributor_login = ] N'*distributor_login*'

The Distributor login to use when connecting to a Distributor when synchronizing. *@distributor_login* is **sysname**, with a default of `NULL`. *@distributor_login* is required if *@distributor_security_mode* is set to `0`.

#### [ @distributor_password = ] N'*distributor_password*'

The Distributor password. *distributor_password* is required if *distributor_security_mode* is set to `0`. *@distributor_password* is **sysname**, with a default of `NULL`.

> [!IMPORTANT]  
> [!INCLUDE [ssnotestrongpass-md](../../includes/ssnotestrongpass-md.md)] When possible, prompt users to enter security credentials at runtime. If you must store credentials in a script file, you must secure the file to prevent unauthorized access.

#### [ @optional_command_line = ] N'*optional_command_line*'

An optional command prompt supplied to the Distribution Agent. For example, `-DefinitionFile C:\Distdef.txt` or `-CommitBatchSize 10`. *@optional_command_line* is **nvarchar(4000)**, with a default of an empty string.

#### [ @frequency_type = ] *frequency_type*

The frequency with which to schedule the Distribution Agent. *@frequency_type* is **int**, and can be one of the following values.

| Value | Description |
| --- | --- |
| `1` | One time |
| `2` (default) | On demand |
| `4` | Daily |
| `8` | Weekly |
| `16` | Monthly |
| `32` | Monthly relative |
| `64` | Autostart |
| `128` | Recurring |

> [!NOTE]  
> Specifying a value of `64` causes the Distribution Agent to run in continuous mode. This corresponds to setting the `-Continuous` parameter for the agent. For more information, see [Replication Distribution Agent](../replication/agents/replication-distribution-agent.md).

#### [ @frequency_interval = ] *frequency_interval*

The value to apply to the frequency set by *@frequency_type*. *@frequency_interval* is **int**, with a default of `1`.

#### [ @frequency_relative_interval = ] *frequency_relative_interval*

The date of the Distribution Agent. This parameter is used when *@frequency_type* is set to `32` (monthly relative). *@frequency_relative_interval* is **int**, and can be one of the following values.

| Value | Description |
| --- | --- |
| `1` (default) | First |
| `2` | Second |
| `4` | Third |
| `8` | Fourth |
| `16` | Last |

#### [ @frequency_recurrence_factor = ] *frequency_recurrence_factor*

The recurrence factor used by *@frequency_type*. *@frequency_recurrence_factor* is **int**, with a default of `1`.

#### [ @frequency_subday = ] *frequency_subday*

Specifies how often to reschedule during the defined period.
*@frequency_subday* is **int**, and can be one of the following values.

| Value | Description |
| --- | --- |
| `1` (default) | Once |
| `2` | Second |
| `4` | Minute |
| `8` | Hour |

#### [ @frequency_subday_interval = ] *frequency_subday_interval*

The interval for *@frequency_subday*. *@frequency_subday_interval* is **int**, with a default of `1`.

#### [ @active_start_time_of_day = ] *active_start_time_of_day*

The time of day when the Distribution Agent is first scheduled, formatted as `HHmmss`. *@active_start_time_of_day* is **int**, with a default of `0`.

#### [ @active_end_time_of_day = ] *active_end_time_of_day*

The time of day when the Distribution Agent stops being scheduled, formatted as `HHmmss`. *@active_end_time_of_day* is **int**, with a default of `0`.

#### [ @active_start_date = ] *active_start_date*

The date when the Distribution Agent is first scheduled, formatted as `yyyyMMdd`. *@active_start_date* is **int**, with a default of `0`.

#### [ @active_end_date = ] *active_end_date*

The date when the Distribution Agent stops being scheduled, formatted as `yyyyMMdd`. *@active_end_date* is **int**, with a default of `0`.

#### [ @distribution_jobid = ] *distribution_jobid* OUTPUT

The ID of the Distribution Agent for this job. *@distribution_jobid* is an OUTPUT parameter of type **binary(16)**, with a default of `NULL`.

#### [ @encrypted_distributor_password = ] *encrypted_distributor_password*

*@encrypted_distributor_password* is **bit**, with a default of `0`.

> [!NOTE]  
> Setting *@encrypted_distributor_password* is no longer supported. Attempting to set this **bit** parameter to `1` will result in an error.

#### [ @enabled_for_syncmgr = ] N'*enabled_for_syncmgr*'

Specifies whether the subscription can be synchronized through [!INCLUDE [msCoName](../../includes/msconame-md.md)] Synchronization Manager. *@enabled_for_syncmgr* is **nvarchar(5)**, with a default of `false`.

- If `false`, the subscription isn't registered with Synchronization Manager.
- If `true`, the subscription is registered with Synchronization Manager and can be synchronized without starting [!INCLUDE [ssManStudioFull](../../includes/ssmanstudiofull-md.md)].

#### [ @ftp_address = ] N'*ftp_address*'

[!INCLUDE [deprecated-parameter](../includes/deprecated-parameter.md)]

#### [ @ftp_port = ] *ftp_port*

[!INCLUDE [deprecated-parameter](../includes/deprecated-parameter.md)]

#### [ @ftp_login = ] N'*ftp_login*'

[!INCLUDE [deprecated-parameter](../includes/deprecated-parameter.md)]

#### [ @ftp_password = ] N'*ftp_password*'

[!INCLUDE [deprecated-parameter](../includes/deprecated-parameter.md)]

#### [ @alt_snapshot_folder = ] N'*alt_snapshot_folder*'

Specifies the location of the alternate folder for the snapshot. *@alt_snapshot_folder* is **nvarchar(255)**, with a default of `NULL`.

#### [ @working_directory = ] N'*working_directory*'

The name of the working directory used to store data and schema files for the publication. *@working_directory* is **nvarchar(255)**, with a default of `NULL`. The name should be specified in UNC format.

#### [ @use_ftp = ] N'*use_ftp*'

Specifies the use of FTP instead of the regular protocol to retrieve snapshots. *@use_ftp* is **nvarchar(5)**, with a default of `false`.

#### [ @publication_type = ] *publication_type*

Specifies the replication type of the publication. *@publication_type* is **tinyint**, with a default of `0`.

- If `0`, publication is a transaction type.
- If `1`, publication is a snapshot type.
- If `2`, publication is a merge type.

#### [ @dts_package_name = ] N'*dts_package_name*'

Specifies the name of the DTS package. *@dts_package_name* is **sysname**, with a default of `NULL`. For example, to specify a package of `DTSPub_Package`, the parameter would be `@dts_package_name = N'DTSPub_Package'`.

#### [ @dts_package_password = ] N'*dts_package_password*'

Specifies the password on the package, if there is one. *@dts_package_password* is **sysname**, with a default of `NULL`, which means a password isn't on the package.

> [!NOTE]  
> You must specify a password if *@dts_package_name* is specified.

#### [ @dts_package_location = ] N'*dts_package_location*'

Specifies the package location. *@dts_package_location* is **nvarchar(12)**, with a default of `subscriber`. The location of the package can be `distributor` or `subscriber`.

#### [ @reserved = ] N'*reserved*'

[!INCLUDE [ssinternalonly-md](../../includes/ssinternalonly-md.md)]

#### [ @offloadagent = ] N'*offloadagent*'

[!INCLUDE [deprecated-parameter-returns-warning](../includes/deprecated-parameter-returns-warning.md)] Setting *@offloadagent* to a value other than `false` generates an error.

#### [ @offloadserver = ] N'*offloadserver*'

[!INCLUDE [deprecated-parameter-returns-warning](../includes/deprecated-parameter-returns-warning.md)] Setting *@offloadserver* to a value other than `false` generates an error.

#### [ @job_name = ] N'*job_name*'

The name of an existing agent job. *@job_name* is **sysname**, with a default of `NULL`. This parameter is only specified when the subscription is synchronized using an existing job instead of a newly created job (the default). If you aren't a member of the **sysadmin** fixed server role, you must specify *@job_login* and *@job_password* when you specify *@job_name*.

#### [ @job_login = ] N'*job_login*'

The login for the Windows account under which the agent runs. *@job_login* is **nvarchar(257)**, with no default. This Windows account is always used for agent connections to the Subscriber.

#### [ @job_password = ] N'*job_password*'

The password for the Windows account under which the agent runs. *@job_password* is **sysname**, with no default.

> [!IMPORTANT]  
> [!INCLUDE [ssnotestrongpass-md](../../includes/ssnotestrongpass-md.md)] When possible, prompt users to enter security credentials at runtime. If you must store credentials in a script file, you must secure the file to prevent unauthorized access.

## Return code values

`0` (success) or `1` (failure).

## Remarks

`sp_addpullsubscription_agent` is used in snapshot replication and transactional replication.

## Examples

:::code language="sql" source="../replication/codesnippet/tsql/sp-addpullsubscription-a_1.sql":::

## Permissions

Only members of the **sysadmin** fixed server role or **db_owner** fixed database role can execute `sp_addpullsubscription_agent`.

## Related content

- [Create a Pull Subscription](../replication/create-a-pull-subscription.md)
- [Subscribe to Publications](../replication/subscribe-to-publications.md)
- [sp_addpullsubscription (Transact-SQL)](sp-addpullsubscription-transact-sql.md)
- [sp_change_subscription_properties (Transact-SQL)](sp-change-subscription-properties-transact-sql.md)
- [sp_droppullsubscription (Transact-SQL)](sp-droppullsubscription-transact-sql.md)
- [sp_helppullsubscription (Transact-SQL)](sp-helppullsubscription-transact-sql.md)
- [sp_helpsubscription_properties (Transact-SQL)](sp-helpsubscription-properties-transact-sql.md)
