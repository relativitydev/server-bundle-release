# Windows Service Configuration

This section describes how to configure Windows service monitoring using the `environmentWatchConfiguration` JSON object.

---

## Overview

This configuration monitors the status of specified Windows services to ensure they are running as expected. Some Windows services are monitored by default based on the installed products, even if they are not explicitly configured in the custom JSON file. 

The table below lists the service names that are monitored by default based on the installed products.

### Default Services

| Service Name                          | Description                                      |
|---------------------------------------|--------------------------------------------------|
| apm-server                          | Elastic Stack APM Server                         |
| elasticsearch                       | Elastic Stack Elasticsearch Service              |
| W3SVC                                 | Internet Information Services (IIS)              |
| QueueManager                         | Invariant Queue Manager                          |
| RabbitMQ                             | RabbitMQ Message Broker                          |
| kCura EDDS Agent Manager             | Relativity Agent Manager                         |
| Relativity Analytics Engine           | Relativity Analytics Engine (CAAT)               |
| Relativity Secret Store               | Relativity Secret Store                          |
| kCura Service Host Manager           | Relativity Service Host Manager                  |
| kCura EDDS Web Processing Manager    | Relativity Web Processing Manager                |

### Properties Table

The following table lists the properties used to configure Windows services monitoring in the custom JSON file.

| Property    | Type     | Description                                                      |
|-------------|----------|------------------------------------------------------------------|
| `enabled`   | boolean  | Enables or disables monitoring for Windows services.              |
| `include`   | array    | List of Windows service names (not display names) to monitor (for example, "WinDefend").|

To identify the correct service name:

- Navigate to the **Services** application on the host.
- Right-click the desired service and select **Properties**.
- Copy the **Service name** value (not the display name).

## Configure Windows Services

> [!NOTE]
> Windows service names are case-sensitive and must match exactly as they appear in the Services application.

Windows services can be monitored at the **hosts**", "**instance**", or "**installedProducts**" level.
For services to monitor, locate "**windowsServices**" under the desired section and update the configuration as below.

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


