# Server Bundle Release

This release bundle contains packages that are required to set up or upgrade two optional Relativity Server features:

- **Environment Watch**: Environment Watch is a comprehensive monitoring solution built on the Elastic stack designed for Relativity Server customers, providing deep insights into their system’s health and performance. By leveraging OpenTelemetry, it collects and structures telemetry data (metrics, logs, and traces) within an Elasticsearch cluster hosted in the user’s environment, ensuring data security and control Key features include real-time alerts for system issues, dashboards for in-depth analysis, and log searching via Kibana. The solution seamlessly integrates with Relativity to provide in-app alert notifications, enabling administrators to quickly detect and resolve potential problems. See [here](docs/environment_watch_product_overview.md) for a comprehensive overview of Environment Watch.  
- **Data Grid Audit**: Audit is a Relativity application that you can use to monitor and run reports on audited user activity. You must have Elasticsearch configured to use the Audit application. This release bundle includes a setup package and documentation for configuring Elasticsearch for Data Grid Audit. Additional Audit setup instructions are available [here](https://help.relativity.com/Server2024/Content/Relativity/Audit/Audit.htm#InstallingandconfiguringAudit).

## Release Bundle Artifacts

| Artifact                            | Environment Watch Usage                                                                                         | DataGrid/Audit Usage                                                                  |
|------------------------------------ |-----------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------|
| Relativity.Server.Cli.24.0.xxxx.zip | Used to set up integration between Elasticsearch and Relativity for Environment Watch and import Kibana objects.| Used to set up integration between Elasticsearch and Relativity for Data Grid Audit.  |
| Relativity.EnvironmentWatch.Installer.24.0.xxxx.exe  | Used to monitor your server.                                                                   | N/A                                                                                   |
| Relativity.Alerts.24000.0.xxxx.rap  | Used to view alerts within Relativity Alerts.                                                                   | N/A                                                                                   |

## Getting Started With This Release Bundle
- For installation instructions, see [Environment Watch install guide](docs/environment_watch_installation.md).
- System requirements: 
  - Minimum required Relativity Server version: Server 2024 Patch 1 (24.0.375.2-update.1.0) 
  - If DataGrid/Audit Only:
    - Elasticsearch version: 7.17.x
  - If Environment Watch Only or Both:
    - Elasticsearch/Kibana/APM Server: 8.17.1, 8.17.2, 8.17.3 (versions 8.17.4 and above, including 9.0, have not been tested and certified by Relativity for this release)

A list of common installation issues and their resolutions are available at [Environment Watch troubleshooting guide](docs/environment_watch_troubleshooting.md).