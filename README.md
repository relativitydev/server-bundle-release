
# Server Bundle Release

This bundle provides packages to set up or upgrade two optional Relativity Server features:

- **Environment Watch**: Environment Watch is a monitoring solution for Relativity Server built on the Elastic stack. Using OpenTelemetry, it captures metrics, logs, and traces into a customer-hosted Elasticsearch cluster, ensuring data security and control. Key features include real-time alerts, dashboards, log search in Kibana, and in-app notifications within Relativity to help admins quickly detect and resolve issues. [Click here to learn more about Environment Watch](docs/environment_watch_product_overview.md)
  
- **Data Grid Audit**: Audit is a Relativity application for monitoring and reporting on user activity, requiring Elasticsearch. This bundle provides the setup package and configuration docs for Data Grid Audit. [Click here to learn more about Data Grid Audit and setup instructions](https://help.relativity.com/Server2024/Content/Relativity/Audit/Audit.htm#InstallingandconfiguringAudit)

## System Requirements

| Requirement                       | Version/Details                                           |
| --------------------------------- | --------------------------------------------------------- |
| Minimum Relativity Server version | Server 2024 Patch 2                                       |
| ElasticStack version              | **8.17.3** (Required for both Environment Watch and Data Grid Audit) |

## Getting Started

- [Environment Watch and Data Grid Audit Installation](docs/environment_watch_installation.md)
- [Environment Watch Product Overview](docs/environment_watch_product_overview.md)
- [Troubleshooting Guide](/docs/troubleshooting.md)
