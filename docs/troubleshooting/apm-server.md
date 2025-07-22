# APM Server Troubleshooting

This document provides troubleshooting guidance for common APM Server issues encountered during installation, configuration, and operation in Relativity environments.

> [!NOTE]
> This guide assumes a default APM Server installation path of `C:\elastic\apm-server`. Adjust paths according to your actual installation directory.

## Table of Contents

- [Windows Service Issues](#windows-service-issues)
  - [Issue 1: APM Server Service Not Starting](#issue-1-apm-server-service-not-starting)
  - [Issue 2: Service Crashes or Stops Unexpectedly](#issue-2-service-crashes-or-stops-unexpectedly)
  - [Issue 3: Permission and Access Issues](#issue-3-permission-and-access-issues)
- [Port Configuration Issues](#port-configuration-issues)
  - [Issue 4: Port Conflicts](#issue-4-port-conflicts)
  - [Issue 5: Network Connectivity Problems](#issue-5-network-connectivity-problems)
  - [Issue 6: SSL/TLS Configuration Issues](#issue-6-ssltls-configuration-issues)
- [Authentication Issues](#authentication-issues)
  - [Issue 7: API Key Authentication Problems](#issue-7-api-key-authentication-problems)
  - [Issue 8: Username/Password Authentication Issues](#issue-8-usernamepassword-authentication-issues)
  - [Issue 9: APM Agent Authentication](#issue-9-apm-agent-authentication)
- [Service Verification](#service-verification)
  - [Issue 10: Verifying APM Server Health and Status](#issue-10-verifying-apm-server-health-and-status)
  - [Issue 11: Data Flow Verification](#issue-11-data-flow-verification)
  - [Issue 12: Performance and Resource Monitoring](#issue-12-performance-and-resource-monitoring)
- [Additional Diagnostic Commands](#additional-diagnostic-commands)

## Windows Service Issues

### Issue 1: APM Server Service Not Starting

**Symptoms:**
- APM Server service fails to start
- Service stops immediately after starting
- Error messages in APM Server logs

**Troubleshooting Steps:**

1. **Check Service Status:**
   ```powershell
   Get-Service -Name apm-server
   ```

2. **Verify Service Configuration:**
   - Open Services.msc
   - Locate "APM Server" service
   - Verify the service is running under Local System account (default configuration)

3. **Check APM Server Logs:**
   - Navigate to `C:\elastic\apm-server\logs\`
   - Review the latest log files (`apm-server.log`) for error messages
   - Look for configuration errors or connection issues with Elasticsearch

4. **Verify Configuration File:**
   ```powershell
   # Test configuration syntax (navigate to APM Server directory first)
   cd "C:\elastic\apm-server"
   .\apm-server.exe test config -c apm-server.yml
   ```

5. **Start Service Manually:**
   ```powershell
   Start-Service apm-server
   ```

6. **Run APM Server in Foreground for Debugging:**
   ```powershell
   # Navigate to APM Server directory
   .\apm-server.exe -e -c apm-server.yml
   ```

### Issue 2: Service Crashes or Stops Unexpectedly

**Symptoms:**
- APM Server service starts but stops after a short period
- Service status shows "Stopped" unexpectedly
- APM data collection stops working

**Troubleshooting Steps:**

1. **Check Windows Event Logs:**
   - Open Event Viewer
   - Navigate to Windows Logs > Application
   - Look for APM Server-related error events

2. **Review APM Server Configuration:**
   - Check `apm-server.yml` file in `%APM_SERVER_HOME%`
   - Verify Elasticsearch connection settings
   - Ensure all required configuration parameters are present

3. **Verify Elasticsearch Connectivity:**
   ```powershell
   Test-NetConnection -ComputerName your-elasticsearch-server -Port 9200
   ```

4. **Check Resource Usage:**
   - Monitor CPU and memory consumption
   - Verify sufficient disk space for logs and data
   - Check for any resource constraints

5. **Validate Output Configuration:**
   ```yaml
   output.elasticsearch:
     hosts: ["https://elasticsearch-server:9200"]
     protocol: "https"
     username: "apm_system"
     password: "your-password"
   ```

### Issue 3: Permission and Access Issues

**Symptoms:**
- Access denied errors when starting service
- Unable to write to log directories
- Configuration file access errors

**Troubleshooting Steps:**

1. **Verify Service Account Permissions:**
   - Ensure service account has read access to APM Server directory
   - Verify write permissions to logs directory
   - Check permissions on configuration files

2. **Grant Required Permissions:**
   ```powershell
   # Grant permissions to APM Server directory
   icacls "C:\apm-server" /grant "NT SERVICE\apm-server:(OI)(CI)F" /T
   ```

3. **Check User Rights Assignment:**
   - Open Local Security Policy (secpol.msc)
   - Navigate to User Rights Assignment
   - Verify "Log on as a service" right for service account

## Port Configuration Issues

### Issue 4: Port Conflicts

**Symptoms:**
- APM Server fails to bind to default port
- "bind: address already in use" errors in logs
- APM agents cannot connect to server

**Troubleshooting Steps:**

1. **Check Default Port:**
   - Default APM Server port: 8200
   - Verify port availability:
   ```powershell
   netstat -an | findstr ":8200"
   ```

2. **Identify Port Conflicts:**
   ```powershell
   Get-NetTCPConnection -LocalPort 8200 -State Listen
   ```

3. **Configure Alternative Port:**
   - Edit `apm-server.yml`:
   ```yaml
   apm-server:
     host: "0.0.0.0:8201"
   ```

4. **Update Firewall Rules:**
   ```powershell
   New-NetFirewallRule -DisplayName "APM Server" -Direction Inbound -Protocol TCP -LocalPort 8200 -Action Allow
   ```

5. **Update APM Agent Configuration:**
   - Update application configuration to use new port
   - Restart applications using APM agents

### Issue 5: Network Connectivity Problems

**Symptoms:**
- APM agents cannot connect to server
- Connection timeouts from applications
- "connection refused" errors in agent logs

**Troubleshooting Steps:**

1. **Verify Network Binding:**
   - Check `apm-server.yml` configuration:
   ```yaml
   apm-server:
     host: "0.0.0.0:8200"  # Listen on all interfaces
     # or
     host: "your-server-ip:8200"
   ```

2. **Test Local Connectivity:**
   ```powershell
   Invoke-WebRequest -Uri "http://localhost:8200/" -Method GET
   ```

3. **Test Remote Connectivity:**
   ```powershell
   Test-NetConnection -ComputerName your-apm-server -Port 8200
   ```

4. **Check Load Balancer Configuration:**
   - If using a load balancer, verify proper configuration
   - Ensure health checks are properly configured
   - Verify SSL termination settings

### Issue 6: SSL/TLS Configuration Issues

**Symptoms:**
- SSL handshake failures
- Certificate validation errors
- APM agents cannot establish secure connections

**Troubleshooting Steps:**

1. **Configure SSL in APM Server:**
   ```yaml
   apm-server:
     ssl:
       enabled: true
       certificate: "path/to/apm-server.crt"
       key: "path/to/apm-server.key"
   ```

2. **Verify Certificate Validity:**
   ```powershell
   openssl x509 -in apm-server.crt -text -noout
   ```

3. **Test SSL Connection:**
   ```powershell
   openssl s_client -connect your-apm-server:8200
   ```

## Authentication Issues

### Issue 7: API Key Authentication Problems

**Symptoms:**
- Authentication failures between APM Server and Elasticsearch
- "Unauthorized" errors in APM Server logs
- APM data not appearing in Elasticsearch

**Troubleshooting Steps:**

1. **Verify API Key Configuration:**
   - Check `apm-server.yml`:
   ```yaml
   output.elasticsearch:
     hosts: ["https://elasticsearch-server:9200"]
     api_key: "your-api-key-here"
   ```

2. **Test API Key Validity:**
   ```curl
   curl -X GET "https://elasticsearch-server:9200/_security/api_key" \
        -H "Authorization: ApiKey your-api-key"
   ```

3. **Create New API Key for APM Server:**
   ```curl
   curl -X POST "https://elasticsearch-server:9200/_security/api_key" \
        -H "Content-Type: application/json" \
        -u elastic:password \
        -d '{
          "name": "apm-server-api-key",
          "role_descriptors": {
            "apm_writer": {
              "cluster": ["monitor", "manage_ilm"],
              "index": [
                {
                  "names": ["apm-*", "logs-apm*", "metrics-apm*", "traces-apm*"],
                  "privileges": ["create_doc", "create_index", "auto_configure"]
                }
              ]
            }
          },
          "expiration": "365d"
        }'
   ```

4. **Update APM Server Configuration:**
   - Replace expired API key in `apm-server.yml`
   - Restart APM Server service

### Issue 8: Username/Password Authentication Issues

**Symptoms:**
- Login failures for APM Server to Elasticsearch
- "authentication failed" errors in logs
- APM data ingestion failures

**Troubleshooting Steps:**

1. **Verify User Configuration:**
   - Check `apm-server.yml`:
   ```yaml
   output.elasticsearch:
     hosts: ["https://elasticsearch-server:9200"]
     username: "apm_system"
     password: "your-password"
   ```

2. **Reset APM System User Password:**
   ```powershell
   # From Elasticsearch bin directory
   .\elasticsearch-reset-password.bat -u apm_system
   ```

3. **Test User Authentication:**
   ```curl
   curl -X GET "https://elasticsearch-server:9200/_security/user/apm_system" \
        -u elastic:admin-password
   ```

4. **Verify User Roles:**
   ```curl
   curl -X GET "https://elasticsearch-server:9200/_security/user/apm_system" \
        -u elastic:admin-password
   ```

   Expected roles for apm_system user:
   - `apm_system`
   - `ingest_admin` (if using ingest pipelines)

### Issue 9: APM Agent Authentication

**Symptoms:**
- APM agents cannot authenticate with APM Server
- Authentication token errors
- APM data not reaching server

**Troubleshooting Steps:**

1. **Configure APM Server Authentication:**
   ```yaml
   apm-server:
     auth:
       api_key:
         enabled: true
       secret_token: "your-secret-token"
   ```

2. **Generate Secret Token:**
   ```powershell
   # Generate random secret token
   -join ((1..64) | ForEach {Get-Random -input ([char[]]"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789")})
   ```

3. **Update APM Agent Configuration:**
   ```yaml
   # Example for .NET APM agent
   ELASTIC_APM_SECRET_TOKEN: "your-secret-token"
   ELASTIC_APM_SERVER_URL: "https://apm-server:8200"
   ```

4. **Test Agent Connection:**
   ```powershell
   # Test with curl
   curl -X POST "https://apm-server:8200/intake/v2/events" \
        -H "Authorization: Bearer your-secret-token" \
        -H "Content-Type: application/x-ndjson"
   ```

## Service Verification

### Issue 10: Verifying APM Server Health and Status

**Symptoms:**
- Need to confirm APM Server is operating correctly
- Performance monitoring requirements
- Health check automation

**Troubleshooting Steps:**

1. **Check APM Server Status:**
   ```curl
   curl -X GET "http://localhost:8200/"
   ```
   
   Expected response for healthy APM Server:
   ```json
   {
     "build_date": "2024-01-01T00:00:00Z",
     "build_sha": "abc123",
     "version": "8.x.x"
   }
   ```

2. **Verify Server Configuration:**
   ```curl
   curl -X GET "http://localhost:8200/config"
   ```

3. **Check Elasticsearch Connection:**
   ```powershell
   # Test from APM Server
   .\apm-server.exe test output -c apm-server.yml
   ```

4. **Monitor Performance Metrics:**
   ```curl
   curl -X GET "http://localhost:8200/stats"
   ```

### Issue 11: Data Flow Verification

**Symptoms:**
- APM data not appearing in Elasticsearch
- Missing transactions or metrics
- Incomplete trace data

**Troubleshooting Steps:**

1. **Verify APM Indices in Elasticsearch:**
   ```curl
   curl -X GET "https://elasticsearch-server:9200/_cat/indices/apm-*?v"
   ```

2. **Check Data Ingestion:**
   ```curl
   curl -X GET "https://elasticsearch-server:9200/apm-*/_search?size=1" \
        -H "Content-Type: application/json" \
        -d '{"sort": [{"@timestamp": {"order": "desc"}}]}'
   ```

3. **Verify Index Templates:**
   ```curl
   curl -X GET "https://elasticsearch-server:9200/_index_template/apm-*"
   ```

4. **Test Data Pipeline:**
   ```powershell
   # Send test event to APM Server
   curl -X POST "http://localhost:8200/intake/v2/events" \
        -H "Content-Type: application/x-ndjson" \
        -d '{"metadata":{"service":{"name":"test-service","version":"1.0.0"}}}
{"transaction":{"name":"test-transaction","type":"request","duration":100}}'
   ```

### Issue 12: Performance and Resource Monitoring

**Symptoms:**
- High CPU or memory usage
- Slow APM data processing
- Queue buildup in APM Server

**Troubleshooting Steps:**

1. **Monitor APM Server Metrics:**
   ```yaml
   # Enable monitoring in apm-server.yml
   monitoring:
     enabled: true
     elasticsearch:
       hosts: ["https://elasticsearch-server:9200"]
   ```

2. **Check Queue Status:**
   ```curl
   curl -X GET "http://localhost:8200/stats" | jq '.apm-server.processor.transaction'
   ```

3. **Monitor Resource Usage:**
   ```powershell
   Get-Process -Name apm-server | Select-Object CPU, WorkingSet, VirtualMemorySize
   ```

4. **Optimize Configuration:**
   ```yaml
   apm-server:
     max_request_size: "1mb"
     idle_timeout: "45s"
     read_timeout: "30s"
     write_timeout: "30s"
     shutdown_timeout: "30s"
   ```

## Additional Diagnostic Commands

### General Health Checks

```powershell
# Check service status
Get-Service apm-server | Select-Object Status, StartType, Name

# Check process information
Get-Process -Name apm-server | Select-Object Id, ProcessName, CPU, WorkingSet

# Check listening ports
Get-NetTCPConnection | Where-Object {$_.LocalPort -eq 8200} | Select-Object LocalAddress, LocalPort, State

# Test configuration file
.\apm-server.exe test config -c apm-server.yml

# Test output connectivity
.\apm-server.exe test output -c apm-server.yml
```

### Configuration Validation

```powershell
# Validate YAML syntax
.\apm-server.exe test config -c apm-server.yml

# Export current configuration
.\apm-server.exe export config -c apm-server.yml

# Test Elasticsearch template
.\apm-server.exe test template -c apm-server.yml
```

### Log Analysis

```powershell
# View recent APM Server logs
Get-Content "C:\apm-server\logs\apm-server*.log" -Tail 50

# Search for specific errors
Select-String -Path "C:\apm-server\logs\*.log" -Pattern "ERROR|WARN|FATAL" | Select-Object -Last 20

# Filter authentication errors
Select-String -Path "C:\apm-server\logs\*.log" -Pattern "authentication|unauthorized|forbidden" | Select-Object -Last 10

# Monitor real-time logs
Get-Content "C:\apm-server\logs\apm-server*.log" -Wait -Tail 10
```

### Network Diagnostics

```powershell
# Test APM Server endpoint
Invoke-WebRequest -Uri "http://localhost:8200/" -Method GET

# Test SSL endpoint
Test-NetConnection -ComputerName localhost -Port 8200

# Check certificate details
openssl s_client -connect localhost:8200 -servername localhost
```


