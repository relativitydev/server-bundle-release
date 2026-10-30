# Email Alert Notifications

Environment Watch can send an email when a Kibana alert fires. Email is configured as a handler in the `alertNotificationHandlers` section of the `environment-watch-configuration.json` file, alongside the [Slack handler](ew-json-configuration-03-slack.md).

## Overview

When an Environment Watch alert becomes active, the Environment Watch agent sends an HTML email to every enabled recipient in the `email` handler. The agent forwards the same Kibana alerts to the email handler that it forwards to the Slack handler. You can use email, Slack, or both.

Each email includes the following details for every alert:

- Alert name and status
- Link to the Kibana rule
- Link to the alert details page, when available
- Resolution guidance text and link, when available
- Alert query
- Alert interval
- Date and time the alert was last executed
- Duration of the last execution, in milliseconds

> **Note**: Date and time values in the email use the local time of the server that runs the Environment Watch agent.

## Prerequisites

Email notifications use the SMTP settings that are already configured in Relativity. You do not need to configure a separate SMTP server for Environment Watch.

Verify that the following Instance Settings are set in the `kCura.Notification` section:

| Instance Setting    | Description                                                                                         |
|---------------------|-----------------------------------------------------------------------------------------------------|
| `SMTPServer`        | The host name or IP address of the SMTP server. Email is skipped when this setting is empty.        |
| `SMTPPort`          | The port of the SMTP server.                                                                        |
| `SMTPUserName`      | The user name used to authenticate with the SMTP server.                                            |
| `SMTPPassword`      | The password used to authenticate with the SMTP server.                                             |
| `SMTPSSLisRequired` | Specifies whether SSL is used for the SMTP connection. SSL is used when the setting is not defined. |

> **Note**: If `SMTPSSLisRequired` is `False` and `SMTPUserName` is set, the SMTP credentials are sent without encryption and the agent logs a warning. Set `SMTPSSLisRequired` to `True` when you use credentials.

## Configure the Email Handler

To configure email notifications, locate the `alertNotificationHandlers` section of the custom JSON configuration file and add an `email` handler. The file is saved in the `BCPPath` share, in the `EnvironmentWatch` folder. For details about the file, see [Custom JSON Configuration](ew-extensibility-configuration-06-json-configuration.md).

### Email handler properties

