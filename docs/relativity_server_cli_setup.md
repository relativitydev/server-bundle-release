# Enable Environment Watch

![Setup Stage](../resources/enable_environmentwatch.png)

> [!NOTE]
> This section applies to Environment Watch Only.

# Set up Environment Watch using the Relativity Server CLI


### Prerequisites

1. Elastic Stack Certificates are installed on all Servers
2. Access to Elastic Stack, Primary SQL Server, and Secret Store (Whitelisted for Secret Store access. Please see [here](https://help.relativity.com/Server2024/Content/System_Guides/Secret_Store/Secret_Store.htm#Configuringclients) for information on whitelisting.)
3. The Server-bundle zip file has been downloaded and extracted to `C:\Server.Bundle.x.y.z'

### Set up instructions

**Step 1** - Run the `setup` command from an elevated Command/Powershell to enter the setup workflow. Select **Environment Watch** when prompted.

```powershell
C:\Server.Bundle.x.y.z> ./relsvr.exe setup

Relativity Server CLI - 24.0.1196
Copyright (c) 2025, Relativity ODA LLC

What would you like to setup?
  DataGrid
> Environment watch
  Exit
```

**Step 2** - Enter the required Relativity parameters.

```
Confirm you would like to perform the 'Environment Watch' setup [y/n] (y): y

Retrieved existing settings
Enter the Relativity admin username (relativity.admin@kcura.com): relativity.admin@kcura.com
Enter the Relativity admin password: *********
Enter the Relativity instance url (https://emttest/Relativity): https://emttest/Relativity
Relativity instance is verified
```

**Step 3** - Enter the required Elastic Stack parameters.

```
Enter the Elasticsearch admin username (elastic): elastic
Enter the Elasticsearch admin password: *********
Enter the Elasticsearch cluster endpoint URL (https://emttest:9200): https://emttest:9200
Elasticsearch cluster endpoint URL is verified
Enter the Elasticsearch APM Server endpoint URL (http://emttest:8200): http://emttest:8200
Enter the Elasticsearch Kibana endpoint URL (http://emttest:5601): http://emttest:5601
```

**Step 4** - Wait for Setup to Complete
The CLI will complete the setup process.

```
Elasticsearch Kibana URL is verified
API Key creation and validation completed ------------------------- 100%
OAuth2 client exists ---------------------------------------------- 100%
Relativity secret store updated ----------------------------------- 100%
Relativity logging updated ---------------------------------------- 100%
Relativity toggles updated ---------------------------------------- 100%
Relativity AppDomain monitoring enabled --------------------------- 100%
APM settings updated ---------------------------------------------- 100%
Elasticsearch indexes updated ------------------------------------- 100%
Kibana Tag imported ----------------------------------------------- 100%
Kibana IndexPattern imported -------------------------------------- 100%
Kibana SavedSearch imported --------------------------------------- 100%
Kibana Dashboard imported ----------------------------------------- 100%
Kibana Alert imported --------------------------------------------- 100%
Kibana Custom Role created. --------------------------------------- 100%
Relativity installation SQL record updated ------------------------ 100%

The Environment Watch setup has been completed. The Relativity Environment Watch installer package should now be installed within each server contained within this Relativity instance. As each server is setup for monitoring, restart all Relativity services within the machine including "kCura Edds Agent Manager," "kCura Edds Web Processing Manager," and "kCura Service Host Manager" to begin sending telemetry to Elasticsearch.

```

If the setup completes successfully, Environment Watch is now configured for the environment. If any errors are encountered while entering Relativity or Elastic parameters, there will be three retry attempts before the CLI forces an exit, at which point the setup process must be restarted.

Refer to the [Troubleshooting Guide](troubleshooting/relativity-server-cli.md) if you encounter any issues.

## Next Steps

* [Click here if setting up Data Grid Audit](datagrid_audit_setup.md)
* [Click here to continue Environment Watch Setup](install_environment_watch_monitoring_agents.md)



