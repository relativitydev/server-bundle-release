# Windows Service Configuration

This section describes the configuration of Windows services in the `environmentWatchConfiguration` JSON object for monitoring Windows services.

---

## Overview

Monitors the status of specified Windows services configured to ensure they are running as expected. There are Windows services by default monitored without configuration and they are based on installed product. 

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

Navigate to the Services application. Right-click the desired service, select Properties, and copy the service name from the dialog.

| Property    | Type     | Description                                                      |
|-------------|----------|------------------------------------------------------------------|
| `enabled`   | boolean  | Enables or disables monitoring for Windows services.              |
| `include`   | array    | List of Windows service names (service name and **NOT** display name) to monitor (e.g., `"WinDefend"`).  |

## Configure Windows Services

> [!NOTE]
> Windows service names are case-sensitive and must match exactly as they appear in the services application.

Windows services can be monitored by "**hosts**", "**instance**", or "**installedProducts**". For Windows services to monitor, locate "**windowsServices**" under the desired section and update the configuration as below.

- `enabled` : Set to `true` to enable Windows services monitoring.
- `include` : List the service names to monitor.

**Example**
```json
{
	"windowsServices": {
		"enabled": true,
		"include": [
			"Spooler"
		]
	}
}
```

### Verification in Kibana

- Navigate to Kibana Discover.
- Select `logs-*` Data View.
- Search for "The Environment Watch shared configuration object is not empty" which indicates that the EW Windows service fetching values from the custom JSON configuration successfully.

![](/resources/custom-json-images/environment-watch-shared-settings-not-empty-generic.png)
- Ensure that the Windows services defined in the custom JSON configuration appear on the Kibana Windows services dashboard. The example below demonstrates how a Windows service specified in the custom JSON is successfully monitored and displayed on the Windows services dashboard.

![](/resources/custom-json-images/windows-service-json-example.png)

![](/resources/custom-json-images/windows-service-dashboard.png)

---


