## Alert Notification Handlers [Early Access]

The "handlers" allow a client to send alert notifications to the specified provider.

### Slack Handler

The Slack handler allows alerts to be sent to a designated Slack channel. Configuration options include:

| Property                   | Description                                                                 |
|----------------------------|-----------------------------------------------------------------------------|
| `accessToken`              | The Slack API token used for authentication.                                |
| `acknowledgeAlertEnabled`  | Boolean flag to enable/disable alert acknowledgment in Slack. This is by default false since implementation is not done.               |
| `channel`                  | The Slack channel ID where alerts will be posted.                         |
| `enabled`                  | Boolean flag to enable/disable Slack notifications.                         |
| `messageIntervalSeconds`   | Interval (in seconds) between alert messages sent to Slack. It should be greater than or equal to the minimum Slack interval in seconds, i.e., 180.               |

---

### Configure Slack in Custom JSON Configuration File

#### Prerequisites

Before configuring Slack notifications:

- Create a Slack App in your Slack workspace.
- Generate a "Bot User OAuth Token" with the required permissions to post messages to channels.

#### Configuration

To configure Slack notification in the custom JSON configuration file, locate the `alertNotificationHandlers` section and update the configuration as below.

```json
{
  "environmentWatchConfiguration": {
    "monitoring": {
      "instance": {
        "sources": {
          "certificates": {},
          "windowsServices": {
            "enabled": false,
            "include": []
          }
        },
        "otelCollectorYamlFiles": []
      },
      "installedProducts": [
        {
          "productName": "Agent",
          "sources": {
            "certificates": {
              "enabled": true,
              "include": []
            },
            "windowsServices": {
              "enabled": true,
              "include": []
            }
          },
          "otelCollectorYamlFiles": []
        }
      ],
      "hosts": [
        {
          "hostName": "emttest",
          "sources": {
            "certificates": {
              "enabled": true,
              "include": []
            },
            "sqlServers": {
              "enabled": false,
              "include": []
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
				"accessToken": "Bot User OAuth Token",
				"acknowledgeAlertEnabled": false,
				"channel": "slack-channel-id",
				"enabled": true,
				"messageIntervalSeconds": 60
			}
		}
  }
}
```

### Verification in Kibana

- Navigate to Kibana Dashboard.
- Select Alerts Overview Dashboard, observe for any triggered alerts.
- Verify the Slack notifications are received in the specified Slack channel.

### Slack Notification Example

![Slack Example](../../resources/slackalerts-images/SlackNotification.png)

## Troubleshooting
Refer to the [Troubleshooting Guide](../troubleshooting/custom-json-troubleshooting.md) to resolve any custom JSON slack configuration issues.

