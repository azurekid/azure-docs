---
title: Back up an app
description: Learn how to create backups of your apps in Azure App Service. Run manual or scheduled backups. Customize backups by including the attached database.
ms.assetid: 6223b6bd-84ec-48df-943f-461d84605694
ms.topic: article
ms.date: 09/02/2021
ms.custom: seodec18

---

# Back up your app in Azure

The Backup and Restore feature in [Azure App Service](overview.md) lets you easily
create app backups manually or on a schedule. You can configure the backups to be retained up to an indefinite amount of time. You can restore the app to a snapshot of a previous state by overwriting the existing app or restoring to another app.

For information on restoring an app from backup, see [Restore an app in Azure](web-sites-restore.md).

<a name="whatsbackedup"></a>

## What gets backed up

App Service can back up the following information to an Azure storage account and container that you have configured your app to use.

* App configuration
* File content
* Database connected to your app

The following database solutions are supported with backup feature:

- [SQL Database](https://azure.microsoft.com/services/sql-database/)
- [Azure Database for MySQL](https://azure.microsoft.com/services/mysql)
- [Azure Database for PostgreSQL](https://azure.microsoft.com/services/postgresql)
- [MySQL in-app](https://azure.microsoft.com/blog/mysql-in-app-preview-app-service/)

> [!NOTE]
> Each backup is a complete offline copy of your app, not an incremental update.
>

<a name="requirements"></a>

## Requirements and restrictions

* The Backup and Restore feature requires the App Service plan to be in the **Standard**, **Premium**, or **Isolated** tier. For more information about scaling your App Service plan to use a higher tier, see [Scale up an app in Azure](manage-scale-up.md). **Premium** and **Isolated** tiers allow a greater number of daily back ups than **Standard** tier.
* You need an Azure storage account and container in the same subscription as the app that you want to back up. For more information on Azure storage accounts, see [Azure storage account overview](../storage/common/storage-account-overview.md).
* Backups can be up to 10 GB of app and database content, up to 4GB of which can be the database backup. If the backup size exceeds this limit, you get an error.
* Backups of [TLS enabled Azure Database for MySQL](../mysql/concepts-ssl-connection-security.md) is not supported. If a backup is configured, you will encounter backup failures.
* Backups of [TLS enabled Azure Database for PostgreSQL](../postgresql/concepts-ssl-connection-security.md) is not supported. If a backup is configured, you will encounter backup failures.
* In-app MySQL databases are automatically backed up without any configuration. If you make manual settings for in-app MySQL databases, such as adding connection strings, the backups may not work correctly.
* Using a [firewall enabled storage account](../storage/common/storage-network-security.md) as the destination for your backups is not supported. If a backup is configured, you will encounter backup failures.
* Using a [private endpoint enabled storage account](../storage/common/storage-private-endpoints.md) for backup and restore is not supported.

<a name="manualbackup"></a>

## Create a manual backup

1. In the [Azure portal](https://portal.azure.com), navigate to your app's page, select **Backups**. The **Backups** page is displayed.

    ![Backups page](./media/manage-backup/access-backup-page.png)

    > [!NOTE]
    > If you see the following message, click it to upgrade your App Service plan before you can proceed with backups.
    > For more information, see [Scale up an app in Azure](manage-scale-up.md).
    > :::image type="content" source="./media/manage-backup/upgrade-plan.png" alt-text="Screenshot of a banner with a message to upgrade the App Service plan to access the Backup and Restore feature.":::
    >
    >

2. In the **Backup** page, select **Backup is not configured. Click here to configure backup for your app**.

    ![Click Configure](./media/manage-backup/configure-start.png)

3. In the **Backup Configuration** page, click **Storage not configured** to configure a storage account.

    :::image type="content" source="./media/manage-backup/configure-storage.png" alt-text="Screenshot of the Backup Storage section with the Storage not configured setting selected.":::

4. Choose your backup destination by selecting a **Storage Account** and **Container**. The storage account must belong to the same subscription as the app you want to back up. If you wish, you can create a new storage account or a new container in the respective pages. When you're done, click **Select**.

5. In the **Backup Configuration** page that is still left open, you can configure **Backup Database**, then select the databases you want to include in the backups (SQL Database or MySQL), then click **OK**.

    :::image type="content" source="./media/manage-backup/configure-database.png" alt-text="Screenshot of Backup Database section showing the Include in backup selection.":::

    > [!NOTE]
    > For a database to appear in this list, its connection string must exist in the **Connection strings** section of the **Application settings** page for your app. 
    >
    > In-app MySQL databases are automatically backed up without any configuration. If you make settings for in-app MySQL databases manually, such as adding connection strings, the backups may not work correctly.
    >
    >

6. In the **Backup Configuration** page, click **Save**.
7. In the **Backups** page, click **Backup**.

    ![BackUpNow button](./media/manage-backup/manual-backup.png)

    You see a progress message during the backup process.

Once the storage account and container is configured, you can initiate a manual backup at any time. Manual backups are retained indefinitely.

<a name="automatedbackups"></a>

## Configure automated backups

1. In the **Backup Configuration** page, set **Scheduled backup** to **On**.

    ![Enable automated backups](./media/manage-backup/scheduled-backup.png)

2. Configure the backup schedule as desired and select **OK**.

<a name="partialbackups"></a>

## Configure Partial Backups

Sometimes you don't want to back up everything on your app. Here are a few examples:

* You [set up weekly backups](#configure-automated-backups) of your app that contains static content that never changes, such as old blog posts or images.
* Your app has over 10 GB of content (that's the max amount you can back up at a time).
* You don't want to back up the log files.

Partial backups allow you choose exactly which files you want to back up.

> [!NOTE]
> Individual databases in the backup can be 4GB max but the total max size of the backup is 10GB

### Exclude files from your backup

Suppose you have an app that contains log files and static images that have been backup once and are not going to change. In such cases, you can exclude those folders and files from being stored in your future backups. To exclude files and folders from your backups, create a `_backup.filter` file in the `D:\home\site\wwwroot` folder of your app. Specify the list of files and folders you want to exclude in this file.

You can access your files by navigating to `https://<app-name>.scm.azurewebsites.net/DebugConsole`. If prompted, sign in to your Azure account.

Identify the folders that you want to exclude from your backups. For example, you want to filter out the highlighted folder and files.

![Images Folder](./media/manage-backup/kudu-images.png)

Create a file called `_backup.filter` and put the preceding list in the file, but remove `D:\home`. List one directory or file per line. So the content of the file should be:

```
\site\wwwroot\Images\brand.png
\site\wwwroot\Images\2014
\site\wwwroot\Images\2013
```

Upload `_backup.filter` file to the `D:\home\site\wwwroot\` directory of your site using [ftp](deploy-ftp.md) or any other method. If you wish, you can create the file directly using Kudu `DebugConsole` and insert the content there.

Run backups the same way you would normally do it, [manually](#create-a-manual-backup) or [automatically](#configure-automated-backups). Now, any files and folders that are specified in `_backup.filter` is excluded from the future backups scheduled or manually initiated.

> [!NOTE]
> You restore partial backups of your site the same way you would [restore a regular backup](web-sites-restore.md). The restore process does the right thing.
>
> When a full backup is restored, all content on the site is replaced with whatever is in the backup. If a file is on the site, but not in the backup it gets deleted. But when a partial backup is restored, any content that is located in one of the restricted directories, or any restricted file, is left as is.
>

<a name="aboutbackups"></a>

## How backups are stored

After you have made one or more backups for your app, the backups are visible on the **Containers** page of your storage account, and your app. In the storage account, each backup consists of a`.zip` file that contains the backup data and an `.xml` file that contains a manifest of the `.zip` file contents. You can unzip and browse these files if you want to access your backups without actually performing an app restore.

The database backup for the app is stored in the root of the .zip file. For SQL Database, this is a BACPAC file (no file extension) and can be imported. To create a database in Azure SQL Database based on the BACPAC export, see [Import a BACPAC file to create a database in Azure SQL Database](../azure-sql/database/database-import.md).

> [!WARNING]
> Altering any of the files in your **websitebackups** container can cause the backup to become invalid and therefore non-restorable.

## Troubleshooting

The **Backups** page shows you the status of each backup. If you click on a failed backup, you can get log details regarding the failure. Use the following table to help you troubleshoot your backup. If the failure isn't documented in the table, open a support ticket.

| Error | Fix |
| - | - |
| Storage access failed. | Delete backup schedule and reconfigure it. Or, reconfigure the backup storage. |
| The website + database size exceeds the {0} GB limit for backups. Your content size is {1} GB. | [Exclude some files](#configure-partial-backups) from the backup, or remove the database portion of the backup and use externally offered backups instead. |
| Error occurred while connecting to the database {0} on server {1}: Authentication to host '{1}' for user '\<username>' using method 'mysql_native_password' failed with message: Unknown database '\<db-name>' | Update database connection string. |
| Cannot resolve {0}. {1} (CannotResolveStorageAccount) | Delete the backup schedule and reconfigure it. |
| Login failed for user '{0}'. | Update the database connection string. |
| Create Database copy of {0} ({1}) threw an exception. Could not create Database copy. | Use an administrative user in the connection string. |
| The server principal "\<name>" is not able to access the database "master" under the current security context. Cannot open database "master" requested by the login. The login failed. Login failed for user '\<name>'. | Use an administrative user in the connection string. |
| A network-related or instance-specific error occurred while establishing a connection to SQL Server. The server was not found or was not accessible. Verify that the instance name is correct and that SQL Server is configured to allow remote connections. (provider: Named Pipes Provider, error: 40 - Could not open a connection to SQL Server). | Check that the connection string is valid. Allow the app's [outbound IPs](overview-inbound-outbound-ips.md) in the database server settings. |
| Cannot open server "\<name>" requested by the login. The login failed. | Check that the connection string is valid. |
| Missing mandatory parameters for valid Shared Access Signature. | Delete the backup schedule and reconfigure it. |
| SSL connection is required. Please specify SSL options and retry. when trying to connect. | Use the built-in backup feature in Azure MySQL or Azure Postgressql instead. |

## Automate with scripts

You can automate backup management with scripts, using the [Azure CLI](/cli/azure/install-azure-cli) or [Azure PowerShell](/powershell/azure/).

For samples, see:

- [Azure CLI samples](samples-cli.md)
- [Azure PowerShell samples](samples-powershell.md)

<a name="nextsteps"></a>

## Next Steps

For information on restoring an app from a backup, see [Restore an app in Azure](web-sites-restore.md).
