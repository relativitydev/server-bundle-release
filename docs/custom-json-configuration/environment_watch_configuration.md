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
    - `installedProducts`: Array of product objects, each with its own sources.
    - `hosts`: Array of host objects, each with its own sources.
  - `alertNotificationHandlers`: Defines notification handlers (e.g., Slack).

---

## Monitoring Sections

### Monitoring by Instance
The `instance` section defines sources that are monitored at the environment or system-wide level, regardless of specific products or hosts.
- **Purpose:** Monitors general resources (like certificates or Windows services) that are relevant to the entire instance.
- **Use Case:** Useful for checks that apply everywhere, such as core system services.

### Monitoring by Installed Product
The `installedProducts` section contains an array of product objects, each with its own monitoring sources.
- **Purpose:** Monitors resources specific to each installed product (e.g., web server, agent).
- **Use Case:** Allows you to tailor monitoring to the needs of each product, such as product-specific services or certificates.

### Monitoring by Host
The `hosts` section contains an array of host objects, each with its own monitoring sources.
- **Purpose:** Monitors resources on a per-host basis, such as services or certificates unique to a particular server.
- **Use Case:** Enables granular monitoring for individual machines, supporting host-specific checks (e.g., SQL services on a database server).

---

### Monitoring Section Breakdown

| Property                    | Description                                                                 |
|----------------------------|-----------------------------------------------------------------------------|
| `sources`                  | Specifies what is monitored (certificates, windowsServices, sqlServers, etc.)|
| `enabled`                  | Boolean flag to enable/disable monitoring for the source.                    |
| `include`                  | List of specific items to monitor (service names, certificate details, etc.) |
| `otelCollectorYamlFiles`   | List of OpenTelemetry Collector YAML files (empty in this example).          |

---


## Monitoring Source Types

This section describes the main types of sources that can be monitored using the Environment Watch configuration: Windows Services, Certificates, and SQL Services. Each source type has its own configuration structure and properties.

---

### Windows Services

Refer - [Windows Service Configuration](windows_services_configuration.md)

### Certificates

Refer - [Certificates Configuration](certificates_configuration.md)


### SQL Cluster Instances

Refer - [SQL Cluster Configuration](../sql-cluster-configuration/sql-cluster-configuration.md)

**Properties Table**

| Property    | Type     | Description                                                      |
|-------------|----------|------------------------------------------------------------------|
| `enabled`   | boolean  | Enables or disables monitoring for SQL services.                 |
| `include`   | array    | List of SQL Server instance names to monitor (e.g., `"VIRTSQL\\INSTANCE01"`). |


**Example**
```json
"sqlServers": {
              "enabled": true,
              "include": [
                {
                  "clusterVirtualName": "SQLClusterName",
                  "instanceName": "InstanceName"
                }
              ]
            }
```

### Alert Notification Handler

Refer - [Alert Notification Handlers](alert_notification_handlers_configuration.md)


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
									"StoreName": "My",
									"StoreLocation": "LocalMachine",
									"Thumbprint": "A54225760344699530649239D175BAA73C70DC1B"
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
									"thumbprint": "F8809D2677E010477847C92C5A1A673784537CBC"
								},
								{
									"storeName": "My",
									"storeLocation": "LocalMachine",
									"thumbprint": "984812C68F059EB19A346D538ECFB072968C11C3"
								},
								{
									"storeName": "My",
									"storeLocation": "LocalMachine",
									"thumbprint": "984812C68F059EB19A346D538ECFB072968C11C3"
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
									"thumbprint": "F8809D2677E010477847C92C5A1A673784537CBC"
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
