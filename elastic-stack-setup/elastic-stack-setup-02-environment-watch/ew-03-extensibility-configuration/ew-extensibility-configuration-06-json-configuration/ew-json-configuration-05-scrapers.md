# Scrapers Configuration

This section describes how to configure scraper intervals and parameters using the `environmentWatchConfiguration` JSON configuration file.

---

## Overview

Scrapers are background tasks that collect metrics from various sources and report them to OpenTelemetry collectors. Each scraper runs at a configurable interval and monitors specific aspects of the environment. By default, all scrapers use predefined interval values, but these can be customized using the JSON configuration file.

The `scrapers` section in the configuration allows users to override the default interval for each scraper, and in some cases, configure additional parameters specific to each scraper type.

---

## Scrapers Section

The `scrapers` configuration is defined under the `monitoring` section in the JSON configuration file.

### Properties Table

| Property     | Type   | Description                                                                 |
|--------------|--------|-----------------------------------------------------------------------------|
| `name`       | string | The name of the scraper. Must match the exact scraper class name.            |
| `parameters` | object | Key-value pairs of parameters for the scraper (e.g., `intervalSeconds`).     |

### Common Parameters

| Parameter          | Type   | Description                                                                 |
|--------------------|--------|-----------------------------------------------------------------------------|
| `intervalSeconds`  | string | The interval (in seconds) at which the scraper runs. Value must be a string.|

---

## Available Scrapers

The following table lists all available scrapers, their default intervals, descriptions, and KQL queries for verification:

| Scraper Name                                 | Default Interval (seconds) | Description                                                         | KQL Query                                                |
|----------------------------------------------|----------------------------|---------------------------------------------------------------------|----------------------------------------------------------|
| `HeartbeatOtelMetricScraper`                 | 30                         | Monitors the heartbeat and uptime of the Environment Watch service. | transaction.name : "HeartbeatOtelMetricScraper.ScrapeMetricsAsync" |
| `ResourceServerNetworkFileShareOtelMetricScraper` | 120                   | Monitors accessibility of resource server file shares.              | relsvr.resourceserver_networkshare.accessible : * |
| `SqlDistributedHealthOtelMetricScraper`      | 300                        | Monitors health and connectivity of distributed SQL servers.        | relsvr.sqlserver_distributed.running : * |
| `SqlPrimaryBcpShareOtelMetricScraper`        | 120                        | Monitors accessibility of the primary SQL BCP file share.           | relsvr.sqlbcp_networkshare.accessible : * |
| `SqlPrimaryHealthOtelMetricScraper`          | 60                         | Monitors health and connectivity of the primary SQL server.         | relsvr.sqlserver_primary.running : *    |
| `WindowsServiceOtelMetricScraper`            | 60                         | Monitors the status of Windows services.                            | host.os.service.running : *             |
| `X509CertificateOtelMetricScraper`           | 3600                       | Monitors the presence and validity of X.509 certificates.           | relsvr.x509_certificate : *             |
| `OAuth2ClientOtelMetricScraper`              | 180                        | Monitors OAuth2 client authentication health.                       | transaction.name : "OAuth2ClientOtelMetricScraper.ScrapeMetricsAsync" |
| `KibanaAlertOtelMetricScraper`               | 180                        | Monitors Kibana alerts and sends notifications.                     | relsvr.alert : *                        |
| `AntiVirusOtelMetricScraper`                 | 300                        | Monitors antivirus status on the host.                              | relsvr.antivirus : *                    |

---

## Scraper-Specific Parameters

### X509CertificateOtelMetricScraper

In addition to `intervalSeconds`, the X509 certificate scraper supports the following parameter:

| Parameter              | Type   | Default | Description                                                                 |
|------------------------|--------|---------|-----------------------------------------------------------------------------|
| `expirationWarningDays`| string | 30      | Number of days before certificate expiration to trigger a warning alert.    |

**Example:**
```json
{
  "name": "X509CertificateOtelMetricScraper",
  "parameters": {
    "intervalSeconds": "3600",
    "expirationWarningDays": "30"
  }
}
```

---

## Configure Scrapers

To configure scrapers, add or update the `scrapers` array under the `monitoring` section in the custom JSON configuration file.

