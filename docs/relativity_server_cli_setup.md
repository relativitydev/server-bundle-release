# Enable Environment Watch

![Setup Stage](../resources/enable_environmentwatch.png)

> [!NOTE]
> This section applies to Environment Watch Only.

# Set up Environment Watch using the Relativity Server CLI


### Prerequisites

1. Elastic Stack Certificates are installed on all Servers
2. Verify that the Elastic Stack services (Elasticsearch, Kibana, and APM) are    running
3. Access to Elastic Stack, Primary SQL Server, and Secret Store (Whitelisted for Secret Store access. Please see [here](https://help.relativity.com/Server2024/Content/System_Guides/Secret_Store/Secret_Store.htm#Configuringclients) for information on whitelisting.)
4. The Server-bundle zip file has been downloaded and extracted to `C:\Server.Bundle.x.y.z'

### Set up instructions

**Step 1** - Open elevated command prompt/powershell. Navigate to the directory where the CLI is extracted, and run ".\relsvr.exe setup"

```
C:\Server.Bundle.x.y.z> .\relsvr.exe setup

Relativity Server CLI - 24.0.1196
Copyright (c) 2025, Relativity ODA LLC

What would you like to setup?
  DataGrid
> Environment watch
  Exit
```

**Step 2** - Confirm the Environment Watch setup.
```
Confirm you would like to perform the 'Environment Watch' setup [y/n] (y): y
```

**Step 3** - Enter the required Relativity parameters.

```
Existing settings do not exist
Enter the Relativity admin username (relativity.admin@kcura.com): relativity.admin@kcura.com
Enter the Relativity admin password: *********
Enter the Relativity instance url (https://emttest/Relativity): https://emttest/Relativity
Relativity instance is verified
```

**Step 4** - Enter the required Elastic Stack parameters.

```
Enter the Elasticsearch admin username (elastic): elastic
Enter the Elasticsearch admin password: *********
Enter the Elasticsearch cluster endpoint URL (https://emttest:9200): https://emttest:9200
Elasticsearch cluster endpoint URL is verified
Enter the Elasticsearch APM Server endpoint URL (http://emttest:8200): http://emttest:8200
Enter the Elasticsearch Kibana endpoint URL (http://emttest:5601): http://emttest:5601
```

**Step 5** - The CLI will now apply the necessary configurations. Please wait for the process to finish.

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

Successful completion indicates that Environment Watch is configured. The setup process will automatically retry three times on parameter entry errors before exiting. If it exits, the process must be restarted..

Refer to the [Troubleshooting Guide](troubleshooting/relativity-server-cli.md) if you encounter any issues.

## Next Steps

* [Click here if setting up Data Grid Audit](datagrid_audit_setup.md)
* [Click here to continue Environment Watch Setup](install_environment_watch_monitoring_agents.md)



