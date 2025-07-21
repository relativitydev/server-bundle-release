# Troubleshooting Guide

This document provides quick reference links to detailed troubleshooting guides for all components in the Relativity Server Bundle environment.

## Component Troubleshooting Guides

### [Elasticsearch Troubleshooting →](troubleshooting/elasticsearch.md)
- Windows service issues
- Port configuration (9200, 9300)
- Memory allocation
- Authentication and API keys
- Service verification

### [Kibana Troubleshooting →](troubleshooting/kibana.md)
- Windows service issues
- Port configuration (5601)
- Memory management
- Authentication and encryption keys
- Service verification

### [APM Server Troubleshooting →](troubleshooting/apm-server.md)
- Windows service issues
- Port configuration (8200)
- Authentication and API keys
- Service verification

### [OpenTelemetry Collector Troubleshooting →](troubleshooting/otel-collector.md)
- Windows service issues
- Port configuration (4317, 4318, 13133)
- Error handling and diagnostics
- Prerequisite access verification

### [Relativity Server CLI Troubleshooting →](troubleshooting/relativity-server-cli.md)
- Elastic APM integration package
- Data view configuration
- Kibana encryption keys
- Prerequisite access verification

### [Relativity Alerts Troubleshooting →](troubleshooting/relativity_alerts_troubleshooting.md)
- Installation issues
- Environment Watch dependencies
- Alert Manager agent configuration
- Post-installation troubleshooting

## Quick Health Check

```powershell
# Check all services
Get-Service -Name elasticsearch, kibana, apm-server, otelcol | Format-Table -AutoSize

# Test connectivity to all components
Test-NetConnection localhost -Port 9200  # Elasticsearch
Test-NetConnection localhost -Port 5601  # Kibana
Test-NetConnection localhost -Port 8200  # APM Server
Test-NetConnection localhost -Port 4317  # OpenTelemetry gRPC
Test-NetConnection localhost -Port 13133 # OpenTelemetry Health
```
