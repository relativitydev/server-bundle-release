# SQL Server Configuration 
> [!NOTE]
> Use the steps below only to explicitly configure SQL Server instances or if the environment uses SQL cluster servers for Environment Watch monitoring.

SQL Primary and SQL Distributed instances are auto-configured by default; however, this feature also allows for manual configuration of the instances. Monitoring these instances using the Environment Watch Windows service requires defining specific configurations in a custom JSON configuration file.

---

## Configure SQL Server Instances

Define SQL server configuration in the custom JSON configuration file within the "**hosts**" section.

> [!NOTE]
> SQL server configuration in the custom JSON configuration file must always be specified within the "**hosts**" section.

Locate the "**hosts**" section in the JSON file and add an entry for each SQL server instance to be monitored. Include each **hostName** with the following details:

**Example:**
For an environment containing a SQL server with two nodes (`SQLNode1` and `SQLNode2`):  
Update the "hosts" section for each node by:
- Specify the correct host name
- Set the `enabled` flag to `true`
- Specify the appropriate SQL server/cluster name (`clusterVirtualName`)
- Provide the corresponding instance name (`instanceName`)
- Below configuration sets both `SQLNode1` and `SQLNode2` nodes to monitor the SQL server instance `SQL_INSTANCE` with the virtual cluster name `SQLCLUSTER`.

```json
{
  "environmentWatchConfiguration": {
    "monitoring": {
      "instance": {
        "sources": {
          "certificates": {},
          "windowsServices": {}
        },
        "otelCollectorYamlFiles": []
      },
      "installedProducts": [
        {
          "productName": "Agent",
          "sources": {
            "certificates": {},
            "windowsServices": {}
          },
          "otelCollectorYamlFiles": []
        }
      ],
      "hosts": [
        {
          "hostName": "SQLNode1",
          "sources": {
            "certificates": {},
            "sqlServers": {
              "enabled": true,
              "include": [
                {
                  "clusterVirtualName": "SQLCLUSTER",
                  "instanceName": "SQL_INSTANCE"
                }
              ]
            },
            "windowsServices": {}
          },
          "otelCollectorYamlFiles": []
        },
        {
          "hostName": "SQLNode2",
          "sources": {
            "certificates": {},
            "sqlServers": {
              "enabled": true,
              "include": [
                {
                  "clusterVirtualName": "SQLCLUSTER",
                  "instanceName": "SQL_INSTANCE"
                }
              ]
            },
            "windowsServices": {}
          },
          "otelCollectorYamlFiles": []
        }
      ]
    },
    "alertNotificationHandlers": {}
  }
}
```

> [!NOTE]
> If the environment does not use SQL server instances, then update `enabled` flag to false for all SQL server instances in the "**hosts**" section to disable SQL monitoring.

---

### Restart the Environment Watch Windows Service

After updating the `environment-watch-configuration.json` file with the SQL server configuration, save the changes, restart the Environment Watch Windows service to apply the changes. This ensures that the service reads the updated configuration and begins monitoring the specified SQL server instances.

Once the Windows service has been restarted, verify the SQL instances are being monitored correctly by checking the Environment Watch discover, dashboards for relevant metrics, alerts.

### Verification in Kibana

- Navigate to the Kibana SQL Dashboards.
- Verify data is being populated for all Kibana Dashboards.
- Navigate to Kibana Overview/Alerts.
- Verify that there are no alerts triggered.

---

## Troubleshooting
Refer to the [Troubleshooting Guide](../../../troubleshooting/custom-json-troubleshooting.md) to resolve any custom JSON SQL server configuration issues.