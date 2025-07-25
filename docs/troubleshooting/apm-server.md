# APM Server Troubleshooting

This document provides troubleshooting guidance for common APM Server issues encountered during installation, configuration, and operation in Relativity Server environments.

> [!NOTE]
> This guide assumes a default APM Server installation path of `C:\elastic\apm-server`. Adjust paths according to your actual installation directory.

## Table of Contents

- [Windows Service Issues](#windows-service-issues)
  - [APM Server Service Not Starting](#apm-server-service-not-starting)
  - [Service Crashes or Stops Unexpectedly](#service-crashes-or-stops-unexpectedly)
  - [Permission and Access Issues](#permission-and-access-issues)
- [Port Configuration Issues](#port-configuration-issues)
  - [Port Conflicts](#port-conflicts)
  - [Network Connectivity Problems](#network-connectivity-problems)
- [Elasticsearch Connection Issues](#elasticsearch-connection-issues)
  - [Connection Failures](#connection-failures)
  - [Authentication Problems](#authentication-problems)
  - [Network and Connectivity](#network-and-connectivity)
- [Authentication Issues](#authentication-issues)
  - [API Key Authentication Problems](#api-key-authentication-problems)
- [Service Verification](#service-verification)
  - [Verifying APM Server Health and Status](#verifying-apm-server-health-and-status)
- [Additional Diagnostic Commands](#additional-diagnostic-commands)

## Windows Service Issues

### APM Server Service Not Starting

**Symptoms:**
- APM Server service fails to start
- Service stops immediately after starting
- Error messages in APM Server logs

**Troubleshooting Steps:**

- **Check APM Server Status:**
   ```bash
   curl.exe -k -X GET "http://<hostname_or_ip>:8200/"
   ```

   <details>
   <summary>Expected response for healthy APM Server</summary>

   ```json
   {
      "build_date": "2025-02-27T18:17:35Z",
      "build_sha": "f6b917b725e1a22af433e5b52c5c6f0ff9164adf",
      "publish_ready": true,
      "version": "8.17.3"
   }
   ```
   </details>

- **Check Service Status:**
   ```powershell
   Get-Service -Name apm-server
   ```

- **Verify Service Configuration:**
   ```powershell
   (Get-CimInstance Win32_Service -Filter "Name = 'apm-server'").StartName
   ```

- **Check APM Server Logs:**
   - Navigate to `C:\Program Files\apm-server\logs\`
   - Review the latest log files (`apm-server.log`) for error messages
   - Look for configuration errors or connection issues with Elasticsearch

> [!NOTE]
> For Elasticsearch connection issues, see [Elasticsearch Connection Issues](#elasticsearch-connection-issues)

- **Verify Configuration File:**
   ```powershell
   # Stop Windows service first, then test configuration syntax
   Stop-Service apm-server
   C:\elastic\apm-server\apm-server.exe test config -c "C:\elastic\apm-server\apm-server.yml"
   ```

- **Start Service Manually:**
   ```powershell
   Start-Service apm-server
   ```

- **Run APM Server in Foreground for Debugging:**
   ```powershell
   # Ensure Windows service is stopped first
   Stop-Service apm-server
   C:\elastic\apm-server\apm-server.exe -e -c "C:\elastic\apm-server\apm-server.yml"
   ```

### Service Crashes or Stops Unexpectedly

**Symptoms:**
- APM Server service starts but stops after a short period
- Service status shows "Stopped" unexpectedly
- APM data collection stops working

**Troubleshooting Steps:**

1. **Check APM Server Logs:**
   - Navigate to `C:\Program Files\apm-server\logs\`
   - Review the latest log files (`apm-server.log`) for error messages
   - Look for service crash indicators or startup failures

2. **Review APM Server Configuration:**
   - Check `apm-server.yml` file in `C:\elastic\apm-server\`
   - Verify Elasticsearch connection settings (see [Elasticsearch Connection Issues](#elasticsearch-connection-issues) for detailed troubleshooting)
   - Common configuration issues:
     - **TLS**: Ensure correct protocol (`http` vs `https`)
     - **Hostname**: Verify correct Elasticsearch server hostname
     - **Port**: Confirm correct Elasticsearch port (usually 9200)

> [!NOTE]
> API keys are the preferred authentication method and expire by default in 6 months. Consider switching from username/password to API key authentication.

   ```yaml
   output.elasticsearch:
     hosts: ["https://<hostname_or_ip>:9200"]
     api_key: "your-api-key-here"
     # OR (not recommended)
     # username: "<username>"
     # password: "<password>"
   ```

3. **Verify Elasticsearch Connectivity:**

> [!NOTE]
> For detailed Elasticsearch connection troubleshooting, see [Elasticsearch Connection Issues](#elasticsearch-connection-issues)

   ```bash
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/"
   ```

   <details>
   <summary>Expected response for healthy Elasticsearch</summary>

   ```json
   {
    "name" : "EMTTEST",
    "cluster_name" : "elasticsearch",
    "cluster_uuid" : "vD-9K0uGRwuFm1XpLkgI5Q",
    "version" : {
      "number" : "8.17.3",
      "build_flavor" : "default",
      "build_type" : "zip",
      "build_hash" : "a091390de485bd4b127884f7e565c0cad59b10d2",
      "build_date" : "2025-02-28T10:07:26.089129809Z",
      "build_snapshot" : false,
      "lucene_version" : "9.12.0",
      "minimum_wire_compatibility_version" : "7.17.0",
      "minimum_index_compatibility_version" : "7.0.0"
      },
   "tagline" : "You Know, for Search"
   }
   ```
   </details>

4. **Check Resource Usage:**
   - The APM Server generates minimal data compared to Elasticsearch/Kibana
   - All logs are written to `C:\Program Files\apm-server\logs\`
   - Verify sufficient disk space for logs

5. **Validate Elasticsearch Configuration:**
   ```yaml
   output.elasticsearch:
     hosts: ["https://<hostname_or_ip>:9200"]
     api_key: "your-api-key-here"
     protocol: "https"
     ssl.enabled: true
     ssl.verification_mode: none
   ```

### Permission and Access Issues

**Symptoms:**
- Access denied errors when starting service
- Unable to write to log directories
- Configuration file access errors

**Troubleshooting Steps:**

1. **Verify Service Configuration:**
   - The APM Server Windows service runs under Local System account by default
   - Verify access to `C:\elastic\apm-server\` directory
   - Check write permissions to `C:\Program Files\apm-server\logs\` directory

## Port Configuration Issues

### Port Conflicts

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

> [!NOTE]
> If no output is returned, port 8200 is available. If you see `LISTENING` status, the port is already in use.

2. **Identify Port Conflicts:**
   ```powershell
   Get-NetTCPConnection -LocalPort 8200 -State Listen
   ```

> [!NOTE]
> If this command returns results, another process is using port 8200. If no results are returned, the port is available.

3. **Resolve Port Conflicts:**

> [!IMPORTANT]
> Do not change the APM Server port. Instead, identify and stop the conflicting service using port 8200, as changing the APM Server port requires extensive configuration changes across Environment Watch, Relativity, and other components.

### Network Connectivity Problems

**Symptoms:**
- Service Not Running: APM Server or Elasticsearch may not be running or listening on the expected endpoints.
- Incorrect Configuration: The APM Server or Elasticsearch endpoint URLs may be misconfigured (wrong host, port, or protocol).
- Firewall Rules: Firewalls on the VM host or network may be blocking required ports (e.g., 9200 for Elasticsearch, 8200 for APM Server).

**Troubleshooting Steps:**

1. **Verify Network Binding:**
   - Check `apm-server.yml` configuration:
   ```yaml
   apm-server:
     host: "0.0.0.0:8200"  # Listen on all interfaces
     # or
     host: "<hostname_or_ip>:8200"
   ```

2. **Test APM Server Connectivity:**
   ```bash
   curl.exe -k -X GET "https://<hostname_or_ip>:8200/"
   ```

   <details>
   <summary>Expected response for healthy APM Server</summary>

   ```json
   {
     "build_date": "2025-02-27T18:17:35Z",
     "build_sha": "f6b917b725e1a22af433e5b52c5c6f0ff9164adf",
     "publish_ready": true,
     "version": "8.17.3"
   }
   ```
   </details>

3. **Test Remote Connectivity:**
   ```powershell
   Test-NetConnection -ComputerName <hostname_or_ip> -Port 8200
   ```

## Elasticsearch Connection Issues

### Connection Failures

**Symptoms:**
- APM Server cannot connect to Elasticsearch
- "connection refused" or "connection timeout" errors in APM Server logs
- APM data not being indexed in Elasticsearch

**Troubleshooting Steps:**

1. **Verify Elasticsearch Service Status:**
   ```powershell
   # Check if Elasticsearch service is running
   Get-Service -Name elasticsearch
   ```

> [!TIP]
> For detailed Elasticsearch troubleshooting, see [Elasticsearch Troubleshooting](elasticsearch.md)

2. **Test Elasticsearch Connectivity:**
   ```bash
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/"
   ```

   <details>
   <summary>Expected response for healthy Elasticsearch</summary>

   ```json
   {
      "name" : "EMTTEST",
      "cluster_name" : "elasticsearch",
      "cluster_uuid" : "PwBZoINKQjGZ53WH4gFfBg",
      "version" : {
            "number" : "8.17.3",
         "build_flavor" : "default",
         "build_type" : "zip",
         "build_hash" : "a091390de485bd4b127884f7e565c0cad59b10d2",
         "build_date" : "2025-02-28T10:07:26.089129809Z",
         "build_snapshot" : false,
         "lucene_version" : "9.12.0",
         "minimum_wire_compatibility_version" : "7.17.0",
         "minimum_index_compatibility_version" : "7.0.0"
      },
      "tagline" : "You Know, for Search"
   }
   ```
   </details>

3. **Check Elasticsearch Cluster Health:**
   ```bash
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/_cluster/health?pretty"
   ```

   <details>
   <summary>Expected response for healthy cluster</summary>

   ```json
   {
      "cluster_name" : "elasticsearch",
      "status" : "green",
      "timed_out" : false,
      "number_of_nodes" : 1,
      "number_of_data_nodes" : 1,
      "active_primary_shards" : 129,
      "active_shards" : 129,
      "relocating_shards" : 0,
      "initializing_shards" : 0,
      "unassigned_shards" : 83,
      "unassigned_primary_shards" : 0,
      "delayed_unassigned_shards" : 0,
      "number_of_pending_tasks" : 0,
      "number_of_in_flight_fetch" : 0,
      "task_max_waiting_in_queue_millis" : 0,
      "active_shards_percent_as_number" : 60.84905660377359
   }
   ```
   </details>

4. **Verify APM Server Output Configuration:**
   ```yaml
   # Check apm-server.yml
   output.elasticsearch:
     hosts: ["https://<hostname_or_ip>:9200"]
     protocol: "https"  # or "http" if not using SSL
     api_key: "your-api-key"
   ```

5. **Test APM Server to Elasticsearch Connection:**
   ```powershell
   # Stop service first, then test output connectivity
   Stop-Service apm-server
   C:\elastic\apm-server\apm-server.exe test output -c "C:\elastic\apm-server\apm-server.yml"
   ```

### Authentication Problems

**Symptoms:**
- "Unauthorized" (401) errors in APM Server logs
- "Forbidden" (403) errors when APM Server tries to write to Elasticsearch
- Authentication failures between APM Server and Elasticsearch

**Troubleshooting Steps:**

1. **Verify API Key Configuration:**
   ```yaml
   # In apm-server.yml
   output.elasticsearch:
     hosts: ["https://<hostname_or_ip>:9200"]
     api_key: "your-base64-encoded-api-key"
   ```

2. **Test API Key Validity:**
   ```bash
   curl.exe -k -X GET "https://<hostname_or_ip>:9200/_security/api_key" \
        -H "Authorization: ApiKey your-api-key"
   ```

3. **Check API Key Expiration:**
   ```bash
   curl.exe -k -X GET "https://<hostname_or_ip>:9200/_security/api_key?owner=true" \
        -H "Authorization: ApiKey your-api-key"
   ```

> [!IMPORTANT]
> API keys expire by default in 6 months. Check the `expiration` field in the response.

4. **Verify API Key Permissions:**
   The API key must have appropriate permissions for APM data:
   ```json
   {
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
     }
   }
   ```

5. **Replace Expired API Key:**
   ```powershell
   # Create the JSON file
   @'
   {
     "name": "apm-server-api-key",
     "expiration": "365d",
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
     }
   }
   '@ | Out-File -FilePath "apm-api-key.json" -Encoding utf8
   
   # Use curl with the file
   curl.exe -k -X POST "https://<hostname_or_ip>:9200/_security/api_key" -H "Content-Type: application/json" -u <username>:<password> -d @apm-api-key.json
   
   # Clean up (optional)
   Remove-Item "apm-api-key.json"
   ```

6. **Update APM Server Configuration:**
   ```yaml
   output.elasticsearch:
    api_key: "your-new-api-key-here"
   ```

   Then restart APM Server service:
   ```powershell
   Restart-Service apm-server
   ```

### Network and Connectivity

**Symptoms:**
- Network timeouts when connecting to Elasticsearch
- DNS resolution failures
- Firewall blocking connections

**Troubleshooting Steps:**

1. **Test Network Connectivity:**
   ```powershell
   Test-NetConnection -ComputerName <hostname_or_ip> -Port 9200
   ```

2. **Verify DNS Resolution:**
   ```powershell
   Resolve-DnsName <hostname_or_ip>
   ```

3. **Check Firewall Rules:**
   ```powershell
   # Check if port 9200 is open
   Test-NetConnection -ComputerName <hostname_or_ip> -Port 9200 -InformationLevel Detailed
   ```

4. **Validate TLS/SSL Configuration:**
   ```bash
   # Test SSL connection to Elasticsearch
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/"
   ```

> [!NOTE]
> The `-k` flag bypasses certificate validation for testing purposes only.

5. **Check APM Server Network Configuration:**
   ```yaml
   # Ensure APM Server can reach Elasticsearch
   output.elasticsearch:
     hosts: ["https://<hostname_or_ip>:9200"]
     timeout: 90s
     max_retries: 3
   ```

6. **Test from APM Server Host:**
   ```bash
   # Run from the same machine as APM Server
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/_cluster/health"
   ```

## Authentication Issues

### API Key Authentication Problems

**Symptoms:**
- Authentication failures between APM Server and Elasticsearch
- "Unauthorized" errors in APM Server logs
- APM data not appearing in Elasticsearch

> [!NOTE]
> For comprehensive Elasticsearch connection and authentication troubleshooting, see [Elasticsearch Connection Issues](#elasticsearch-connection-issues)

**Troubleshooting Steps:**

1. **Verify API Key Configuration:**
   - Check `apm-server.yml`:
   ```yaml
   output.elasticsearch:
     hosts: ["https://<hostname_or_ip>:9200"]
     api_key: "your-api-key-here"
   ```

2. **Test API Key Validity:**
   ```bash
   curl.exe -k -X GET "https://<hostname_or_ip>:9200/_security/api_key" \
        -H "Authorization: ApiKey your-api-key"
   ```

   <details>
   <summary>Expected response for valid API key</summary>

   ```json
   {
      "api_keys": [
      {
         "id": "YGcGXJcBAkfpte8rEst4",
         "name": "rel-infrawatch",
         "type": "rest",
         "creation": 1749595591294,
         "expiration": 1765147591294,
         "invalidated": false,
         "username": "elastic",
         "realm": "reserved",
         "realm_type": "reserved",
         "metadata": {},
         "role_descriptors": {}
      }
      ]
   }
   ```
   </details>

3. **Replace Expired API Key:**
   For detailed steps on creating API keys, see [Authentication Problems](#authentication-problems) section.

   ```powershell
   # Create the JSON file
   @'
   {
     "name": "apm-server-api-key",
     "expiration": "365d",
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
     }
   }
   '@ | Out-File -FilePath "apm-api-key.json" -Encoding utf8
   
   # Use curl with the file
   curl.exe -k -X POST "https://<hostname_or_ip>:9200/_security/api_key" -H "Content-Type: application/json" -u <username>:<password> -d @apm-api-key.json
   
   # Clean up (optional)
   Remove-Item "apm-api-key.json"
   ```

4. **Update APM Server Configuration:**
   ```yaml
   output.elasticsearch:
    hosts: ["https://<hostname_or_ip>:9200"]
    api_key: "your-api-key-here"
    protocol: "https"
    ssl.enabled: true
    ssl.verification_mode: none
   ```

   Then restart APM Server service:
   ```powershell
   Restart-Service apm-server
   ```

## Service Verification

### Verifying APM Server Health and Status

**Symptoms:**
- Need to confirm APM Server is operating correctly
- Performance monitoring requirements
- Health check automation

**Troubleshooting Steps:**

- **Verify Server Configuration:**
   ```bash
   C:\elastic\apm-server\apm-server.exe test config -c "C:\elastic\apm-server\apm-server.yml"
   ```

   <details>
   <summary>Expected response includes server configuration details</summary>

   ```
   Config OK
   ```
   </details>

- **Check Elasticsearch Connection:**
   ```powershell
   # Stop Windows service first, then test output connectivity
   Stop-Service apm-server
   C:\elastic\apm-server\apm-server.exe test output -c "C:\elastic\apm-server\apm-server.yml"
   ```

   <details>
   <summary>Expected output for successful connection</summary>

   ```
   elasticsearch: https://<hostname_or_ip>:9200...
     parse url... OK
     connection...
       parse host... OK
       dns lookup... OK
       addresses: 192.168.1.100
       dial up... OK
     TLS... WARN secure connection disabled
     talk to server... OK
   ```
   </details>

> [!NOTE]
> If this test fails, see [Elasticsearch Connection Issues](#elasticsearch-connection-issues) for detailed troubleshooting steps.

## Additional Diagnostic Commands

### General Health Checks

```powershell
# Check service status
Get-Service apm-server | Select-Object Status, StartType, Name

# Check process information
Get-Process -Name apm-server | Select-Object Id, ProcessName, CPU, WorkingSet

# Check listening ports
Get-NetTCPConnection | Where-Object {$_.LocalPort -eq 8200} | Select-Object LocalAddress, LocalPort, State

# Test configuration file (stop service first)
Stop-Service apm-server
C:\elastic\apm-server\apm-server.exe test config -c "C:\elastic\apm-server\apm-server.yml"

# Test output connectivity
C:\elastic\apm-server\apm-server.exe test output -c "C:\elastic\apm-server\apm-server.yml"

# Note: If Elasticsearch connection tests fail, see Elasticsearch Connection Issues section
```

### Configuration Validation

```powershell
# Validate YAML syntax (stop service first)
Stop-Service apm-server
C:\elastic\apm-server\apm-server.exe test config -c "C:\elastic\apm-server\apm-server.yml"

# Export current configuration
C:\elastic\apm-server\apm-server.exe export config -c "C:\elastic\apm-server\apm-server.yml"

# Test Elasticsearch template
C:\elastic\apm-server\apm-server.exe test template -c "C:\elastic\apm-server\apm-server.yml"
```

### Log Analysis

```powershell
# View recent APM Server logs
Get-Content "C:\Program Files\apm-server\logs\apm-server*.log" -Tail 50

# Search for specific errors
Select-String -Path "C:\Program Files\apm-server\logs\*.log" -Pattern "ERROR|WARN|FATAL" | Select-Object -Last 20

# Filter authentication errors
Select-String -Path "C:\Program Files\apm-server\logs\*.log" -Pattern "authentication|unauthorized|forbidden" | Select-Object -Last 10

# Monitor real-time logs
Get-Content "C:\Program Files\apm-server\logs\apm-server*.log" -Wait -Tail 10
```

### Network Diagnostics

```powershell
# Test network connectivity
Test-NetConnection -ComputerName <hostname_or_ip> -Port 8200
```


