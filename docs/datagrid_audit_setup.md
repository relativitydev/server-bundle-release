# Enable Data Grid Audit

![Setup Stage](../resources/enable_environmentwatch.png)

After installing the required Elastic components for Data Grid Audit, the integration between Elastic and Relativity is configured by running the Relativity Server CLI on the Primary SQL Server.

For customers already using Data Grid Audit prior to upgrading to Relativity Server 2024, refer to the **_Important information for existing Data Grid Audit customers section_** for additional context on what happens when running the ‘Set up Data Grid’ workflow using the Relativity Server CLI for the first time.

### Prerequisites

> [!NOTE]
> Ensuring the minimum supported version of Elasticsearch for Data Grid Audit is installed, as specified in the release bundle, is especially important for existing Data Grid Audit customers that may be running legacy versions of Elasticsearch. Existing Data Grid Audit users must be on Elasticsearch 7.17 when initially running the Data Grid Audit setup using the Relativity Server CLI. After successfully configuring Data Grid Audit using the Relativity Server CLI, an upgrade to Elasticsearch 8.17 in any cluster being used for Data Grid Audit can be performed.

    While 7.17 is the minimum supported version for the initial release of the Relativity Server CLI in Server 2024 Patch 1, it is recommended to always check the minimum version requirements in the specific Environment Watch release bundle being used.

1. [Data Grid Audit only] Install the mapper-size plugin on all nodes in the Elasticsearch cluster (instructions available [here](https://www.elastic.co/guide/en/elasticsearch/plugins/current/mapper-size.html)). The Elasticsearch service must also be restarted after installing the plugin.

2. [Data Grid Audit only] Before upgrading to Elasticsearch 8.17.3, the ESIndexCreationSetting may need to be updated. For further details, refer to the [Instance settings' descriptions - Server2024](https://help.relativity.com/Server2024/Content/System_Guides/Instance_Setting_Guide/Instance_setting_descriptions.htm#ESIndexCreationSettings).

3. The Server-bundle zip file has been downloaded and extracted to `C:\Server.Bundle.x.y.z'
   
4. Verify that the InfraWatch Services application is installed in the Relativity instance (this RAP is delivered as part of the base Relativity Server 2024 installation package).

### Set up Data Grid Audit

This section covers the steps for configuring the integration between Relativity and Elasticsearch for Data Grid Audit.

For a first-time setup of Data Grid Audit, the Audit application will also need to be installed to workspaces and the Audit agents added. See [here](https://help.relativity.com/Server2024/Content/Relativity/Audit/Audit.htm#InstallingandconfiguringAudit) for more information about the Audit agents.

### Important information for existing Data Grid Audit customers
> [!NOTE]
> Existing Data Grid Audit users must be on Elasticsearch 7.17 when initially running the Data Grid Audit setup using the Relativity Server CLI. After successfully configuring Data Grid Audit using the Relativity Server CLI, an upgrade to Elasticsearch 8.17.3 in any cluster being used for Data Grid Audit can be performed.

#### Set up instructions

Follow these steps to set up Data Grid Audit using the Relativity Server CLI. All setup will occur on the SQL Primary server.

**Step 1** - Run the Setup Command
Execute the following command in an elevated Command/Powershell to enter the setup workflow.
```powershell
./relsvr.exe setup
```

**Step 2** - Select Data Grid

```
What would you like to setup?
> DataGrid
  Environment watch
  Exit
```

**Step 3**- Choose Setup Type (if applicable)
If Data Grid Audit has been set up on this host previously using the CLI, a prompt will appear to select **Rerun Setup**. For a first-time setup, this step will be skipped.
```
You have already set up Data Grid on this machine. Would you like to rerun setup?
> Rerun Setup
  Exit
```

**Step 4** - Enter Relativity and Elasticsearch Parameters

```
Enter the Relativity admin username (relativity.admin@kcura.com): relativity.admin@kcura.com
Enter the Relativity admin password: *********
Enter the Relativity instance url (https://emttest/Relativity): https://emttest/Relativity
Relativity instance is verified
Enter the Elasticsearch admin username (elastic): elastic
Enter the Elasticsearch admin password: *********
Enter the Elasticsearch cluster endpoint URL (https://emttest:9200): https://emttest:9200

```

**Step 6** - Verify API Key Generation
The CLI will confirm that the API keys have been created.
```
API Key creation and validation completed ------------------------- 100%
OAuth2 client exists -------------------------------------------- 100%
```
To manually verify, navigate to `/app/management/security/api_keys` in Kibana and confirm that an API key for `rel-datagrid` is present. This key must be refreshed every six months.

**Step 7** - Restart Relativity Services
Restart the Relativity services on all machines for the changes to take effect.

> [!NOTE]
> The following steps are only for users cutting over from the legacy custom realms authentication.

**Step 8** - Verify API Key Authentication
Execute the following queries against the EDDS database to verify that API key authentication is enabled.
```sql
SELECT TOP(50) FROM [EDDSLogging].[eddsdbo].[RelativityLogs] WHERE message LIKE '%elastic api key authentication%' ORDER BY 1 DESC
SELECT * FROM [EDDS].[eddsdbo].[Toggle] WHERE name = 'ElasticAPIKeyAuthenticationToggle'
```

**Step 9** - Verify Audit Dashboard
Navigate to the Audit tab in the Relativity environment and confirm that the dashboard and its data are loading correctly.

**Step 10** - Update License Key
In Kibana, navigate to **Stack Management > License Management** and update the license to the free/open tier or a different Platinum/Enterprise license not provided by Relativity.

**Step 11** - Install Certificates (if needed)
If not already done, install the Elastic certificates on all Web and Agent Servers, then restart their services.

> [!NOTE]
> The final step is only for users performing a first-time setup of Audit.

**Step 12** - Install Audit Application and Agents
Install the Audit application into workspaces and add the required Audit agents. For more information, see the [Audit documentation](https://help.relativity.com/Server2024/Content/Relativity/Audit/Audit.htm#InstallingandconfiguringAudit).

If the setup completes successfully, the integration is configured. If any errors are encountered, there will be three retry attempts before the CLI exits.

## Next

[Click here for the next step](install_environment_watch_monitoring_agents.md)