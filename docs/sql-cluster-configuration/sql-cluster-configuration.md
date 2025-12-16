# SQL Cluster Configuration  
> [!NOTE]
> Use the steps below only if your environment uses SQL cluster servers for Environment Watch monitoring.

An environment may include SQL cluster instances comprising two or more nodes. To monitor these instances with the Environment Watch Windows Service, specific configurations must be defined in a custom JSON file.

---

## Configure SQL Cluster Instances

To configure SQL cluster instances in the Custom JSON file, the "**hosts**" section needs to be updated.
Locate the "hosts" section in the JSON file and add an entry for each SQL cluster instance you want to monitor. Each **hostName** should be included with the following details:

**Example:**
For an environment containing a SQL cluster with two nodes (`SQLNode1` and `SQLNode2`):  
- Update the "hosts" section for each node by:
- Specifying the correct host name
- Setting the `enabled` flag to `true`
- Including the appropriate SQL cluster name (`clusterVirtualName`)
- Providing the corresponding instance name (`instanceName`)
- Below configuration sets both `SQLNode1` and `SQLNode2` cluster nodes to monitor the SQL cluster instance `SQL_INSTANCE` with the virtual cluster name `SQLCLUSTER`.

> [!NOTE]
> SQL cluster configuration in the custom JSON file should always be specified within the "**hosts**" section.

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
> If the environment does not use SQL cluster instances, then update `enabled` flag to false for all SQL cluster instances in the "**hosts**" section to disable SQL monitoring.

---

### Restart the Environment Watch Windows Service

After updating the `environment-watch-configuration.json` file with the SQL cluster configuration, save the changes, restart the Environment Watch Windows Service to apply the changes. This ensures that the service reads the updated configuration and begins monitoring the specified SQL cluster instances.

Once the windows service has been restarted, verify that the SQL cluster instances are being monitored correctly by checking the Environment Watch discover, dashboards for relevant metrics, alerts.

### Verification in Kibana

- Navigate to Kibana Discover.
- Select `logs-*` Data View.
- Search for "The Environment Watch shared configuration object is not empty" which indicates that the EW windows service fetching values from the Custom JSON configuration successfully.

![](/resources/sql-cluster-images/environment-watch-shared-settings-not-empty.png)

- Search for "Processed SQL instance details" and "labels.IsProvidedByCustomConfiguration attribute" should be true indicating that the SQL instance details were processed through custom JSON successfully as shown below:

![](/resources/sql-cluster-images/processed-sql-details-true.png)

- Select `APM-*` Data View.
- Search for "SQL Metrics" to verify that SQL cluster instance metrics are being ingested. KQL query example:
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