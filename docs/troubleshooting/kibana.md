# Kibana Troubleshooting

This document provides troubleshooting guidance for common Kibana issues encountered during installation, configuration, and operation in Relativity environments.

> [!NOTE]
> This guide assumes a default Kibana installation path of `C:\elastic\kibana`. Adjust paths according to your actual installation directory.

## Table of Contents

- [Windows Service Issues](#windows-service-issues)
  - [Kibana Service Not Starting](#kibana-service-not-starting)
  - [Service Crashes or Stops Unexpectedly](#service-crashes-or-stops-unexpectedly)
- [Port Configuration Issues](#port-configuration-issues)
  - [Port Conflicts](#port-conflicts)
  - [Network Binding Problems](#network-binding-problems)
- [Memory Issues](#memory-issues)
  - [Insufficient Memory Allocation](#insufficient-memory-allocation)
  - [Large Dataset Performance Issues](#large-dataset-performance-issues)
- [Authentication Issues](#authentication-issues)
  - [API Key Authentication Problems](#api-key-authentication-problems)
  - [Username/Password Authentication Issues](#usernamepassword-authentication-issues)
  - [SSL/TLS Configuration Problems](#ssltls-configuration-problems)
- [Kibana Encryption Keys Configuration](#kibana-encryption-keys-configuration)
  - [Missing or Invalid Encryption Keys](#missing-or-invalid-encryption-keys)
- [Service Verification](#service-verification)
  - [Verifying Kibana Health and Status](#verifying-kibana-health-and-status)
  - [Dashboard and Visualization Verification](#dashboard-and-visualization-verification)
- [Additional Diagnostic Commands](#additional-diagnostic-commands)

## Windows Service Issues

### Kibana Service Not Starting

**Symptoms:**
- Kibana service fails to start
- Service stops immediately after starting
- Error messages in Kibana logs

**Troubleshooting Steps:**

- **Check Service Status:**
   ```powershell
   Get-Service -Name kibana
   ```

- **Verify Service Configuration:**
   - Open Services.msc
   - Locate "Kibana" service
   - Verify the service is running under Local System account (default configuration)

- **Check Kibana Logs:**
   - Navigate to `C:\elastic\kibana\logs\`
   - Review the latest log files (`kibana.log`) for error messages
   - Look for configuration errors or Elasticsearch connection issues

- **Verify Node.js Runtime:**
   - Kibana includes its own Node.js runtime
   - No additional Node.js installation required
   - Check for conflicting Node.js installations if issues persist

- **Start Service Manually:**
   ```powershell
   Start-Service kibana
   ```

### Service Crashes or Stops Unexpectedly

**Symptoms:**
- Kibana service starts but stops after a short period
- Service status shows "Stopped" unexpectedly
- Users lose access to Kibana interface

**Troubleshooting Steps:**

1. **Check Kibana Logs:**
   - Navigate to `C:\elastic\kibana\logs\`
   - Review the latest log files (`kibana.log`) for crash details
   - Look for memory issues or connection failures

2. **Review Kibana Configuration:**
   - Check `C:\elastic\kibana\config\kibana.yml` file
   - Verify Elasticsearch connection settings
   - Ensure all required configuration parameters are present

3. **Verify Elasticsearch Connectivity:**
   ```bash
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/"
   ```

4. **Check Memory Usage:**
   - Monitor Kibana process memory consumption
   - Verify sufficient system memory is available

## Port Configuration Issues

### Port Conflicts

**Symptoms:**
- Kibana fails to bind to default port
- "EADDRINUSE" errors in logs
- Cannot access Kibana web interface

**Troubleshooting Steps:**

- **Check Default Port:**
   - Default Kibana port: 5601
   - Verify port availability:
   ```powershell
   netstat -an | findstr ":5601"
   ```

- **Test Kibana Connectivity:**
   ```bash
   curl.exe -k -X GET "http://<hostname_or_ip>:5601/"
   ```

- **Configure Alternative Port (if needed):**
   - Edit `C:\elastic\kibana\config\kibana.yml`:
   ```yaml
   server.port: 5602
   ```

- **Update Firewall Rules:**
   ```powershell
   New-NetFirewallRule -DisplayName "Kibana Web Interface" -Direction Inbound -Protocol TCP -LocalPort 5601 -Action Allow
   ```

### Network Binding Problems

**Symptoms:**
- Cannot access Kibana from remote hosts
- Connection refused errors
- Kibana only accessible from localhost

**Troubleshooting Steps:**

- **Verify Network Configuration:**
   - Check `C:\elastic\kibana\config\kibana.yml` configuration:
   ```yaml
   server.host: "0.0.0.0"  # For all interfaces
   # or
   server.host: "<hostname_or_ip>"
   ```

- **Test Local Access:**
   ```bash
   curl.exe -k -X GET "http://localhost:5601/"
   ```

- **Test Remote Access:**
   ```bash
   curl.exe -k -X GET "http://<hostname_or_ip>:5601/"
   ```

- **Check Load Balancer Configuration:**
   - If using a load balancer, verify proper configuration
   - Ensure health checks are properly configured

## Memory Issues

### Insufficient Memory Allocation

**Symptoms:**
- Kibana becomes unresponsive
- Slow loading of dashboards and visualizations
- Out of memory errors in logs

**Troubleshooting Steps:**

1. **Check Current Memory Usage:**
   ```powershell
   Get-Process -Name node | Where-Object {$_.ProcessName -eq "node"} | Select-Object WorkingSet, VirtualMemorySize
   ```

2. **Configure Node.js Memory Limits:**
   - Edit Kibana startup script or service configuration:
   ```
   NODE_OPTIONS="--max-old-space-size=4096"
   ```

3. **Monitor Memory Usage:**
   - Use Task Manager or Performance Monitor
   - Track memory consumption over time
   - Look for memory leaks

4. **Optimize Kibana Configuration:**
   - Reduce concurrent search requests:
   ```yaml
   elasticsearch.requestTimeout: 30000
   elasticsearch.maxSockets: 1024
   ```

### Large Dataset Performance Issues

**Symptoms:**
- Slow visualization rendering
- Timeouts when loading large datasets
- Browser becomes unresponsive

**Troubleshooting Steps:**

1. **Adjust Query Settings:**
   ```yaml
   elasticsearch.requestTimeout: 60000
   elasticsearch.shardTimeout: 30000
   ```

2. **Configure Data Sampling:**
   ```yaml
   discover.sampleSize: 500
   ```

3. **Optimize Index Patterns:**
   - Use time-based indices
   - Configure appropriate field mappings
   - Limit field discovery

## Authentication Issues

### API Key Authentication Problems

**Symptoms:**
- Authentication failures between Kibana and Elasticsearch
- "Unable to retrieve version information" errors
- 401 Unauthorized responses in Kibana logs

**Troubleshooting Steps:**

- **Verify API Key Configuration:**
   - Check `kibana.yml`:
   ```yaml
   elasticsearch.apiVersion: "8.x"
   elasticsearch.hosts: ["https://<hostname_or_ip>:9200"]
   elasticsearch.apiKey: "your-api-key-here"
   ```

- **Test API Key Validity:**
   ```bash
   curl.exe -k -X GET "https://<hostname_or_ip>:9200/_security/api_key" \
        -H "Authorization: ApiKey your-api-key"
   ```

- **Create New API Key for Kibana:**
   ```powershell
   # Create the JSON file
   @'
   {
     "name": "kibana-api-key",
     "expiration": "365d",
     "role_descriptors": {
       "kibana_admin": {
         "cluster": ["all"],
         "index": [
           {
             "names": ["*"],
             "privileges": ["all"]
           }
         ]
       }
     }
   }
   '@ | Out-File -FilePath "kibana-api-key.json" -Encoding utf8
   
   # Use curl with the file
   curl.exe -k -X POST "https://<hostname_or_ip>:9200/_security/api_key" -H "Content-Type: application/json" -u <username>:<password> -d @kibana-api-key.json
   
   # Clean up (optional)
   Remove-Item "kibana-api-key.json"
   ```

- **Update Kibana Configuration:**
   - Replace expired API key in `kibana.yml`
   - Restart Kibana service

### Username/Password Authentication Issues

**Symptoms:**
- Login failures at Kibana interface
- "Invalid username or password" errors
- Users cannot access Kibana dashboards

**Troubleshooting Steps:**

1. **Verify User Configuration:**
   - Check `kibana.yml`:
   ```yaml
   elasticsearch.username: "kibana_system"
   elasticsearch.password: "<password>"
   ```

2. **Reset Kibana System User Password:**
   ```powershell
   # From Elasticsearch bin directory
   C:\elastic\elasticsearch\bin\elasticsearch-reset-password.bat -u kibana_system
   ```

3. **Test User Authentication:**
   ```bash
   curl.exe -k -X GET "https://<hostname_or_ip>:9200/_security/user/kibana_system" \
        -u <username>:<password>
   ```

4. **Configure LDAP/Active Directory Integration:**
   ```yaml
   xpack.security.authc.providers:
     basic.basic1:
       order: 0
     ldap.ldap1:
       order: 1
       realm: "ldap_realm"
   ```

### SSL/TLS Configuration Problems

**Symptoms:**
- SSL certificate errors
- "unable to verify the first certificate" warnings
- HTTPS connection failures

**Troubleshooting Steps:**

1. **Configure SSL for Elasticsearch Connection:**
   ```yaml
   elasticsearch.ssl.certificateAuthorities: ["path/to/ca.crt"]
   elasticsearch.ssl.verificationMode: "certificate"
   ```

2. **Disable SSL Verification (Not Recommended for Production):**
   ```yaml
   elasticsearch.ssl.verificationMode: "none"
   ```

3. **Configure Kibana HTTPS:**
   ```yaml
   server.ssl.enabled: true
   server.ssl.certificate: "path/to/kibana.crt"
   server.ssl.key: "path/to/kibana.key"
   ```

## Kibana Encryption Keys Configuration

### Missing or Invalid Encryption Keys

**Symptoms:**
- Kibana fails to start with encryption-related errors
- "Saved objects encryption key is missing" warnings
- Unable to save or retrieve saved objects

**Troubleshooting Steps:**

1. **Generate Encryption Keys:**
   ```powershell
   # Navigate to Kibana bin directory
   .\kibana-encryption-keys.bat generate
   ```

2. **Configure Required Encryption Keys in kibana.yml:**
   ```yaml
   # Saved Objects encryption key (required)
    xpack.encryptedSavedObjects.encryptionKey: "<randomly-generated-key-1>"
   
   # Reporting encryption key (if using reporting)
   xpack.reporting.encryptionKey: "<randomly-generated-key-2>"
   
   # Security session encryption key (if using security)
   xpack.security.encryptionKey: "<randomly-generated-key-3>"
   
   # Alerting encryption key (if using alerting)
   xpack.encryptedSavedObjects.encryptionKey: "<randomly-generated-key-4>"
   ```

3. **Key Requirements:**
   - Keys must be at least 32 characters long
   - Use cryptographically secure random strings
   - Keep keys consistent across Kibana instances in a cluster

4. **Generate Secure Keys:**
   ```powershell
   # Generate random 32-character string
   -join ((1..32) | ForEach {Get-Random -input ([char[]]"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789")})
   ```

## Service Verification

### Verifying Kibana Health and Status

**Symptoms:**
- Need to confirm Kibana is operating correctly
- Performance monitoring requirements
- Health check automation

**Troubleshooting Steps:**

- **Check Kibana Status:**
   ```bash
   curl.exe -k -X GET "http://<hostname_or_ip>:5601/api/status"
   ```
   
   <details>
   <summary>Expected response for healthy Kibana</summary>

   ```json
   {
     "name": "kibana",
     "version": {
       "number": "8.x.x"
     },
     "status": {
       "overall": {
         "level": "available"
       }
     }
   }
   ```
   </details>

- **Verify Elasticsearch Connection:**
   ```bash
   curl.exe -k -X GET "http://<hostname_or_ip>:5601/api/status" | jq '.status.statuses[] | select(.id=="elasticsearch")'
   ```

- **Check Plugin Status:**
   ```bash
   curl.exe -k -X GET "http://<hostname_or_ip>:5601/api/status" | jq '.status.statuses[] | select(.id=="plugin:*")'
   ```

- **Monitor Performance Metrics:**
   ```bash
   curl.exe -k -X GET "http://<hostname_or_ip>:5601/api/stats"
   ```

### Dashboard and Visualization Verification

**Symptoms:**
- Dashboards not loading properly
- Visualizations showing no data
- Index pattern issues

**Troubleshooting Steps:**

- **Verify Index Patterns:**
   ```bash
   curl.exe -k -X GET "http://<hostname_or_ip>:5601/api/saved_objects/_find?type=index-pattern"
   ```

- **Check Data View Configuration:**
   - Navigate to Stack Management > Data Views
   - Verify index patterns match Elasticsearch indices
   - Refresh field mappings if necessary

- **Test Search Functionality:**
   ```bash
   curl.exe -k -X POST "http://<hostname_or_ip>:5601/api/console/proxy?path=_search&method=GET" \
        -H "Content-Type: application/json" \
        -d '{"query": {"match_all": {}}}'
   ```

- **Validate Time Range Settings:**
   - Check time picker configuration
   - Verify data exists within selected time range
   - Ensure timezone settings are correct

## Additional Diagnostic Commands

### General Health Checks

```powershell
# Check service status
Get-Service kibana | Select-Object Status, StartType, Name

# Check process information
Get-Process -Name node | Where-Object {$_.MainModule.FileName -like "*kibana*"}

# Check listening ports
Get-NetTCPConnection | Where-Object {$_.LocalPort -eq 5601} | Select-Object LocalAddress, LocalPort, State

# Check disk space for Kibana data
Get-WmiObject -Class Win32_LogicalDisk | Select-Object DeviceID, @{Name="Size(GB)";Expression={[math]::Round($_.Size/1GB,2)}}, @{Name="FreeSpace(GB)";Expression={[math]::Round($_.FreeSpace/1GB,2)}}

# Test web interface connectivity
Test-NetConnection -ComputerName <hostname_or_ip> -Port 5601
```

### Configuration Validation

```powershell
# Validate YAML syntax
C:\elastic\kibana\bin\kibana.bat --validate-config

# Check current configuration
C:\elastic\kibana\bin\kibana.bat --config-path="C:\elastic\kibana\config\kibana.yml" --dry-run
```

### Log Analysis

```powershell
# View recent Kibana logs
Get-Content "C:\elastic\kibana\logs\kibana.log" -Tail 50

# Search for specific errors
Select-String -Path "C:\elastic\kibana\logs\*.log" -Pattern "ERROR|WARN|FATAL" | Select-Object -Last 20

# Filter authentication errors
Select-String -Path "C:\elastic\kibana\logs\*.log" -Pattern "authentication|unauthorized" | Select-Object -Last 10
```

