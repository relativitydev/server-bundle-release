# Email Alert Notifications

Environment Watch can send an email while a Kibana alert is in a non-OK status. Email is configured as a handler in the `alertNotificationHandlers` section of the `environment-watch-configuration.json` file, alongside the [Slack handler](ew-json-configuration-03-slack.md).

## Overview

While a Kibana alert is in a non-OK status, the Environment Watch agent sends an HTML email to every enabled recipient in the `email` handler. You can use email, Slack, or both.

### Alerts that trigger an email

The agent checks all Kibana alerting rules, including the alerts provided by Relativity and any custom rules, every 3 minutes by default. An email is sent for each alert whose last execution status is not `ok`, for example `active` or `error`. The agent sends the same alerts to the email handler and to the Slack handler.

An alert that stays in a non-OK status is included again each time its rule runs again. Emails therefore repeat about once per send window until the alert returns to the `ok` status.

> **Note**: Environment Watch does not send an email when an alert returns to the `ok` status. Only alerts that are in a non-OK status are included in emails.

### Email content

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
| `SMTPPort`          | Optional. The port of the SMTP server. If the setting is empty, port 25 is used.                    |
| `SMTPUserName`      | Optional. The user name used to authenticate with the SMTP server. If the setting is empty, the agent connects without credentials. |
| `SMTPPassword`      | Optional. The password used to authenticate with the SMTP server.                                   |
| `SMTPSSLisRequired` | Specifies whether SSL is used for the SMTP connection. SSL is used when the setting is not defined. |

> **Note**: If `SMTPSSLisRequired` is `False`, the SMTP connection is not encrypted. The email content, including alert names, queries, and links, is sent without encryption. If `SMTPUserName` is also set, the SMTP credentials are sent without encryption and the agent logs a warning. Set `SMTPSSLisRequired` to `True` when you use credentials.

## Configure the email handler

To configure email notifications, locate the `alertNotificationHandlers` section of the custom JSON configuration file and add an `email` handler. For details about the file, see [Custom JSON Configuration](ew-extensibility-configuration-06-json-configuration.md).

The configuration file is located in the `EnvironmentWatch` folder under `BCPPath`:

```text
<BCPPath>\EnvironmentWatch\environment-watch-configuration.json
```

