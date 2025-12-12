# Install Other Integrations

![](../resources/caat_environment_watch_setup.png)

> [!NOTE]
> This step is required for Environment Watch.

This guide provides information on integrating various Relativity components with Environment Watch. These integrations extend observability capabilities across your environment, enabling comprehensive monitoring and performance analysis.

## Available Integrations

### Relativity Analytics Engine (CAAT) Integration 

The Relativity Analytics Engine (CAAT) integration provides observability into Analytics operations by implementing OpenTelemetry instrumentation.

- [Setting Up OpenTelemetry Java Agent for Relativity Analytics Engine](analytics/caat_environment_watch_setup.md)

### RabbitMQ Integration

The RabbitMQ integration enables monitoring of RabbitMQ queues, exchanges, and nodes.

- [Setting Up RabbitMQ Integration](rabbitmq/rabbitmq_integration.md)

### Windows Services Using Custom-JSON Integration

The Custom-JSON integration enables monitoring of Windows services, allowing you to track their status and performance.

- [Setting Up Windows Service Custom-JSON Integration](custom-json-configuration/windows_services_configuration.md)

### Certificates Using Custom-JSON Integration

The Custom-JSON integration enables monitoring of certificate expiration dates, allowing you to ensure timely renewals, monitoring of certificate status in Kibana alerts/dashboard and maintain system security.

- [Setting Up Certificate Custom-JSON Integration](custom-json-configuration/certificates_configuration.md)

### Kibana Alerts - Slack Notification Using Custom-JSON Integration

The Custom-JSON integration enables monitoring of Kibana alerts and sending notifications to Slack channels.

- [Setting Up Kibana Alerts - Slack Notification Custom-JSON Integration](custom-json-configuration/alert_notification_handlers_configuration.md)

### SQL Cluster Instance Using Custom-JSON Integration

The Custom-JSON integration enables monitoring of SQL Cluster instances, allowing to receive metrics and performance data from SQL clusters instances in Kibana dashboards.

- [Setting Up SQL Cluster Instance Custom-JSON Integration](sql-cluster-configuration/sql-cluster-configuration.md)

## Next Step

[Click here for the next step](./environment-watch/post-install-verification.md)