# Windows Service Configuration

This section describes the configuration of windows services in the `environmentWatchConfiguration` JSON object for monitoring Windows services.

---

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
| `include`   | array    | List of Windows service names to monitor (e.g., `"WinDefend"`).  |

**Example**---
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
