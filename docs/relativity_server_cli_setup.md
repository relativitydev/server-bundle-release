# Enable Environment Watch

![Setup Stage](../resources/enable_environmentwatch.png)

# Set up Environment Watch using the Relativity Server CLI


### Prerequisites

1. Elastic Stack Certificates are installed on all Servers
2. Access to Elastic Stack, Primary SQL Server, and Secret Store (Whitelisted for Secret Store access. Please see [here](https://help.relativity.com/Server2024/Content/System_Guides/Secret_Store/Secret_Store.htm#Configuringclients) for information on whitelisting.)
3. The Server-bundle zip file has been downloaded and extracted to `C:\Server.Bundle.x.y.z'

### Set up instructions

**Step 1** - Execute the following command in an elevated Command/Powershell to enter the setup workflow.
```powershell
./relsvr.exe setup
```

**Step 2** - Select Environment Watch

```
Relativity Server CLI - 24.0.1182
Copyright (c) 2025, Relativity ODA LLC

What would you like to setup?
DataGrid
> Environment watch
Exit
```

**Step 3** - Enter Relativity Parameters

```
Confirm you would like to perform the 'Environment Watch' setup [y/n] (y): y

Retrieved existing settings
Enter the Relativity admin username (relativity.admin@kcura.com): relativity.admin@kcura.com
Enter the Relativity admin password: *********
Enter the Relativity instance url (https://emttest/Relativity): https://emttest/Relativity
Relativity instance is verified
```

**Step 4** - Enter Elastic Stack Parameters

```
Enter the Elasticsearch admin username (elastic): elastic
Enter the Elasticsearch admin password: *********
Enter the Elasticsearch cluster endpoint URL (https://emttest:9200): https://emttest:9200
Elasticsearch cluster endpoint URL is verified
Enter the Elasticsearch APM Server endpoint URL (http://emttest:8200): http://emttest:8200
```

**Step 5** - Wait for Setup to Complete
The CLI will now complete the setup process.
```
API Key creation and validation completed ------------------------- 100%
OAuth2 client exists -------------------------------------------- 100%
```

If the setup completes successfully, Environment Watch is now configured for the environment. If any errors are encountered while entering Relativity or Elastic parameters, there will be three retry attempts before the CLI forces an exit, at which point the setup process must be restarted.

Refer to the [Troubleshooting Guide](troubleshooting/relativity-server-cli.md) if you encounter any issues.

## Next Steps

* [Click here if setting up Data Grid Audit](datagrid_audit_setup.md)
* [Click here to continue Environment Watch Setup](install_environment_watch_monitoring_agents.md)



