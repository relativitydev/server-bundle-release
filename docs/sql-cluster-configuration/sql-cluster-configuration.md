# SQL Cluster Configuration  
> [!NOTE]
> Use the steps below only if your environment uses SQL Cluster servers for Environment Watch monitoring.

An environment may include SQL Cluster instances comprising two or more nodes. To monitor these instances with the Environment Watch Windows Service, specific configurations must be defined in a custom JSON file.

---

## Custom JSON Configuration Structure

To monitor SQL Cluster instances using the Environment Watch Windows Service, configure the SQL cluster details in a Custom JSON file. This file should be stored in the `BCPPath` directory within the `EnvironmentWatch` folder and named `environment-watch-configuration.json`. An example of the BCPPath and folder structure is shown below:

![](/resources/sql-cluster-images/bcp-path-custom-json-file-name.png)

To identify the BCP path for the environment, execute the following SQL query against the 'EDDS' database:

```sql
SELECT 
[TemporaryDirectory]
FROM [EDDS].[eddsdbo].[ResourceServer] AS rs WITH(NOLOCK)
INNER JOIN [EDDS].[eddsdbo].[ExtendedArtifact] AS ea WITH(NOLOCK)
	ON ea.[ArtifactID] = rs.[ArtifactID]
INNER JOIN [EDDS].[eddsdbo].[Code] AS c WITH(NOLOCK)
	ON c.[ArtifactID] = rs.[Type]
INNER JOIN [EDDS].[eddsdbo].[ResourceGroupSQLServers] AS rgss WITH(NOLOCK)
	ON rgss.[SQLServerArtifactID] = rs.[ArtifactID]
INNER JOIN [EDDS].[eddsdbo].[ResourceGroup] AS rg WITH(NOLOCK)
	ON rg.[ArtifactID] = rgss.[ResourceGroupArtifactID]
WHERE c.[Name] = 'SQL - Primary'
```
![](/resources/custom-json-images/sql-bcp-path-query.png)

The Custom JSON file includes the following key sections:
- Monitoring by Instance
- Monitoring by Installed Product
- Monitoring by Host

For more information about the Custom JSON structure, refer to the [Custom JSON Configuration Structure](../custom-json-configuration/environment_watch_configuration.md) document.

---

## Configure SQL Cluster Instances

To configure SQL Cluster instances in the Custom JSON file, the "**hosts**" section needs to be updated.
Locate the "hosts" section in the JSON file and add an entry for each SQL Cluster instance you want to monitor. Each **hostName** should be included with the following details:

**Example:**
For an environment containing a SQL cluster with two nodes (`SQLNode1` and `SQLNode2`):  
- Update the "hosts" section for each node by:
- Specifying the correct host name
- Setting the `enabled` flag to `true`
- Including the appropriate SQL cluster name (`clusterVirtualName`)
- Providing the corresponding instance name (`instanceName`)
- Below configuration sets both `SQLNode1` and `SQLNode2` cluster nodes to monitor the SQL cluster instance `SQL_INSTANCE` with the virtual cluster name `SQLCLUSTER`.

```json
"hosts": [
                {
                    "hostName": "SQLNode1",
                    "sources": {
                        "certificates": {
                            "enabled": false,
                            "include": []
                        },
                        "sqlServers": {
                            "enabled": true,
                            "include": [
                            {
                                "clusterVirtualName": "SQLCLUSTER",
                                "instanceName": "SQL_INSTANCE"
                            }
                          ]
                        },
                        "windowsServices": {
                            "enabled": false,
                            "include": [ "MSSQLSERVER" ]
                        }
                    },
                    "otelCollectorYamlFiles": []
                },
                {
                    "hostName": "SQLNode2",
                    "sources": {
                        "certificates": {
                            "enabled": false,
                            "include": []
                        },
                        "sqlServers": {
                            "enabled": true,
                            "include": [
                            {
                                "clusterVirtualName": "SQLCLUSTER",
                                "instanceName": "SQL_INSTANCE"
                            }
                          ]
                        },
                        "windowsServices": {
                            "enabled": false,
                            "include": [ "Dhcp" ]
                        }
                    },
                    "otelCollectorYamlFiles": []
                }
            ]
```

> [!NOTE]
> If the environment does not use SQL Cluster instances, then update `enabled` flag to false for all SQL Server instances in the "**hosts**" section to disable SQL monitoring.

---

### Restart the Environment Watch Windows Service

After updating the `environment-watch-configuration.json` file with the SQL cluster configuration, save the changes, restart the Environment Watch Windows Service to apply the changes. This ensures that the service reads the updated configuration and begins monitoring the specified SQL cluster instances.

Once the windows service has been restarted, verify that the SQL cluster instances are being monitored correctly by checking the Environment Watch discover, dashboards for relevant metrics, alerts.

### Verification in Kibana

- Navigate to Kibana Discover.
- Select `logs-*` Data View.
- Search for "The Environment Watch shared configuration object is not empty" which indicates that the EW Windows Service fetching values from the Custom JSON configuration successfully.
![](/resources/sql-cluster-images/environment-watch-shared-settings-not-empty.png)
- Search for "Processed SQL instance details" and "labels.IsProvidedByCustomConfiguration attribute" should be true indicating that the SQL instance details were processed through custom JSON successfully as shown below:
![](/resources/sql-cluster-images/processed-sql-details-true.png)
- Select `APM-*` Data View.
- Search for "SQL Metrics" to verify that SQL Cluster instance metrics are being ingested. KQL query example:
- Example 1:
  <pre> host.name : "SqlClusterNode" and sqlserver.user.connection.count: * and labels.sqlserver_computer_name : "SqlClusterName" </pre>
  ![](/resources/sql-cluster-images/sql-cluster-distributed-metrics.png)
- Example 2:
  <pre> relsvr.agent.exists: * and host.name: "SqlClusterNode" </pre>
  ![](/resources/sql-cluster-images/sql-cluster-primary-metrics.png)
- Navigate to the Kibana SQL Dashboards.
- Verify data is being populated for all Kibana Dashboards.
- Navigate to Kibana Overview/Alerts.
- Verify that there are no alerts triggered.

---