
# Server Bundle Release

This bundle provides packages to set up or upgrade two optional Relativity Server features:

- **Environment Watch**: Monitoring solution for Relativity Server, built on the Elastic stack. Collects metrics, logs, and traces using OpenTelemetry, stores data in your own Elasticsearch cluster, and provides real-time alerts and dashboards. [Click here to learn more about Environment Watch](docs/environment_watch_product_overview.md)
- **Data Grid Audit**: Application for monitoring and reporting on audited user activity. Requires Elasticsearch. [Click here to learn more about Data Grid Audit and setup instructions](https://help.relativity.com/Server2024/Content/Relativity/Audit/Audit.htm#InstallingandconfiguringAudit)

## System Requirements

- Minimum Relativity Server version: Server 2024 Patch 2
- ElasticStack version: **8.17.3** (Required for both Environment Watch and Data Grid Audit)

## Getting Started

- [Environment Watch and Data Grid Audit Installation](docs/environment_watch_installation.md)
- [Environment Watch Product Overview](docs/environment_watch_product_overview.md)
- [Troubleshooting Guide](/docs/troubleshooting.md)
