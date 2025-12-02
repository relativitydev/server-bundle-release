# Enable Data Grid Audit

![Setup Stage](../resources/enable_environmentwatch.png)

> [!NOTE]
> This section applies to Datagrid Only.

After installing the required Elastic components for Data Grid Audit, the integration between Elastic and Relativity is configured by running the Relativity Server CLI on the Primary SQL Server.


> Please review the following important information before proceeding:
> * **For Existing Data Grid Audit Customers:** You must be on Elasticsearch 7.17 when initially running this setup. After the setup is complete, you can upgrade to Elasticsearch 8.17.3.
> * Before upgrading to Elasticsearch 8.17.3, the `ESIndexCreationSetting` may need to be updated. For details, refer to the [Instance setting Details](https://help.relativity.com/Server2024/Content/System_Guides/Instance_Setting_Guide/Instance_setting_descriptions.htm#ESIndexCreationSettings).
> * Always verify the minimum required Elasticsearch version in your specific release bundle, as it may differ from the versions mentioned here.

### Prerequisites 

1. Install the mapper-size plugin on all nodes in the Elasticsearch cluster (instructions available [here](https://www.elastic.co/guide/en/elasticsearch/plugins/current/mapper-size.html)). The Elasticsearch service must also be restarted after installing the plugin.

2. The Server-bundle zip file has been downloaded and extracted to `C:\Server.Bundle.x.y.z'
   
3. Verify that the InfraWatch Services application is installed in the Relativity instance (this RAP is delivered as part of the base Relativity Server 2024 installation package).



### Set up instructions

Follow these steps to set up Data Grid Audit using the Relativity Server CLI. All setup will occur on the SQL Primary server.

1. Open elevated command prompt/powershell. Run below command. Select **Datagrid**
    ```
    C:\Server.Bundle.x.y.z\relsvr.exe setup
    Relativity Server CLI - 24.0.1196
    Copyright (c) 2025, Relativity ODA LLC

    What would you like to setup?
    > DataGrid
      Environment watch
      Exit
    ```

2. Enter the required Relativity and Elasticsearch parameters.

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

3. Wait for Setup to Complete.

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

4. Restart the Relativity services on all machines for the changes to take effect.


5. Verify Audit Dashboard - navigate to the Audit tab in the Relativity environment and confirm that the dashboard and its data are loading correctly.


## Elasticsearch License Configuration

Relativity Server customers are currently using the platinum license for authentication between Relativity and Elasticsearch for the Audit application. From Elasticsearch 8.x onwards, authentication will use API keys instead of the platinum license.

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


## Next

[Click here for the next step](install_environment_watch_monitoring_agents.md)