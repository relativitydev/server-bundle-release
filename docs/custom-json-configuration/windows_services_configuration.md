# Windows Service Configuration

This section describes the configuration of windows services in the `environmentWatchConfiguration` JSON object for monitoring Windows services.

---

## Custom JSON Configuration Structure

To monitor Windows Services, configure the Windows Services details in a Custom JSON file. This file should be stored in the `BCPPath` directory within the `EnvironmentWatch` folder and named `environment-watch-configuration.json`. An example of the BCPPath and folder structure is shown below:

![](/resources/sql-cluster-images/bcp-path-custom-json-file-name.png)

To identify the BCP path for the environment, execute the following SQL query against the 'EDDS' database:

```sql
SELECT TemporaryDirectory FROM [EDDS].[eddsdbo].[ResourceServer] WHERE ArtifactID=1015096
```

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

## Configure Windows Services

> [!NOTE]
> Windows Service names are case-sensitive and must match exactly as they appear in the Services application.

Windows Services can be monitored by "**hosts**", "**instance**", or "**installedProducts**". For Windows Services to monitor, locate "**windowsServices**" under the desired section and update the configuration as below.

- `enabled` : Set to `true` to enable Windows services monitoring.
- `include` : List the service names to monitor.

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

### Verification in Kibana

- Navigate to Kibana Discover.
- Select `logs-*` Data View.
- Search for "The Environment Watch shared configuration object is not empty" which indicates that the EW Windows Service fetching values from the Custom JSON configuration successfully.
![](/resources/custom-json-images/environment-watch-shared-settings-not-empty-generic.png)
- Ensure that the Windows services defined in the Custom JSON configuration appear on the Kibana Windows Services Dashboard. The example below demonstrates how a Windows service specified in the Custom JSON is successfully monitored and displayed on the Windows Services Dashboard.

![](/resources/custom-json-images/windows-service-json-example.png)

![](/resources/custom-json-images/windows-service-dashboard.png)

---