**Example Configuration:**

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
      "installedProducts": [],
      "hosts": [],
      "scrapers": [
        {
          "name": "HeartbeatOtelMetricScraper",
          "parameters": {
            "intervalSeconds": "30"
          }
        },
        {
          "name": "ResourceServerNetworkFileShareOtelMetricScraper",
          "parameters": {
            "intervalSeconds": "120"
          }
        },
        {
          "name": "SqlDistributedHealthOtelMetricScraper",
          "parameters": {
            "intervalSeconds": "300"
          }
        },
        {
          "name": "SqlPrimaryBcpShareOtelMetricScraper",
          "parameters": {
            "intervalSeconds": "120"
          }
        },
        {
          "name": "SqlPrimaryHealthOtelMetricScraper",
          "parameters": {
            "intervalSeconds": "60"
          }
        },
        {
          "name": "WindowsServiceOtelMetricScraper",
          "parameters": {
            "intervalSeconds": "60"
          }
        },
        {
          "name": "X509CertificateOtelMetricScraper",
          "parameters": {
            "intervalSeconds": "3600",
            "expirationWarningDays": "30"
          }
        },
        {
          "name": "OAuth2ClientOtelMetricScraper",
          "parameters": {
            "intervalSeconds": "180"
          }
        },
        {
          "name": "KibanaAlertOtelMetricScraper",
          "parameters": {
            "intervalSeconds": "180"
          }
        },
        {
          "name": "AntiVirusOtelMetricScraper",
          "parameters": {
            "intervalSeconds": "300"
          }
        }
      ]
    },
    "alertNotificationHandlers": {}
  }
}
```

---

## Customizing Scraper Intervals

To customize a scraper's interval:

1. Identify the scraper name from the [Available Scrapers](#available-scrapers) table.
2. Add or update the scraper entry in the `scrapers` array.
3. Set the `intervalSeconds` parameter to the desired value (as a string).

**Example 1**: Increase the heartbeat scraper interval to 60 seconds:
```json
{
  "name": "HeartbeatOtelMetricScraper",
  "parameters": {
    "intervalSeconds": "60"
  }
}
```

**Example 2**: Reduce the certificate scraper interval to 30 minutes (1800 seconds) and set expiration warning to 45 days:
```json
{
  "name": "X509CertificateOtelMetricScraper",
  "parameters": {
    "intervalSeconds": "1800",
    "expirationWarningDays": "45"
  }
}
```

**Example 3**: Increase the SQL distributed health check interval to 10 minutes (600 seconds):
```json
{
  "name": "SqlDistributedHealthOtelMetricScraper",
  "parameters": {
    "intervalSeconds": "600"
  }
}
```

---

## Default Behavior

If a scraper is not explicitly configured in the `scrapers` array, it will use its default interval. The configuration supports partial overrides—only scrapers that need customization need to be included in the `scrapers` array.

> [!NOTE]
> Scraper configurations are cached for performance. After updating the `environment-watch-configuration.json` file, the Environment Watch Windows service will automatically read the updated configuration within the cache expiration period (default: 15 minutes).

---

## Restart the Environment Watch Windows Service

After updating the `environment-watch-configuration.json` file with scraper configuration changes, save the file. The Environment Watch service will automatically detect and apply the changes. For immediate application, restart the Environment Watch Windows service.

---

## Verification in Kibana

To verify that scrapers are running at the configured intervals, use Kibana Discover with the APM data view:

1. Navigate to **Kibana** > **Discover**.
2. Select the **APM** data view from the data view dropdown.
3. Use the KQL query from the [Available Scrapers](#available-scrapers) table to filter results for the specific scraper.
4. Verify that metrics are being reported at the configured intervals by checking the timestamps.

**Example Verification Steps:**

### Verifying HeartbeatOtelMetricScraper
1. In Discover, select the **APM** data view.
2. Enter the KQL query: transaction.name : "HeartbeatOtelMetricScraper.ScrapeMetricsAsync"
3. Adjust the time range as needed.
4. Verify that spans are appearing at the configured interval (default: 30 seconds).

### Verifying X509CertificateOtelMetricScraper
1. In Discover, select the **APM** data view.
2. Enter the KQL query: relsvr.x509_certificate : *
3. Verify that metrics are appearing at the configured interval (default: 3600 seconds).
4. Check the `expiration_warning_days` attribute to confirm the configured threshold.

### Verifying WindowsServiceOtelMetricScraper
1. In Discover, select the **APM** data view.
2. Enter the KQL query: host.os.service.running : *
3. Verify that Windows service metrics are being reported at the configured interval.

---

## Troubleshooting

### Scraper Not Running at Expected Interval

1. Verify the scraper name matches exactly as listed in the [Available Scrapers](#available-scrapers) table (case-sensitive).
2. Ensure the `intervalSeconds` value is provided as a string (e.g., `"60"`, not `60`).
3. Check the Environment Watch service logs for any configuration parsing errors.

### Certificate Expiration Warnings Not Appearing

1. Verify the `expirationWarningDays` parameter is set correctly.
2. Ensure certificates are within the warning threshold.
3. Check that the certificate scraper is enabled and running.

Refer to the [Troubleshooting Guide](../../../troubleshooting/custom-json-troubleshooting.md) to resolve any custom JSON scraper configuration issues.
