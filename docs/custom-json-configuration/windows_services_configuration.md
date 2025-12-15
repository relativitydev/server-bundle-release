# Windows Service Configuration

This section describes the configuration of windows services in the `environmentWatchConfiguration` JSON object for monitoring Windows services.

---

## Custom JSON Configuration Structure

To monitor Windows Services, configure the Windows Services details in a Custom JSON file. This file should be stored in the `BCPPath` directory within the `EnvironmentWatch` folder and named `environment-watch-configuration.json`. An example of the BCPPath and folder structure is shown below:

![](/resources/sql-cluster-images/bcp-path-custom-json-file-name.png)

The Custom JSON file includes the following key sections:
- Monitoring by Instance
- Monitoring by Installed Product
- Monitoring by Host

For more information about the Custom JSON structure, refer to the [Custom JSON Configuration Structure](./environment_watch_configuration.md) document.

## Overview

Monitors the status of specified Windows services configured to ensure they are running as expected. There are windows services by default monitored without configuration and they are based on installed product. 

### Default Services

| Service Display Name                  | Description                                      |
|---------------------------------------|--------------------------------------------------|
| apm-server                           | Elastic Stack APM Server                         |
| Elasticsearch                        | Elastic Stack Elasticsearch Service              |
| W3SVC                                 | Internet Information Services (IIS)              |
| QueueManager                         | Invariant Queue Manager                          |
| RabbitMQ                             | RabbitMQ Message Broker                          |
| kCura EDDS Agent Manager             | Relativity Agent Manager                         |
| Relativity Analytics Engine           | Relativity Analytics Engine (CAAT)               |
| Relativity Secret Store               | Relativity Secret Store                          |
| kCura Service Host Manager           | Relativity Service Host Manager                  |
| kCura EDDS Web Processing Manager    | Relativity Web Processing Manager                |

### Properties Table

Go to each host and select the service you want to monitor. We need to choose service name instead of display name. 

Navigate to the Services application. Right-click the desired service, select Properties, and copy the Service name from the dialog.

| Property    | Type     | Description                                                      |
|-------------|----------|------------------------------------------------------------------|
| `enabled`   | boolean  | Enables or disables monitoring for Windows services.              |
| `include`   | array    | List of Windows service names (service name and **NOT** display name) to monitor (e.g., `"WinDefend"`).  |

## Usage
`enabled` : Set to `true` to enable Windows services monitoring.
`include` : List the service names to monitor.

> [!NOTE]
> Windows Service names are case-sensitive and must match exactly as they appear in the Services application.

Windows Services can be monitored by Hosts, Instances, or Installed Products. Below example can be used to monitor Windows Services by Hosts, Instances or Installed Products.

**Example**
```json
{
	"windowsServices": {
		"enabled": true,
		"include": [
			"WindowsAzureGuestAgent",
			"mpssvc"
		]
	}
}
```
