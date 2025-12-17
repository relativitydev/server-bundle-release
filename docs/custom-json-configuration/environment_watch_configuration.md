# Custom JSON Configuration

This document provides an overview of the custom JSON configuration for Environment Watch, allowing user to monitor Windows services and certificates from a shared path for any host, and to send alerts to a Slack channel based on the specified Slack settings.

---

## Configuration Structure

The configuration is organized in a hierarchical JSON format, with top-level sections and nested objects for each monitored entity. It will be saved in the BCPPath and the inside folder EnvironmentWatch. The name of the file will be environment-watch-configuration.json. Below is an example structure and explanation:

---

## Reference Structure

- **Top-level object**: `environmentWatchConfiguration`
- **Sections**:
  - `monitoring`: Contains configuration for instance, installed products, and hosts.
    - `instance`: Defines sources monitored at the instance level.
    - `installedProducts`: A list of installed products, where each product defines its own monitoring sources.
    - `hosts`: A list of hosts, where each host defines its own monitoring sources.
  - `alertNotificationHandlers`: Defines notification handlers (e.g., Slack).

---

## Monitoring Sections

### Monitoring by Instance
The `instance` section defines sources that are monitored at the environment or system-wide level, regardless of specific products or hosts.
- **Purpose:** Monitors general resources (like certificates or Windows services) that are relevant to the entire instance.
- **Use Case:** Useful for checks that apply everywhere, such as core system services.

### Monitoring by Installed Product
The `installedProducts` section contains a list of installed products, where each product defines its own monitoring sources.
- **Purpose:** Monitors resources specific to each installed product (e.g., web server, agent).
- **Use Case:** Allows to tailor monitoring to the needs of each product, such as product-specific services or certificates.
- The following installed product values can be used in the `installedProducts` section of the custom JSON configuration.

| Property                    | Description                                                                 |
|----------------------------|-----------------------------------------------------------------------------|
| `Generic`                  | No Relativity Product is Installed                                          |
| `QM`                       | Queue Manager Server                                                        |
| `Worker`                   | Worker Server                                                               | 
| `Agent`                    | Agent Server                                                                |
| `Web`                      | Web Server Server                                                           |
| `SecretStore`              | Secret Store Server                                                         |
| `ServiceBus`               | Service Bus Server                                                          |
| `ServiceHost`              | Service Host Server                                                         |
| `SQLDistributed`           | SQL Distributed Server                                                      |
| `SQLPrimary`               | SQL Primary Server                                                          |
| `Caat`                     | Analytics Server                                                            |


### Monitoring by Host
The `hosts` section contains an array of host objects, each with its own monitoring sources.
- **Purpose:** Monitors resources on a per-host basis, such as Services or certificates unique to a particular server.
- **Use Case:** Enables granular monitoring for individual machines, supporting host-specific checks (e.g., SQL Services on a database server).

---

### Monitoring Section Breakdown

| Property                    | Description                                                                 |
|----------------------------|-----------------------------------------------------------------------------|
| `sources`                  | Specifies what is monitored (certificates, windowsServices, sqlServers, etc.)|
| `enabled`                  | Boolean flag to enable/disable monitoring for the source.                    |
| `include`                  | List of specific items to monitor (service names, certificate details, etc.) |
| `otelCollectorYamlFiles`   | List of OpenTelemetry Collector YAML files (empty in this example).          |

---

## Configuration File Location and BCP Path

Base path: `BCPPath`

Folder: `EnvironmentWatch`

File name: `environment-watch-configuration.json`

Environment Watch automatically reads this file and applies the defined monitoring rules to the relevant instances, products, and hosts.

To identify the BCP path for the environment, execute the following SQL query against the '**EDDS**' database:

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

 An example of the BCPPath and folder structure is shown below:

![](/resources/custom-json-images/bcp-path-custom-json-file-name.png)

---

## Monitoring Source Types

This section describes the main types of sources that can be monitored using the Environment Watch configuration: Windows services, certificates, Kibana Alerts through Slack notifications, and SQL cluster instances. Each source type has its own configuration structure and properties. Following are the details for each source type:

---
### Windows Services