> **Note**: If this is the first custom configuration on this instance, the `EnvironmentWatch` folder and the `environment-watch-configuration.json` file do not exist yet. Create the folder and the file manually. For details about `BCPPath`, including local paths, see [Configuration File Location](ew-extensibility-configuration-06-json-configuration.md#configuration-file-location).

### Email handler properties

| Property                 | Description                                                                                                                                                                                                                |
|--------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `enabled`                | Boolean flag to enable or disable email notifications. The default value is `false`.                                                                                                                                       |
| `fromAddress`            | The sender email address. Always set this property. If it is empty, the `SMTPUserName` Instance Setting is used, and a user name that is not an email address, such as `DOMAIN\svc`, is not a valid sender address. In that case the alerts are not sent and are re-queued.   |
| `fromDisplayName`        | Optional. The sender name shown in the email.                                                                                                                                                                              |
| `recipients`             | The list of people who receive the email. At least one recipient must be enabled. Each recipient has the properties listed in the [recipient properties](#recipient-properties) table.                                      |
| `notifyOnAlertFired`     | Boolean flag. When `true`, an email is sent while an alert is in a non-OK status. When `false`, these alerts are skipped and no email is sent. The default value is `true`.                                                                  |
| `messageIntervalSeconds` | The minimum time, in seconds, between emails. The minimum and default effective value is 180 seconds (3 minutes). A lower value is replaced with 180. See [Batching and cooldown](#batching-and-cooldown).               |
| `maxQueueSize`           | The maximum number of alerts that wait to be sent. The default value is 100. Set a value greater than 0. A value of 0 or less drops every alert. When the limit is exceeded, for example during a long SMTP outage, the oldest alerts are dropped and a warning is logged.                      |

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

The agent checks for emails to send only when at least one alert is in a non-OK status. There is no separate timer that sends queued alerts. If the cooldown window ends while no alert is in a non-OK status, the queued alerts wait until the next time an alert is in a non-OK status. An email can therefore arrive later than the alert that it reports.

The email subject depends on the number of alerts in the email:

| Alerts in the email | Subject                                                |
|---------------------|--------------------------------------------------------|
| One                 | `Relativity Environment Watch Alert: <alert name> (<status>)` |
| More than one       | `Relativity Environment Watch: <N> Alerts`             |

## Apply configuration changes

Changes to `environment-watch-configuration.json` are applied without restarting the Environment Watch agent. The agent reloads the file when it detects that the file changed. If the agent does not detect the change, it reloads the file at the latest 15 minutes after the previous load.

## Reliability

- If the SMTP server cannot be reached or a send fails, the alerts are put back in the queue. The agent tries to send them again the next time it finds an alert in a non-OK status, which is every 3 minutes by default while an alert is active. The failure is logged with the alert names and the number of recipients.
- If the SMTP server rejects a recipient or the message is invalid, the alerts are not re-queued because the same error would occur again. The failure is logged.
- If a recipient address is not valid, that recipient is skipped and the email is sent to the remaining recipients.
- If the number of queued alerts exceeds `maxQueueSize`, the oldest alerts are dropped and a warning is logged. The warning is the only indication that alerts were dropped.

### Behavior when email cannot be sent

| Condition                                              | Behavior                                                                                         |
|--------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| `email` handler is not present or `enabled` is `false` | No email is sent. A debug message is logged.                                                     |
| SMTP is not configured in Instance Settings            | No email is sent. A warning is logged.                                                           |
| No recipient is enabled                                | No email is sent. A warning is logged when the configuration is loaded.                          |
| `fromAddress` and `SMTPUserName` are both empty        | The alerts are re-queued and a warning is logged.                                                |
| `fromAddress` is not valid                             | The alerts are re-queued and a warning is logged.                                                |
| All recipient addresses are not valid                  | The alerts are re-queued and a warning is logged.                                                |
| The SMTP server cannot be reached or a send fails      | The alerts are re-queued and a warning is logged.                                                |
| The SMTP server rejects a recipient or the message     | The alerts are not re-queued and a warning is logged.                                            |

For help with these conditions, see [Email Notifications](../../../troubleshooting/custom-json-troubleshooting.md#email-notifications) in the troubleshooting guide.

## Test email notifications

> **Testing tip**: You do not need a real mailbox to test email notifications in a test or development environment. Use [Papercut SMTP](https://github.com/ChangemakerStudios/Papercut-SMTP) as a local SMTP server that receives the emails and does not deliver them.

> **Note**: Papercut SMTP is a third-party tool that is not provided or supported by Relativity. Use it only for testing.

1. Download Papercut SMTP from the [Papercut SMTP releases page](https://github.com/ChangemakerStudios/Papercut-SMTP/releases). Download the Service zip for your platform, for example `Papercut.Smtp.Service.<version>-win-x64.zip`. Do not download the desktop edition.
2. Extract the zip file to a folder on the test server, for example `C:\Papercut`.
3. Open `appsettings.json` in that folder and set `SmtpServer.Port` to the port that you will use for the `SMTPPort` Instance Setting, for example `25`. You can leave the web interface port (`Urls`, default `8080`) as it is.
4. Open PowerShell as an Administrator and run the install script from that folder. The script creates and starts the `Papercut.Smtp.Service` Windows service with automatic startup.

   ```powershell
   cd C:\Papercut
   .\install-papercut-service.ps1
   ```

5. In Relativity, set the following Instance Settings in the `kCura.Notification` section:

   | Instance Setting    | Value                                                                 |
   |---------------------|-----------------------------------------------------------------------|
   | `SMTPServer`        | `localhost`, or the host name of the server that runs Papercut SMTP.  |
   | `SMTPPort`          | The port that you set in step 3.                                      |
   | `SMTPSSLisRequired` | `False`                                                               |

   You do not need to set a user name or password.
6. Verify that the `Papercut.Smtp.Service` service is running and that it listens on the SMTP port and the web interface port.

   ```powershell
   Get-Service Papercut.Smtp.Service
   Get-NetTCPConnection -State Listen -LocalPort <SMTP port>,<web interface port>
   ```

7. Configure the `email` handler in `environment-watch-configuration.json` with any recipient address, for example `admin@company.com`. The address does not have to exist.
8. Trigger or wait for an Environment Watch alert.
9. Open the Papercut inbox in a browser:

   ```text
   http://localhost:<web interface port>
   ```

   Every email that Environment Watch sends appears in the inbox immediately. Select an email to view it as HTML, the same way a recipient sees it. No mail is sent outside the server.

> **Important**: Use Papercut SMTP only for testing. For a production environment, configure the Instance Settings with your organization's SMTP server.

## Security

- The agent does not log SMTP credentials.
- Dynamic content in the email, such as alert names and queries, is HTML-encoded.

## Notification health alert

Environment Watch monitors its own notification delivery. When sending an alert notification fails with an error, for example because the SMTP server cannot be reached or rejects the credentials, the agent records the `relsvr.notification.send_failure` metric. This metric is recorded for each notification provider, including email and Slack.

The **Environment Watch alert notifications are not being sent** alert fires when the sum of `relsvr.notification.send_failure` is greater than 0 within a 5-minute window. The alert is checked every minute. It does not fire when no notification handler is configured.

> **Note**: The alert covers only sends that fail with an error. Configuration problems do not activate the alert. These problems are reported only as warnings in the agent logs: SMTP not configured in Instance Settings, no enabled recipient, no sender address or an invalid `fromAddress`, and recipient addresses that are all invalid. Alerts that are dropped because the queue exceeds `maxQueueSize` are also reported only as a warning. For the log messages, see [Email Notifications](../../../troubleshooting/custom-json-troubleshooting.md#email-notifications) in the troubleshooting guide.

> **Note**: The alert has no notification actions of its own. It is shown in the Relativity Alerts application and in Kibana. Environment Watch also sends this alert through the configured handlers, so an email about a failing email channel can fail for the same reason. Check the alert status in the Relativity Alerts application or in Kibana.

### Import the alert

The alert is packaged with the Environment Watch installation. The Environment Watch CLI imports it when you run the setup command:

```text
C:\Server.Bundle.x.y.z\relsvr.exe setup
```

### Resolve the alert

When the alert is active, check the following:

- Verify the SMTP host, port, user name, and password in the `kCura.Notification` Instance Settings. If the SMTP credentials were changed on the SMTP server, update the Instance Settings.
- Verify that the SMTP port is open on the firewall and that the SMTP host is reachable from the server that runs the Environment Watch agent.
- Verify the `email` handler in `environment-watch-configuration.json`. For Slack, verify the access token and channel.

After the cause is fixed, the agent sends the queued alerts the next time it finds an alert in a non-OK status, and the alert returns to a recovered state.

## Troubleshooting

Refer to the [Troubleshooting Guide](../../../troubleshooting/custom-json-troubleshooting.md#email-notifications) to resolve email notification issues.
