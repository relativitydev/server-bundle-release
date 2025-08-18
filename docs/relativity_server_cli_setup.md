# Set up Environment Watch and/or Data Grid Audit using the Relativity Server CLI

![Setup Stage](../resources/enable_environmentwatch.png)

Now that you have installed the required Elastic components for Environment Watch and/or Data Grid Audit, you will configure the integration between Elastic and Relativity by running the Relativity Server CLI on your Primary SQL Server. For Environment Watch, the Relativity Server CLI will also import all of the indexes, alerts, and dashboards that Relativity has developed and shipped as part of the Environment Watch product.

For customers already using Data Grid Audit prior to upgrading to Relativity Server 2024, please see the **_Important information for existing Data Grid Audit customers section_** below for additional context on what will happen when you run the ‘Set up Data Grid’ workflow using the Relativity Server CLI for the first time.

### Before you start

Before entering the Environment Watch or Data Grid Audit setup workflows, perform or check the following:

1. Confirm that all Elastic components are installed and verified from stages 1 and 2 of this installation guide. This includes ensuring that the minimum versions of Elasticsearch, Kibana, and APM Server that are specified in the Environment Watch release bundle that you are using are installed and that Elastic certificates have been installed on all Elastic hosts.<br/>

   **Note:**  Ensuring that you are on the minimum supported version of Elasticsearch for Data Grid Audit as specified in the release bundle is especially important for existing Data Grid Audit customers that may be running legacy versions of Elasticsearch. If you are an existing Data Grid Audit user, you must be on Elasticsearch 7.17 when you initially run the Data Grid Audit setup using the Relativity Server CLI. After you successfully configure Data Grid Audit using the Relativity Server CLI, you can then upgrade to Elasticsearch 8.17 in any cluster being used for Data Grid Audit.<br/>

    While 7.17 is the minimum supported version for the initial release of the Relativity Server CLI in Server 2024 Patch 1, you should always check the minimum version requirements in the specific Environment Watch release bundle that you are using.
   

