# Enable Data Grid Audit

![Setup Stage](../resources/enable_environmentwatch.png)

> [!NOTE]
> This section applies to Datagrid Only.

After installing the required Elastic components for Data Grid Audit, the integration between Elastic and Relativity is configured by running the Relativity Server CLI on the Primary SQL Server.

> [!NOTE]
> Please review the following important information before proceeding:
> * **For Existing Data Grid Audit Customers:** You must be on Elasticsearch 7.17 when initially running this setup. After the setup is complete, you can upgrade to Elasticsearch 8.17.3.
> * Before upgrading to Elasticsearch 8.17.3, the `ESIndexCreationSetting` may need to be updated. For details, refer to the [Instance setting Details](https://help.relativity.com/Server2024/Content/System_Guides/Instance_Setting_Guide/Instance_setting_descriptions.htm#ESIndexCreationSettings).
> * Always verify the minimum required Elasticsearch version in your specific release bundle, as it may differ from the versions mentioned here.

### Prerequisites 


1. Install the mapper-size plugin on all nodes in the Elasticsearch cluster (instructions available [here](https://www.elastic.co/guide/en/elasticsearch/plugins/current/mapper-size.html)). The Elasticsearch service must also be restarted after installing the plugin.

2. The Server-bundle zip file has been downloaded and extracted to `C:\Server.Bundle.x.y.z'
   
3. Verify that the InfraWatch Services application is installed in the Relativity instance (this RAP is delivered as part of the base Relativity Server 2024 installation package).



#### Set up instructions

Follow these steps to set up Data Grid Audit using the Relativity Server CLI. All setup will occur on the SQL Primary server.

**Step 1** - Open elevated command prompt/powershell. Navigate to the directory where the CLI is extracted, and run ".\relsvr.exe setup". Select Datagrid

```
C:\Server.Bundle.x.y.z> .\relsvr.exe setup

Relativity Server CLI - 24.0.1196
Copyright (c) 2025, Relativity ODA LLC

What would you like to setup?
> DataGrid
  Environment watch
  Exit
```

**Step 2** - Enter the required Relativity and Elasticsearch parameters.

```
Confirm you would like to perform the 'DataGrid' setup [y/n] (y): y

Existing settings do not exist
Enter the Relativity admin username (relativity.admin@kcura.com): relativity.admin@kcura.com
Enter the Relativity admin password: *********
Enter the Relativity instance url (https://emttest/Relativity): https://emttest/Relativity
Relativity instance is verified
Enter the Elasticsearch admin username (elastic): elastic
Enter the Elasticsearch admin password: *********
Enter the Elasticsearch cluster endpoint URL (https://emttest:9200): https://emttest:9200

```

**Step 3** - Wait for Setup to Complete.

```
Elasticsearch cluster endpoint URL is verified
Elasticsearch plugin verified

API Key creation and validation completed ------------------------- 100%
Relativity instance setting validation completed ------------------ 100%
Relativity secret store updated ----------------------------------- 100%
Elastic Stack settings validation completed ----------------------- 100%
Relativity toggles validation completed --------------------------- 100%

The Relativity Data Grid setup has been completed. Please restart all Relativity services, including "kCura Edds Agent Manager," "kCura Edds Web Processing Manager," and "kCura Service Host Manager" on each server contained within this Relativity instance to complete the setup.
```
If the setup completes successfully, Datagrid is now configured for the environment.

**Step 4** - Restart the Relativity services on all machines for the changes to take effect.


**Step 5** - Verify Audit Dashboard
Navigate to the Audit tab in the Relativity environment and confirm that the dashboard and its data are loading correctly.


## Next

[Click here for the next step](install_environment_watch_monitoring_agents.md)