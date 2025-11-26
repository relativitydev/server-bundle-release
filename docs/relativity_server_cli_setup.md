# Set up Environment Watch using the Relativity Server CLI

![Setup Stage](../resources/enable_environmentwatch.png)

> [!NOTE]
> This step is required for Environment Watch.

> It is recommended to run the CLI from the Primary SQL Server.

### Prerequisites

- Elastic Stack Certificates are installed on all Servers
- Verify that the Elastic Stack services (Elasticsearch, Kibana, and APM) are    running
- Access to Elastic Stack, Primary SQL Server, and Secret Store (Whitelisted for Secret Store access. Please see [here](https://help.relativity.com/Server2024/Content/System_Guides/Secret_Store/Secret_Store.htm#Configuringclients) for information on whitelisting.)
- The Server-bundle zip file has been downloaded and extracted to `C:\Server.Bundle.x.y.z'

### Steps for changing license

If you need to change the Elasticsearch license from platinum to basic, follow these steps:

1. **Check the current license status**

   Verify the current license type by accessing the Elasticsearch license endpoint. Replace `<hostname_or_ip>` with your Elasticsearch hostname or IP address:

   ```
   https://<hostname_or_ip>:9200/_license
   ```

   Example response for platinum license:
   ```json
   {
     "license" : {
       "status" : "active",
       "uid" : "a523a4e6-9cec-4f97-a922-eadb0d7c8fda",
       "type" : "platinum",
       "issue_date" : "2025-04-16T00:00:00.000Z",
       "issue_date_in_millis" : 1744761600000,
       "expiry_date" : "2026-04-30T23:59:59.999Z",
       "expiry_date_in_millis" : 1777593599999,
       "max_nodes" : 72,
       "max_resource_units" : null,
       "issued_to" : "kCura Corporation (non-production environments)",
       "issuer" : "API",
       "start_date_in_millis" : 1744761600000
     }
   }
   ```

2. **Change license to basic using Kibana Dev Tools**

   Navigate to Kibana Dev Tools and execute the following API call:

   ```
   POST /_license/start_basic?acknowledge=true
   ```

   Expected response:
   ```json
   {
     "acknowledged": true,
     "basic_was_started": true
   }
   ```

3. **Verify the license change**

   Confirm that the license has been successfully changed to basic by checking the license status again:

   ```
   https://<hostname_or_ip>:9200/_license
   ```

   Example response for basic license:
   ```json
   {
     "license" : {
       "status" : "active",
       "uid" : "3c332613-ef4f-4fcf-8505-982551476532",
       "type" : "basic",
       "issue_date" : "2025-11-26T09:22:27.879Z",
       "issue_date_in_millis" : 1764148947879,
       "max_nodes" : 1000,
       "max_resource_units" : null,
       "issued_to" : "elasticsearch",
       "issuer" : "elasticsearch",
       "start_date_in_millis" : -1
     }
   }
   ```

### Set up instructions

1. Open elevated command prompt/powershell. Run the below command. Select Environment Watch

	```
	C:\Server.Bundle.x.y.z\relsvr.exe setup

	Relativity Server CLI - 24.0.1196
	Copyright (c) 2025, Relativity ODA LLC

	What would you like to setup?
	  DataGrid
	> Environment watch
	  Exit
	```

2. Confirm the Environment Watch setup.

	```
	Confirm you would like to perform the 'Environment Watch' setup [y/n] (y): y
	```

2. Enter the required Relativity parameters.

	```
	Existing settings do not exist
	Enter the Relativity admin username (relativity.admin@kcura.com): relativity.admin@kcura.com
	Enter the Relativity admin password: *********
	Enter the Relativity instance url (https://emttest/Relativity): https://emttest/Relativity
	Relativity instance is verified
	```

4. Enter the required Elastic Stack parameters.

	```
	Enter the Elasticsearch admin username (elastic): elastic
	Enter the Elasticsearch admin password: *********
	Enter the Elasticsearch cluster endpoint URL (https://emttest:9200): https://emttest:9200
	Elasticsearch cluster endpoint URL is verified
	Enter the Elasticsearch APM Server endpoint URL (http://emttest:8200): http://emttest:8200
	Enter the Elasticsearch Kibana endpoint URL (http://emttest:5601): http://emttest:5601
	```

5. The CLI will now apply the necessary configurations. Please wait for the process to finish.

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

6. Successful completion indicates that Environment Watch is configured. The setup process will automatically retry three times on parameter entry errors before exiting. If it exits, the process must be restarted..

Refer to the [Troubleshooting Guide](troubleshooting/relativity-server-cli.md) if you encounter any issues.

## Next Steps

* [Click here if setting up Data Grid Audit](datagrid_audit_setup.md)
* [Click here to continue Environment Watch Setup](install_environment_watch_monitoring_agents.md)



