# Troubleshooting Guide

This document provides quick reference links to detailed troubleshooting guides for all components in the Relativity Server Bundle environment.

## Component Troubleshooting Guides

### [Elasticsearch Troubleshooting →](troubleshooting/elasticsearch.md)

### [Kibana Troubleshooting →](troubleshooting/kibana.md)

### [APM Server Troubleshooting →](troubleshooting/apm-server.md)

### [OpenTelemetry Collector Troubleshooting →](troubleshooting/otel-collector.md)

### [Relativity Server CLI Troubleshooting →](troubleshooting/relativity-server-cli.md)

### [Relativity Alerts Troubleshooting →](troubleshooting/relativity_alerts_troubleshooting.md)

## Quick Health Check

```powershell
# Check all services
Get-Service -Name elasticsearch, kibana, apm-server, otelcol | Format-Table -AutoSize

# Test connectivity using curl commands
curl -X GET "localhost:9200/"     # Elasticsearch
curl -X GET "localhost:5601/"     # Kibana  
curl -X GET "localhost:8200/"     # APM Server
curl -X GET "localhost:13133/"    # OpenTelemetry Health
```