| Property                 | Description                                                                                                                                                                                                                |
|--------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `enabled`                | Boolean flag to enable or disable email notifications. The default value is `false`.                                                                                                                                       |
| `fromAddress`            | The sender email address. If it is empty, the `SMTPUserName` Instance Setting is used.                                                                                                                                     |
| `fromDisplayName`        | Optional. The sender name shown in the email.                                                                                                                                                                              |
| `recipients`             | The list of people who receive the email. At least one recipient must be enabled. Each recipient has the properties listed in the [recipient properties](#recipient-properties) table.                                      |
| `notifyOnAlertFired`     | Boolean flag. When `true`, an email is sent when an alert fires. When `false`, fired alerts are skipped and no email is sent. The default value is `true`.                                                                  |
| `messageIntervalSeconds` | The minimum time, in seconds, between emails. The minimum and default effective value is 180 seconds (3 minutes). A lower value is replaced with 180. See [Batching and cooldown](#batching-and-cooldown).               |
| `maxQueueSize`           | The maximum number of alerts that wait to be sent. The default value is 100. When the limit is exceeded, for example during a long SMTP outage, the oldest alerts are dropped and a warning is logged.                      |

### Recipient properties

| Property       | Description                                                                |
|----------------|----------------------------------------------------------------------------|
| `emailAddress` | The recipient email address.                                               |
| `displayName`  | Optional. The recipient name.                                              |
| `enabled`      | Boolean flag to include or exclude the recipient. The default value is `true`. |

### Example

```json
{
  "environmentWatchConfiguration": {
    "alertNotificationHandlers": {
      "email": {
        "enabled": true,
        "fromAddress": "no-reply@company.com",
        "fromDisplayName": "Environment Watch",
        "messageIntervalSeconds": 180,
        "maxQueueSize": 100,
        "notifyOnAlertFired": true,
        "recipients": [
          {
            "emailAddress": "admin@company.com",
            "displayName": "Environment Watch Admin",
            "enabled": true
          },
          {
            "emailAddress": "oncall@company.com",
            "enabled": false
          }
        ]
      }
    }
  }
}
```

In this example, only `admin@company.com` receives emails because the `oncall@company.com` recipient is disabled.

> **Note**: The example shows only the `alertNotificationHandlers` section. In your file, keep the monitoring sections that are already defined under `environmentWatchConfiguration`.

### Turn off notifications for fired alerts

Set `notifyOnAlertFired` to `false` when you want to keep the email handler configured but stop emails for fired alerts, for example during planned maintenance. To stop all email, set `enabled` to `false`.

## Batching and cooldown

The agent groups alerts into emails to avoid sending many messages:

- The first alert after the Environment Watch agent starts is sent immediately.
- Alerts that arrive during the cooldown window are queued and sent together in the next email.
- The cooldown window is `messageIntervalSeconds`. The minimum is 180 seconds (3 minutes). If the value is not set, or is set to 180 or lower, 180 seconds is used.
- To receive emails less often, set `messageIntervalSeconds` to a larger value.

The email subject depends on the number of alerts in the email:

| Alerts in the email | Subject                                                |
|---------------------|--------------------------------------------------------|
| One                 | `Relativity Environment Watch Alert: <alert name> (<status>)` |
| More than one       | `Relativity Environment Watch: <N> Alerts`             |

## Apply configuration changes

Changes to `environment-watch-configuration.json` are applied without restarting the Environment Watch agent. The agent detects that the file changed and reloads it. When you save a change, allow a short time for the agent to pick it up.

## Reliability

- If the SMTP server cannot be reached or a send fails, the alerts are put back in the queue and sent in the next send window. The failure is logged with the alert names and the number of recipients.
- If the SMTP server rejects a recipient or the message is invalid, the alerts are not re-queued because the same error would occur again. The failure is logged.
- If a recipient address is not valid, that recipient is skipped and the email is sent to the remaining recipients.

### Behavior when email cannot be sent

| Condition                                       | Behavior                                                                                                       |
|-------------------------------------------------|----------------------------------------------------------------------------------------------------------------|
| `email` handler is not present or `enabled` is `false` | No email is sent. A debug message is logged.                                                           |
| SMTP is not configured in Instance Settings     | No email is sent. A warning is logged.                                                                         |
| No recipient is enabled                         | No email is sent. A warning is logged when the configuration is loaded.                                        |
| `fromAddress` is not valid                      | The alerts are re-queued and a warning is logged.                                                              |

For help with these conditions, see [Email Notifications](../../../troubleshooting/custom-json-troubleshooting.md#email-notifications) in the troubleshooting guide.

## Security

- SMTP credentials are never written to the logs.
- Dynamic content in the email, such as alert names and queries, is HTML-encoded.

## Notification health alert

Environment Watch monitors its own notification delivery. When an alert notification fails to send, the agent records the `relsvr.notification.send_failure` metric. This metric is recorded for each notification provider, including email and Slack.

The **Environment Watch alert notifications are not being sent** alert fires when the sum of `relsvr.notification.send_failure` is greater than 0 within a 5-minute window. The alert is checked every minute. It does not fire when no notification handler is configured.

### Import the alert

The alert is packaged with the Environment Watch installation and is imported by the Environment Watch CLI when you run `relsvr.exe setup`.

### Resolve the alert

When the alert is active:

1. Verify the SMTP host, port, user name, and password in the `kCura.Notification` Instance Settings. Rotated SMTP credentials that were not updated in Relativity are the most common cause.
2. Verify that the SMTP port is open on the firewall and that the SMTP host is reachable from the server that runs the Environment Watch agent.
3. Verify the `email` handler in `environment-watch-configuration.json`. For Slack, verify the access token and channel.

After the cause is fixed, the queued alerts are sent in the next send window and the alert returns to a recovered state.

## Troubleshooting

Refer to the [Troubleshooting Guide](../../../troubleshooting/custom-json-troubleshooting.md#email-notifications) to resolve email notification issues.
