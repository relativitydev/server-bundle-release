# SQL Cluster Configuration  
> [!NOTE]
> This SQL Cluster Configuration is only required if the environment is using a SQL Cluster servers for Environment Watch.

SQL Cluster Configuration ensures high availability and fault tolerance by distributing database workloads across multiple nodes. The setup involves configuring failover clustering, shared storage, and network connectivity.

An environment may include SQL Cluster instances comprising two or more nodes. To monitor these instances with the Environment Watch Windows Service, specific configurations must be defined in a custom JSON file.

---

## Custom JSON Configuration Structure

To monitor SQL Cluster instances using the Environment Watch Windows Service, configure the SQL cluster details in a custom JSON file. This file should be stored in the `BCPPath` directory within the `EnvironmentWatch` folder and named `environment-watch-configuration.json`. An example of the BCPPath and folder structure is shown below:

![](/resources/bcp-path-custom-json-file-name.png)

The custom JSON file includes the following key sections:
- Monitoring by Instance
- Monitoring by Installed Product
- Monitoring by Host

More details about this sections will be available in an upcoming https://github.com/relativitydev/server-bundle-release/pull/45 PR, once it is merged a link to the document will be provided here.

---

## Configuring SQL Cluster Instances

To configure SQL Cluster instances in the custom JSON file, the "hosts" section needs to be updated.
Locate the "hosts" section in the JSON file and add an entry for each SQL Cluster instance you want to monitor. Each host should be included with the following details:

Example:
If an environment includes a SQL cluster with two nodes named `SQLNode1` and `SQLNode2`, the "hosts" section should be updated for each host with the SQL Cluster Name and Instance Name, as shown below:

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

In this example, both `SQLNode1` and `SQLNode2` are configured to monitor the SQL Cluster instance named `SQL_INSTANCE` on the virtual cluster name `SQLCLUSTER`.

---

## Restart the Environment Watch Windows Service

After updating the `environment-watch-configuration.json` file with the SQL Cluster configuration, restart the Environment Watch Windows Service to apply the changes. This ensures that the service reads the updated configuration and begins monitoring the specified SQL Cluster instances.

Once the windows service has been restarted, verify that the SQL Cluster instances are being monitored correctly by checking the Environment Watch dashboard for relevant metrics, alerts, and dashboards.

### Verification in Kibana

- Navigate to Kibana Discover.
- Select `logs-*` Data View.
- Search for "The Environment Watch shared configuration object is not empty" which indicates that the EW Windows Service fetching values from the Custom JSON configuration successfully.
![](/resources/environment-watch-shared-settings-not-empty.png)
- Search for "Processed SQL instance details" and "labels.IsProvidedByCustomConfiguration attribute" should be true indicating that the SQL instance details were processed through custom JSON successfully as shown below:
![](/resources/processed-sql-details-true.png)
- Navigate to the Kibana SQL Dashboards.
- Verify data is being populated for all Kibana Dashboards.
- Navigate to Kibana Overview/Alerts.
- Verify that there are no alerts triggered.

---






