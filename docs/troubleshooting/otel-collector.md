# OpenTelemetry Collector Troubleshooting

This document provides troubleshooting guidance for common OpenTelemetry Collector issues encountered during installation, configuration, and operation in Relativity environments.

> [!NOTE]
> This guide assumes a default OpenTelemetry Collector installation path of `C:\otel\otelcol`. Adjust paths according to your actual installation directory.

## Table of Contents

- [Windows Service Issues](#windows-service-issues)
  - [OpenTelemetry Collector Service Not Starting](#opentelemetry-collector-service-not-starting)
  - [Service Crashes or Stops Unexpectedly](#service-crashes-or-stops-unexpectedly)
  - [Permission and Access Issues](#permission-and-access-issues)
- [Port Configuration Issues](#port-configuration-issues)
  - [Port Conflicts](#port-conflicts)
  - [Network Connectivity Problems](#network-connectivity-problems)
- [Error Handling and Diagnostics](#error-handling-and-diagnostics)
  - [Windows Event Viewer Analysis](#windows-event-viewer-analysis)
  - [Standard Output and Error Stream Analysis](#standard-output-and-error-stream-analysis)
  - [Configuration Validation and Testing](#configuration-validation-and-testing)
- [Prerequisite Access Verification](#prerequisite-access-verification)
  - [BCP Share Access](#bcp-share-access)
  - [Secret Server Access](#secret-server-access)
  - [Kepler SSL Certificate Access](#kepler-ssl-certificate-access)
  - [Elasticsearch Access Verification](#elasticsearch-access-verification)
  - [Kibana Access Verification](#kibana-access-verification)
  - [APM Server Access Verification](#apm-server-access-verification)
  - [Relativity Service Account Verification](#relativity-service-account-verification)
- [Additional Diagnostic Commands](#additional-diagnostic-commands)

## Windows Service Issues

### OpenTelemetry Collector Service Not Starting

**Symptoms:**
- Relativity Environment Watch service fails to start
- Service stops immediately after starting
- Error messages in Relativity Environment Watch logs

**Troubleshooting Steps:**

1. **Check Service Status:**
   ```powershell
   Get-Service -Name Relativity Environment Watch
   ```

2. **Start Service Manually:**
   ```powershell
   Start-Service Relativity Environment Watch
   ```

### Service Crashes or Stops Unexpectedly

**Symptoms:**
- Relativity Environment Watch service starts but stops after a short period
- Service status shows "Stopped" unexpectedly
- Data collection stops working

**Troubleshooting Steps:**

1. **Check Windows Event Logs:**
   - Open Event Viewer
   - Navigate to Windows Logs > Application
   - Look for Relativity Environment Watch-related error events

2. **Review Configuration File:**
   - Navigate to `C:\ProgramData\Relativity\EnvironmentWatch\Services\InfraWatchAgent` and Check `otelcol-config-auto-generated.yaml` file for syntax errors ()
   - Verify all required sections (receivers, processors, exporters, service)
   - Ensure proper indentation and YAML format

3. **Validate Configuration:**
   ```powershell
   .\otelcol-relativity.exe --config-validate --config=config.yaml
   ```

4. **Check Resource Usage:**
   - Monitor CPU and memory consumption
   - Verify sufficient disk space for logs and temporary data
   - Check for any resource constraints

### Permission and Access Issues

**Symptoms:**
- Access denied errors when starting service
- Unable to write to log directories
- Configuration file access errors

**Troubleshooting Steps:**

1. **Verify Service Account Permissions:**
   - Ensure service account has read access to OpenTelemetry Collector directory
   - Verify write permissions to logs directory
   - Check permissions on configuration files

2. **Grant Required Permissions:**
   ```powershell
   # Grant permissions to OpenTelemetry Collector directory
   icacls "C:\otelcol" /grant "NT SERVICE\otelcol-relativity:(OI)(CI)F" /T
   ```

3. **Check User Rights Assignment:**
   - Open Local Security Policy (secpol.msc)
   - Navigate to User Rights Assignment
   - Verify "Log on as a service" right for service account

## Port Configuration Issues

### Port Conflicts

**Symptoms:**
- OpenTelemetry Collector fails to bind to configured ports
- "bind: address already in use" errors in logs
- Applications cannot send telemetry data

**Troubleshooting Steps:**

1. **Check Default Ports:**
   - OTLP gRPC: 4317
   - OTLP HTTP: 4318
   - Health check: 13133
   - Verify port availability:
   ```powershell
   netstat -an | findstr ":4317"
   netstat -an | findstr ":4318"
   netstat -an | findstr ":13133"
   ```

2. **Identify Port Conflicts:**
   ```powershell
   Get-NetTCPConnection -LocalPort 4317 -State Listen
   Get-NetTCPConnection -LocalPort 4318 -State Listen
   Get-NetTCPConnection -LocalPort 13133 -State Listen
   ```

3. **Configure Alternative Ports:**
   - Edit `otelcol-config-auto-generated.yaml`:
   ```yaml
   receivers:
     otlp:
       protocols:
         grpc:
           endpoint: 0.0.0.0:4319
         http:
           endpoint: 0.0.0.0:4320
   
   extensions:
     health_check:
       endpoint: 0.0.0.0:13134
   ```

4. **Update Firewall Rules:**
   ```powershell
   New-NetFirewallRule -DisplayName "OpenTelemetry OTLP gRPC" -Direction Inbound -Protocol TCP -LocalPort 4317 -Action Allow
   New-NetFirewallRule -DisplayName "OpenTelemetry OTLP HTTP" -Direction Inbound -Protocol TCP -LocalPort 4318 -Action Allow
   New-NetFirewallRule -DisplayName "OpenTelemetry Health Check" -Direction Inbound -Protocol TCP -LocalPort 13133 -Action Allow
   ```

### Network Connectivity Problems

**Symptoms:**
- Applications cannot connect to OpenTelemetry Collector
- Connection timeouts from telemetry sources
- "connection refused" errors in application logs

**Troubleshooting Steps:**

1. **Verify Network Binding:**
   - Check `config.yaml` configuration:
   ```yaml
   receivers:
     otlp:
       protocols:
         grpc:
           endpoint: 0.0.0.0:4317  # Listen on all interfaces
         http:
           endpoint: 0.0.0.0:4318
   ```

2. **Test Local Connectivity:**
   ```powershell
   Test-NetConnection -ComputerName localhost -Port 4317
   Test-NetConnection -ComputerName localhost -Port 4318
   ```

3. **Test Remote Connectivity:**
   ```powershell
   Test-NetConnection -ComputerName your-otelcol-server -Port 4317
   ```

4. **Verify Health Check Endpoint:**
   ```powershell
   Invoke-WebRequest -Uri "http://localhost:13133/" -Method GET
   ```

## Error Handling and Diagnostics

### Windows Event Viewer Analysis

**Symptoms:**
- Need to diagnose service startup failures
- Understanding error patterns
- Monitoring service health

**Troubleshooting Steps:**

1. **Check Application Event Log:**
   ```powershell
   Get-WinEvent -FilterHashtable @{LogName='Application'; ProviderName='OpenTelemetry Collector'} -MaxEvents 50
   ```

2. **Search for Specific Error Patterns:**
   ```powershell
   Get-WinEvent -FilterHashtable @{LogName='Application'; Level=2} | Where-Object {$_.Message -like "*otelcol-relativity*"}
   ```

3. **Monitor Real-time Events:**
   ```powershell
   Get-WinEvent -FilterHashtable @{LogName='Application'} -MaxEvents 1 | Select-Object TimeCreated, Id, LevelDisplayName, Message
   ```

4. **Export Event Log for Analysis:**
   ```powershell
   Get-WinEvent -FilterHashtable @{LogName='Application'; ProviderName='OpenTelemetry Collector'} | Export-Csv -Path "otelcol-events.csv"
   ```

### Standard Output and Error Stream Analysis

**Symptoms:**
- Configuration validation errors
- Runtime processing errors
- Performance issues

**Troubleshooting Steps:**

1. **Run with Verbose Logging:**
   ```powershell
   .\otelcol-relativity.exe --config=config.yaml --log-level=debug
   ```

2. **Capture Output to File:**
   ```powershell
   .\otelcol-relativity.exe --config=config.yaml 2>&1 | Tee-Object -FilePath "otelcol-output.log"
   ```

3. **Monitor Specific Log Levels:**
   ```powershell
   .\otelcol-relativity.exe --config=config.yaml --log-level=warn
   ```

4. **Enable Structured Logging:**
   ```yaml
   service:
     telemetry:
       logs:
         level: debug
         development: true
         encoding: json
   ```

### Configuration Validation and Testing

**Symptoms:**
- YAML syntax errors
- Invalid receiver/processor/exporter configurations
- Pipeline configuration issues

**Troubleshooting Steps:**

1. **Validate Configuration Syntax:**
   ```powershell
   .\otelcol-relativity.exe --config-validate --config=config.yaml
   ```

2. **Test Configuration with Dry Run:**
   ```powershell
   .\otelcol-relativity.exe --config=config.yaml --dry-run
   ```

3. **Check Individual Components:**
   ```yaml
   # Test minimal configuration
   receivers:
     otlp:
       protocols:
         grpc:
           endpoint: 0.0.0.0:4317
   
   exporters:
     logging:
       loglevel: debug
   
   service:
     pipelines:
       traces:
         receivers: [otlp]
         exporters: [logging]
   ```

## Prerequisite Access Verification

### BCP Share Access

**Symptoms:**
- Cannot access required BCP share for data processing
- Authentication failures when connecting to share
- File access denied errors

**Troubleshooting Steps:**

1. **Test BCP Share Connectivity:**
   ```powershell
   Test-Path "\\bcp-share\relativity-data"
   ```

2. **Verify Credentials:**
   ```powershell
   # Test with explicit credentials
   $credential = Get-Credential
   New-PSDrive -Name "BCPTest" -PSProvider FileSystem -Root "\\bcp-share\relativity-data" -Credential $credential
   ```

3. **Check Service Account Access:**
   ```powershell
   # Run as service account
   runas /user:domain\otelcol-service "powershell -command Test-Path \\bcp-share\relativity-data"
   ```

4. **Configure BCP Share in OpenTelemetry:**
   ```yaml
   processors:
     filelog:
       include:
         - "\\\\bcp-share\\relativity-data\\*.log"
       operators:
         - type: regex_parser
           regex: '^(?P<timestamp>\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2})'
   ```

### Secret Server Access

**Symptoms:**
- Cannot retrieve secrets from Secret Server
- Authentication failures with Secret Server API
- Missing or expired credentials

**Troubleshooting Steps:**

1. **Test Secret Server Connectivity:**
   ```powershell
   Test-NetConnection -ComputerName secret-server.domain.com -Port 443
   ```

2. **Verify API Access:**
   ```powershell
   $headers = @{
     'Authorization' = 'Bearer your-token'
     'Content-Type' = 'application/json'
   }
   Invoke-RestMethod -Uri "https://secret-server.domain.com/api/v1/secrets" -Headers $headers -Method GET
   ```

3. **Configure Secret Server Integration:**
   ```yaml
   extensions:
     file_storage:
       directory: C:\otelcol\secrets
   
   processors:
     attributes:
       actions:
         - key: secret.password
           action: insert
           from_attribute: env.SECRET_PASSWORD
   ```

4. **Test Secret Retrieval:**
   ```powershell
   # Test environment variable setup
   $env:SECRET_PASSWORD = (Get-Secret -Name "RelativityPassword").SecretValueText
   ```

### Kepler SSL Certificate Access

**Symptoms:**
- SSL certificate validation errors
- Cannot connect to Kepler endpoints
- Certificate chain validation failures

**Troubleshooting Steps:**

1. **Verify Certificate Store Access:**
   ```powershell
   Get-ChildItem -Path Cert:\LocalMachine\My | Where-Object {$_.Subject -like "*kepler*"}
   ```

2. **Test Kepler Connectivity:**
   ```powershell
   Test-NetConnection -ComputerName kepler.domain.com -Port 443
   ```

3. **Validate Certificate Chain:**
   ```powershell
   $cert = Get-ChildItem -Path Cert:\LocalMachine\My | Where-Object {$_.Subject -like "*kepler*"}
   Test-Certificate -Cert $cert -AllowUntrustedRoot
   ```

4. **Configure SSL in OpenTelemetry:**
   ```yaml
   exporters:
     otlphttp/kepler:
       endpoint: https://kepler.domain.com:4318
       tls:
         cert_file: C:\certs\kepler-client.crt
         key_file: C:\certs\kepler-client.key
         ca_file: C:\certs\kepler-ca.crt
   ```

### Elasticsearch Access Verification

**Symptoms:**
- Cannot connect to Elasticsearch cluster
- Authentication failures with Elasticsearch
- Data export failures

**Troubleshooting Steps:**

1. **Test Elasticsearch Connectivity:**
   ```powershell
   Test-NetConnection -ComputerName elasticsearch.domain.com -Port 9200
   ```

2. **Verify Authentication:**
   ```powershell
   $headers = @{
     'Authorization' = 'ApiKey your-api-key'
     'Content-Type' = 'application/json'
   }
   Invoke-RestMethod -Uri "https://elasticsearch.domain.com:9200/_cluster/health" -Headers $headers
   ```

3. **Configure Elasticsearch Exporter:**
   ```yaml
   exporters:
     elasticsearch:
       endpoints: ["https://elasticsearch.domain.com:9200"]
       api_key: "your-api-key"
       index: "otel-traces-%{+yyyy.MM.dd}"
       tls:
         insecure_skip_verify: false
   ```

4. **Test Data Export:**
   ```powershell
   # Send test data
   curl -X POST "http://localhost:4318/v1/traces" -H "Content-Type: application/json" -d '{"resourceSpans":[]}'
   ```

### Kibana Access Verification

**Symptoms:**
- Cannot access Kibana dashboards
- Visualization setup failures
- Index pattern creation issues

**Troubleshooting Steps:**

1. **Test Kibana Connectivity:**
   ```powershell
   Test-NetConnection -ComputerName kibana.domain.com -Port 5601
   ```

2. **Verify Kibana API Access:**
   ```powershell
   Invoke-WebRequest -Uri "https://kibana.domain.com:5601/api/status" -Method GET
   ```

3. **Check Index Patterns:**
   ```powershell
   $headers = @{
     'kbn-xsrf' = 'true'
     'Content-Type' = 'application/json'
   }
   Invoke-RestMethod -Uri "https://kibana.domain.com:5601/api/saved_objects/_find?type=index-pattern" -Headers $headers
   ```

4. **Configure Kibana Integration:**
   ```yaml
   processors:
     resource:
       attributes:
         - key: service.name
           value: "relativity-otelcol"
           action: upsert
   ```

### APM Server Access Verification

**Symptoms:**
- Cannot send APM data to APM Server
- Connection failures to APM endpoints
- APM data not appearing in Elasticsearch

**Troubleshooting Steps:**

1. **Test APM Server Connectivity:**
   ```powershell
   Test-NetConnection -ComputerName apm-server.domain.com -Port 8200
   ```

2. **Verify APM Server Status:**
   ```powershell
   Invoke-WebRequest -Uri "https://apm-server.domain.com:8200/" -Method GET
   ```

3. **Configure APM Exporter:**
   ```yaml
   exporters:
     otlphttp/apm:
       endpoint: https://apm-server.domain.com:8200
       headers:
         authorization: "Bearer your-secret-token"
   ```

4. **Test APM Data Transmission:**
   ```powershell
   curl -X POST "https://apm-server.domain.com:8200/intake/v2/events" -H "Authorization: Bearer your-secret-token"
   ```

### Relativity Service Account Verification

**Symptoms:**
- Service account authentication failures
- Insufficient permissions for required operations
- Cannot access Relativity resources

**Troubleshooting Steps:**

1. **Verify Service Account Status:**
   ```powershell
   Get-ADUser -Identity "relativity-otelcol" -Properties AccountExpirationDate, PasswordLastSet, Enabled
   ```

2. **Test Service Account Authentication:**
   ```powershell
   $credential = Get-Credential -UserName "domain\relativity-otelcol"
   Test-Connection -ComputerName relativity-server.domain.com -Credential $credential
   ```

3. **Check Required Permissions:**
   ```powershell
   # Verify group memberships
   Get-ADUser -Identity "relativity-otelcol" -Properties MemberOf | Select-Object -ExpandProperty MemberOf
   ```

4. **Configure Service with Service Account:**
   ```powershell
   # Set service to run as specific account
   $service = Get-WmiObject -Class Win32_Service -Filter "Name='otelcol-relativity'"
   $service.Change($null,$null,$null,$null,$null,$null,"domain\relativity-otelcol","password")
   ```

## Additional Diagnostic Commands

### General Health Checks

```powershell
# Check service status
Get-Service otelcol-relativity | Select-Object Status, StartType, Name

# Check process information
Get-Process -Name otelcol-relativity | Select-Object Id, ProcessName, CPU, WorkingSet

# Check listening ports
Get-NetTCPConnection | Where-Object {$_.LocalPort -in @(4317,4318,13133)} | Select-Object LocalAddress, LocalPort, State

# Validate configuration
.\otelcol-relativity.exe --config-validate --config=config.yaml

# Test health endpoint
Invoke-WebRequest -Uri "http://localhost:13133/" -Method GET
```

### Configuration Testing

```powershell
# Test minimal configuration
.\otelcol-relativity.exe --config=minimal-config.yaml --dry-run

# Validate YAML syntax
Get-Content config.yaml | ConvertFrom-Yaml

# Check component availability
.\otelcol-relativity.exe --help | Select-String "receivers|processors|exporters"
```

### Network Diagnostics

```powershell
# Test all prerequisite connections
$endpoints = @(
    @{Host="bcp-share"; Port=445},
    @{Host="secret-server.domain.com"; Port=443},
    @{Host="kepler.domain.com"; Port=443},
    @{Host="elasticsearch.domain.com"; Port=9200},
    @{Host="kibana.domain.com"; Port=5601},
    @{Host="apm-server.domain.com"; Port=8200}
)

foreach ($endpoint in $endpoints) {
    Test-NetConnection -ComputerName $endpoint.Host -Port $endpoint.Port
}
```
````