2. Be prepared to enter admin usernames, passwords, and URLs for Relativity and Elastic components.<br/>
3. \[Data Grid Audit only\] Install the mapper-size plugin on all nodes in your Elasticsearch cluster (instructions available [here](https://www.elastic.co/guide/en/elasticsearch/plugins/current/mapper-size.html)). You also must restart the Elasticsearch service after installing the plugin.<br/>
4. \[Data Grid Audit only\] Before upgrading to Elasticsearch 8.17, the ESIndexCreationSetting may need to be updated. For further details, please refer to the [Instance settings' descriptions - Server2024](https://help.relativity.com/Server2024/Content/System_Guides/Instance_Setting_Guide/Instance_setting_descriptions.htm#ESIndexCreationSettings).<br/>
5. At least the minimum Relativity major version and patch specified in the Environment Watch bundle you intend to deploy is installed on all servers in the environment. See the [release bundle](https://github.com/relativitydev/server-bundle-release/releases) requirements for the minimum version required.<br/>
6. Verify that the InfraWatch Services application is installed in your Relativity instance (this RAP is delivered as part of the base Relativity Server 2024 installation package).<br/>
7. Follow [these instructions](https://help.relativity.com/Server2024/Content/System_Guides/Secret_Store/Secret_Store.htm#Configuringclients) to whitelist all hosts with Elastic installed for Secret Store access.<br/>
8. Ensure that you have access to Relativity, as well as the Primary and Distributed SQL Servers<br/>
9. The user must have Command Prompt installed to run the CLI executable.<br/>

### Set up Environment Watch

Follow these steps to configure Environment Watch using the Relativity Server CLI. If you do not need to set up Environment Watch, proceed to the ‘Set up Data Grid Audit’ section below.<br/>

This only needs to be done on your SQL Primary Server.<br/>

1. Install Elastic certificates on your SQL Primary Server<br/>
2. Download the CLI from [here](https://github.com/relativitydev/server-bundle-release/releases), download the release bundle.<br/>
3. Open Command Terminal by launching Command Prompt v7 from the Start menu.<br/>
4. Extract the CLI by navigating to the directory where the gold release bundle was downloaded to, extract the bundle and then extract Relativity.Server.Cli.YY.x.xxxx.zip.<br/>
5. Run the Setup Command from Command Prompt, execute the following command to enter the setup workflow:<br/>
    ```
    ./relsvr.exe setup
    ```
6. Select Environment Watch
7. \[Only applicable if Environment Watch has been set up previously\] Choose setup type – If Environment Watch has been set up on this host previously, you will be prompted to select "Upgrade" or "Rerun Setup". If you are setting up Environment Watch for the first time, you will not be prompted to make this selection and the setup process will continue to the next step.<br/>
8. Provide Relativity parameters - Enter the Relativity admin username and password and Relativity URL.<br/>
9. Provide Elasticsearch parameters - Enter the Elasticsearch admin username and password and Elasticsearch cluster endpoint URL (any node in your cluster will work, but we recommend providing the node URL for the master node where you first installed Elasticsearch in step 1 of this installation guide).<br/>
10. Provide APM Server parameters - Enter your Elasticsearch APM Server Endpoint URL.<br/>
11. Provide Kibana parameters – Enter your Elasticsearch Kibana server Endpoint URL.<br/>
12. Verify the generated API keys – To verify that the API keys used for authenticating Elastic to Relativity were generated:<br/>
	a. Open Kibana
	b. Navigate to the path: /app/management/security/api_keys
	c. Verify that API keys for rel-alerts and rel-infrawatch are present. 
	**Note:** These API keys will need to be refreshed every six months.<br/>
13. Install Elastic certificates on all Web Servers in your environment. Restart services on each host after installing the certificates<br/>
14. Install Elastic certificates on all Agent Servers in your environment. Restart services on each host after installing the certificates<br/>

If the setup completes successfully, Environment Watch is now configured for your environment. If you encountered any errors while entering Relativity or Elastic parameters, you will have three retry attempts before the CLI forces an exit and you must restart the setup process.

The following toggles are configured during the Environment Watch setup, which includes enabling application-level instrumentation and other related features:
1. Primary Environment Watch Toggle
	- Relativity.EnvironmentWatch.Toggles.EnableEnvironmentWatchToggle
2. OTEL (OpenTelemetry) Toggles
	- Relativity.EnvironmentWatch.Toggles.EnableOtelProvidersForKeplerServicesToggle
	- Relativity.EnvironmentWatch.Toggles.EnableOtelProvidersForAgentsToggle
	- Relativity.EnvironmentWatch.Toggles.EnableOtelProvidersForCustomPagesToggle
	- Relativity.EnvironmentWatch.Toggles.EnableOtelProvidersForServiceHostToggle
	- Relativity.EnvironmentWatch.Toggles.EnableOtelSqlClientDbStatementForTextToggle
3. Invariant Toggles
	- Invariant.EnvironmentWatch.Toggles.EnableEnvironmentWatchToggle
	- Invariant.EnvironmentWatch.Toggles.EnableOtelProvidersForQueueManager
	- Invariant.EnvironmentWatch.Toggles.EnableOtelProvidersForWorker
	- Invariant.EnvironmentWatch.Toggles.EnableOtelSqlClientDbStatementForTextToggle
4. Audit-Related Toggle
	- Relativity.Audit.Common.Toggles.ElasticAPIKeyAuthenticationToggle

Refer to the [Troubleshooting Guide](environment_watch_troubleshooting.md) if you encounter any issues.

### Set up Data Grid Audit

This section covers the steps for configuring the integration between Relativity and Elasticsearch for Data Grid Audit.

If you are setting up Data Grid Audit for the first time, you will also need to install the Audit application to workspaces and add the Audit agents. See [here](https://help.relativity.com/Server2024/Content/Relativity/Audit/Audit.htm#InstallingandconfiguringAudit) for more information about the Audit agents.

#### Important information for existing Data Grid Audit customers

If you have been using Data Grid Audit prior to upgrading to Relativity Server 2024 Patch 1 or later, running the Relativity Server CLI’s ‘Set up Data Grid’ workflow will update how your Elasticsearch cluster authenticates into Relativity. Prior to Server 2024 Patch 1, Data Grid Audit relied on a Elastic plug-in called Custom Realms for authentication. Starting with Server 2024 Patch 1, authentication no longer requires Custom Realms and is now based on an API key-based OAuth2 authentication method.

Once you have cut over to the new API key-based authentication using the Relativity Server CLI, you no longer need to use the Elastic Platinum license key that Relativity previously provided as part of our Elasticsearch installation package ("DataTron") to use Data Grid Audit. Moving forward, you will use the free/open Elastic license, or optionally apply a Platinum or Enterprise license key held by your organization if you are interested in utilizing Elasticsearch features that are not supported under the free/open license. No core Data Grid Audit functionality is impacted when you downgrade to the free/open license.

All Relativity Server customers that use Data Grid Audit will be required to upgrade to at least Server 2024 Patch 1, cut over to the new API key-based authentication using the Relativity Server CLI, and swap out the Relativity-provided license key for a free/open license or a Platinum/Enterprise license held by your organization by early 2026 in order for Data Grid Audit to continue working.

**Note:** Steps 12-15 below include important instructions for any existing Audit users that are setting up Data Grid using the Relativity Server CLI for the first time.

**Note:** If you are an existing Data Grid Audit user, you must be on Elasticsearch 7.17 when you initially run the Data Grid Audit setup using the Relativity Server CLI. After you successfully configure Data Grid Audit using the Relativity Server CLI, you can then upgrade to Elasticsearch 8.17 in any cluster being used for Data Grid Audit.

#### Set up instructions

Follow these steps to set up Data Grid Audit using the Relativity Server CLI.

**All setup will occur on the SQL Primary server.**

**Note:** If you have already run the ‘Set up Environment Watch’ workflow on this host, you do not need to repeat steps 1-2 below.

1. Install Elastic certificate on SQL Primary Server<br/>
2. Download the CLI – From [here](https://github.com/relativitydev/server-bundle-release/releases), download the release bundle.<br/>
3. Open Command Terminal - Launch Command Terminal v7 from the Start menu.<br/>
4. Extract the CLI - Navigate to the directory where the release bundle was downloaded to and extract the Relativity.Server.Cli.YY.x.xxxx.zip.<br/>
5. Run the Setup Command – From Command Prompt, execute the following command to enter the setup workflow:<br/>
    ```
    ./relsvr.exe setup
    ```
6. Select DataGrid  <br/>
7. \[Only applicable if Data Grid Audit has been set up previously using the Relativity Server CLI\] Choose setup type – If Data Grid Audit has been set up on this host previously, you will be prompted to select "Rerun Setup" or "Exit". If you are using the Relativity Server CLI to set up Data Grid Audit on this host for the first time (even if you began adopting Data Grid Audit before the Relativity Server CLI was initially released), you will not be prompted to make this selection and the setup process will continue to the next step.<br/>
8. Provide Relativity parameters – Enter the Relativity admin username and password and Relativity URL.<br/>
9. Provide Elasticsearch parameters - Enter the Elasticsearch admin username and password and Elasticsearch cluster endpoint URL (any node in your cluster will work, but we recommend providing the node URL for the master node where you first installed Elasticsearch in step 1 of this installation guide).<br/>
10. Verify the generated API keys – To verify that the API keys used for authenticating Elastic to Relativity were generated:<br/>
    a. Open Kibana
    b. Navigate to the path: /app/management/security/api_keys
    c. Verify that API keys for rel-datagrid is present. 
	**Note:** This API key will need to be refreshed every six months.
11. Restart Relativity services<br/>

**Note:** Steps 12-15 are only applicable if you are running the CLI setup workflow to cut over from the legacy custom realms-based authentication to API key-based authentication.<br/>

12.  Verify that API key authentication is being used – Execute the below query at the EDDS database level to verify that Elastic API key authentication is enabled.
       
        ```
        SELECT TOP(50) FROM [EDDSLogging][eddsdbo][RelativityLogs] where message like '%elastic api key authentication%' ORDER by 1 desc
        SELECT * FROM [EDDS].[eddsdbo].[Toggle] where name ='ElasticAPIKeyAuthenticationToggle'
        ```

13.  Verify that the Audit dashboard is functional<br/>
    a. Navigate to the Audit tab in your Relativity environment.
    b. Verify that the Audit dashboard graphs and data is loading.
14.  Update license key – After successfully running the Data Grid setup and verifying that API key authentication is now being used, you need to update your license to free/open or a different Platinum or Enterprise license that is not the one previously provided by Relativity.<br/>
    a. Open Kibana and navigate to Stack Management -> License management
    b. Update your license<br/>

**Note:** Steps 15-16 are only applicable if you have not already installed Elastic certificates on Web and Agent Servers during Environment Watch setup or a previous Data Grid Audit setup.<br/>

15. Install Elastic certificates on all Web Servers in your environment. Restart services on each host after installing the certificates.<br/>
16. Install Elastic certificates on all Agent Servers in your environment. Restart services on each host after installing the certificates.<br/>
**Note:** Step 17 is only applicable if you are doing a first-time setup of Audit in your Relativity instance.

17.  Install Audit application and agents - If you are setting up Data Grid Audit for the first time, you will also need to install the Audit application to workspaces and add the Audit agents. See [here](https://help.relativity.com/Server2024/Content/Relativity/Audit/Audit.htm#InstallingandconfiguringAudit) for more information about the Audit agents.

If the setup completes successfully, the integration between Elasticsearch and Relativity for Data Grid Audit is now configured for your environment. If you encountered any errors while entering Relativity or Elasticsearch parameters, you will have three retry attempts before the CLI forces an exit and you must restart the setup process.

Refer to the [Troubleshooting Guide](environment_watch_troubleshooting.md) if you encounter any issues.

### Elastic relativity_dashboard_user role

After you complete all five steps of this installation guide and have Environment Watch fully set up in your environment, you need to provide any user that should have access to the alerts and dashboards in Kibana with Elastic access credentials. You will do this by creating users in Kibana. When you create users in Kibana, you will need to assign them a ‘role’. When you run the Environment Watch or Data Grid setup workflow, the Relativity Server CLI will create a security role called ‘relativity_dashboard_user’. We recommend using this role for any users in your organization that should be able to access alerts and dashboards. The role has least privileges to ensure they can see all dashboards and alerts but with permissions restrictions to prevent them from creating or editing alerts, dashboards, and indexes.

You can see the privileges associated with the relativity_dashboard_user role by navigating to Stack Management > Roles > relativity_dashboard_user in Kibana.

**Note:** In order to extend the ability to export saved searches, go to the kibana.yml file on the server where Kibana is installed, update **xpack.reporting.roles.enabled** to "false", and then restart the Kibana service.

## Next

[Click here for the next step](install_environment_watch_monitoring_agents.md)