For detailed instructions, see [Windows Service Configuration](windows_services_configuration.md).

### Certificates

For detailed instructions, see [Certificates Configuration](certificates_configuration.md).

### SQL Cluster Instances

For detailed instructions, see [SQL Cluster Configuration](../sql-cluster-configuration/sql-cluster-configuration.md).

### Kibana Alert Slack Notification Handler

For detailed instructions, see [Alert Notification Handlers](alert_notification_handlers_configuration.md).

## Example Configuration

```json
{
	"environmentWatchConfiguration": {
		"monitoring": {
			"instance": {
				"sources": {
					"certificates": {
						"enabled": false,
						"include": []
					},
					"windowsServices": {
						"enabled": true,
						"include": [
							"WindowsAzureGuestAgent",
							"mpssvc"
						]
					}
				},
				"otelCollectorYamlFiles": []
			},
			"installedProducts": [
				{
					"productName": "Web",
					"sources": {
						"certificates": {
							"enabled": true,
							"include": [
								{
									"storeName": "My",
									"storeLocation": "LocalMachine",
									"thumbPrint": "A54225760344699530649239D175BAA73C70DC1B"
								}
							]
						},
						"windowsServices": {
							"enabled": true,
							"include": [
								"IISADMIN",
								"WinDefend"
							]
						}
					},
					"otelCollectorYamlFiles": []
				},
				{
					"productName": "Agent",
					"sources": {
						"certificates": {
							"enabled": false,
							"include": []
						},
						"windowsServices": {
							"enabled": true,
							"include": [
								"RpcSs"
							]
						}
					},
					"otelCollectorYamlFiles": []
				}
			],
			"hosts": [
				{
					"hostName": "SQL01",
					"sources": {
						"certificates": {
							"enabled": true,
							"include": []
						},
						"sqlServers": {
							"enabled": true,
							"include": [
								{
									"clusterVirtualName": "SQLClusterName",
									"instanceName": "InstanceName"
								}
							]
						},
						"windowsServices": {
							"enabled": true,
							"include": [
								"MSSQLSERVER"
							]
						}
					},
					"otelCollectorYamlFiles": []
				},
				{
					"hostName": "DG01",
					"sources": {
						"certificates": {
							"enabled": true,
							"include": []
						},
						"windowsServices": {
							"enabled": true,
							"include": [
								"Dhcp"
							]
						}
					},
					"otelCollectorYamlFiles": []
				},
				{
					"hostName": "DG02",
					"sources": {
						"certificates": {
							"enabled": false,
							"include": []
						},
						"windowsServices": {
							"enabled": true,
							"include": [
								"Schedule"
							]
						}
					},
					"otelCollectorYamlFiles": []
				},
				{
					"hostName": "CORE01",
					"sources": {
						"certificates": {
							"enabled": true,
							"include": [
								{
									"storeName": "My",
									"storeLocation": "LocalMachine",
									"thumbPrint": "F8809D2677E010477847C92C5A1A673784537CBC"
								},
								{
									"storeName": "My",
									"storeLocation": "LocalMachine",
									"thumbPrint": "984812C68F059EB19A346D538ECFB072968C11C3"
								},
								{
									"storeName": "My",
									"storeLocation": "LocalMachine",
									"thumbPrint": "984812C68F059EB19A346D538ECFB072968C11C3"
								}
							]
						},
						"windowsServices": {
							"enabled": false,
							"include": []
						}
					},
					"otelCollectorYamlFiles": []
				},
				{
					"hostName": "CORE02",
					"sources": {
						"certificates": {
							"enabled": true,
							"include": [
								{
									"storeName": "My",
									"storeLocation": "LocalMachine",
									"thumbPrint": "F8809D2677E010477847C92C5A1A673784537CBC"
								}
							]
						},
						"windowsServices": {
							"enabled": false,
							"include": []
						}
					},
					"otelCollectorYamlFiles": []
				}
			]
		},
		"alertNotificationHandlers": {
			"slack": {
				"accessToken": "slack-access-token",
				"acknowledgeAlertEnabled": false,
				"channel": "ABC12M3PQR4",
				"enabled": true,
				"messageIntervalSeconds": 180
			}
		}
	}
}
```
