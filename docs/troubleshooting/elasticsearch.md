# Elasticsearch Troubleshooting

This document provides troubleshooting guidance for common Elasticsearch issues encountered during installation, configuration, and operation in Relativity environments.

> [!NOTE]
> This guide assumes a default Elasticsearch installation path of `C:\elastic\elasticsearch`. Adjust paths according to your actual installation directory.

## Table of Contents

- [Windows Service Issues](#windows-service-issues)
  - [Issue 1: Elasticsearch Service Not Starting](#issue-1-elasticsearch-service-not-starting)
  - [Issue 2: Service Crashes or Stops Unexpectedly](#issue-2-service-crashes-or-stops-unexpectedly)
- [Port Configuration Issues](#port-configuration-issues)
  - [Issue 3: Port Conflicts](#issue-3-port-conflicts)
  - [Issue 4: Network Connectivity Problems](#issue-4-network-connectivity-problems)
- [Memory Issues](#memory-issues)
  - [Issue 5: Insufficient Memory Allocation](#issue-5-insufficient-memory-allocation)
  - [Issue 6: Virtual Memory Configuration](#issue-6-virtual-memory-configuration)
- [Authentication Issues](#authentication-issues)
  - [Issue 7: API Key Expiration](#issue-7-api-key-expiration)
  - [Issue 8: Username/Password Authentication Problems](#issue-8-usernamepassword-authentication-problems)
  - [Issue 9: SSL/TLS Certificate Issues](#issue-9-ssltls-certificate-issues)
- [Service Verification](#service-verification)
  - [Issue 10: Verifying Elasticsearch Health](#issue-10-verifying-elasticsearch-health)
  - [Issue 11: Service Dependencies Verification](#issue-11-service-dependencies-verification)
- [Additional Diagnostic Commands](#additional-diagnostic-commands)

## Windows Service Issues

### Issue 1: Elasticsearch Service Not Starting

**Symptoms:**
- Elasticsearch service fails to start
- Service stops immediately after starting
- Error messages in Elasticsearch logs

**Troubleshooting Steps:**

1. **Check Service Status:**
   ```powershell
   Get-Service -Name elasticsearch
   ```

2. **Verify Service Configuration:**
   - Open Services.msc
   - Locate "Elasticsearch" service
   - Verify the service is running under Local System account (default configuration)

3. **Check Elasticsearch Logs:**
   - Navigate to `C:\elastic\elasticsearch\logs\`
   - Review the cluster log file (`elasticsearch.log`) for error messages
   - Check the slow logs and garbage collection logs if present
   - Look for Java heap space issues or configuration errors
   
   > [!TIP]
   > For detailed logging information, refer to the [official Elasticsearch logging documentation](https://www.elastic.co/guide/en/elasticsearch/reference/8.17/logging.html)

4. **Verify Java Installation:**
   ```powershell
   java -version
   ```
   - Ensure Java 11 or later is installed
   - Check JAVA_HOME environment variable is set correctly

5. **Start Service Manually:**
   ```powershell
   Start-Service elasticsearch
   ```

### Issue 2: Service Crashes or Stops Unexpectedly

**Symptoms:**
- Elasticsearch service starts but stops after a short period
- Service status shows "Stopped" unexpectedly

**Troubleshooting Steps:**

1. **Check Elasticsearch Logs:**
   - Navigate to `C:\elastic\elasticsearch\logs\`
   - Review the cluster log file (`elasticsearch.log`) for errors
   - Look for OutOfMemoryError or service crash indicators

2. **Review JVM Settings (Advanced):**
   - Check `jvm.options` file in `C:\elastic\elasticsearch\config\`
   - Monitor current memory usage to determine if heap adjustment is needed
   - If modifying heap settings:
     - `-Xms`: Initial heap size (should match -Xmx)
     - `-Xmx`: Maximum heap size (recommend 50% of available RAM, max 32GB)
   - Example configuration for a system with 16GB RAM:
   ```
   -Xms4g
   -Xmx4g
   ```

3. **Verify Disk Space:**
   - Ensure sufficient disk space on data and log directories (minimum 15% free)
   - Verify data and log files are on separate drives from the Operating System drive
   ```powershell
   # Check disk space
   Get-WmiObject -Class Win32_LogicalDisk | Select-Object DeviceID, @{Name="FreeSpace(%)";Expression={[math]::Round(($_.FreeSpace/$_.Size)*100,2)}}
   ```

## Port Configuration Issues

### Issue 3: Port Conflicts

**Symptoms:**
- Elasticsearch fails to bind to default ports
- "Address already in use" errors in logs
- Cannot access Elasticsearch via HTTP/HTTPS

> [!NOTE]
> A comprehensive port diagram will be available soon and referenced across multiple documentation pages.

**Troubleshooting Steps:**

1. **Check Default Ports:**
   - HTTP: 9200
   - Transport: 9300
   - Verify these ports are available:
   ```powershell
   netstat -an | findstr ":9200"
   netstat -an | findstr ":9300"
   ```

2. **Test Elasticsearch Connectivity:**
   ```powershell
   # Test Elasticsearch HTTP port
   curl -X GET -u username:password "localhost:9200/"
   
   # Alternative using PowerShell
   Invoke-RestMethod -Uri "http://localhost:9200/" -Method GET
   ```

3. **Identify Port Conflicts:**
   ```powershell
   Get-NetTCPConnection -LocalPort 9200 -State Listen
   Get-NetTCPConnection -LocalPort 9300 -State Listen
   ```

3. **Configure Alternative Ports (if needed):**
   - Edit `C:\elastic\elasticsearch\config\elasticsearch.yml`:
   ```yaml
   http.port: 9201
   transport.port: 9301
   ```

4. **Update Firewall Rules:**
   ```powershell
   New-NetFirewallRule -DisplayName "Elasticsearch HTTP" -Direction Inbound -Protocol TCP -LocalPort 9200 -Action Allow
   New-NetFirewallRule -DisplayName "Elasticsearch Transport" -Direction Inbound -Protocol TCP -LocalPort 9300 -Action Allow
   ```

### Issue 4: Network Connectivity Problems

**Symptoms:**
- Cannot connect to Elasticsearch from remote hosts
- Connection timeouts
- "No route to host" errors

**Troubleshooting Steps:**

1. **Verify Network Binding:**
   - Check `C:\elastic\elasticsearch\config\elasticsearch.yml` configuration:
   ```yaml
   network.host: 0.0.0.0  # For all interfaces
   # or
   network.host: ["_local_", "_site_"]
   ```

2. **Test Local Connectivity:**
   ```powershell
   curl -X GET "localhost:9200/"
   ```

3. **Test Remote Connectivity:**
   ```powershell
   Test-NetConnection -ComputerName your-es-server -Port 9200
   ```

## Memory Issues

### Issue 5: Insufficient Memory Allocation

**Symptoms:**
- OutOfMemoryError in Elasticsearch logs
- Poor performance or slow queries
- Node becomes unresponsive

**Troubleshooting Steps:**

1. **Check Current Memory Usage:**
   ```powershell
   Get-WmiObject -Class Win32_PhysicalMemory | Measure-Object -Property Capacity -Sum
   ```

2. **Review JVM Heap Settings:**
   - Edit `C:\elastic\elasticsearch\config\jvm.options` file:
   ```
   # Recommended: Set Xms and Xmx to same value
   # Example for system with 8GB+ RAM:
   -Xms4g
   -Xmx4g
   ```
   > [!IMPORTANT]
   > Rule of thumb: Set heap to 50% of available RAM, maximum 32GB. Monitor current memory usage before making changes.

3. **Monitor Memory Usage:**
   ```curl
   curl -X GET "localhost:9200/_nodes/stats/jvm?pretty"
   ```

4. **Check for Memory Leaks:**
   - Monitor heap usage over time
   - Look for continuously increasing memory consumption
   - Review application logs for memory-related warnings

### Issue 6: Virtual Memory Configuration

**Symptoms:**
- "max virtual memory areas vm.max_map_count [65530] is too low" error
- Bootstrap check failures

**Troubleshooting Steps:**

1. **For Windows Systems:**
   - This is typically a Linux-specific issue, but if running in WSL or containers:
   - Increase virtual memory in WSL configuration

2. **Alternative Solutions:**
   - Add to `C:\elastic\elasticsearch\config\elasticsearch.yml`:
   ```yaml
   bootstrap.memory_lock: false
   ```
   - Or configure system memory settings appropriately

## Authentication Issues

### Issue 7: API Key Expiration

**Symptoms:**
- Authentication failures after previously working
- "invalid API key" errors in logs
- 401 Unauthorized responses

**Troubleshooting Steps:**

1. **Check API Key Status:**
   ```curl
   curl -X GET "localhost:9200/_security/api_key" \
        -H "Authorization: ApiKey <your-api-key>"
   ```

2. **Create New API Key:**
   ```curl
   curl -X POST "localhost:9200/_security/api_key" \
        -H "Content-Type: application/json" \
        -u elastic:password \
        -d '{
          "name": "relativity-api-key",
          "expiration": "365d"
        }'
   ```

3. **Update Application Configuration:**
   - Update Relativity configuration with new API key
   - Restart affected services

### Issue 8: Username/Password Authentication Problems

**Symptoms:**
- Login failures
- "authentication failed" errors
- Cannot access Elasticsearch with credentials

**Troubleshooting Steps:**

1. **Verify User Exists:**
   ```curl
   curl -X GET "localhost:9200/_security/user/username" \
        -u elastic:password
   ```

2. **Reset Password:**
   ```powershell
   # Navigate to Elasticsearch bin directory
   .\elasticsearch-reset-password.bat -u elastic
   ```

3. **Check User Roles:**
   ```curl
   curl -X GET "localhost:9200/_security/user/username" \
        -u elastic:password
   ```

4. **Verify Security Configuration:**
   - Check `C:\elastic\elasticsearch\config\elasticsearch.yml`:
   ```yaml
   xpack.security.enabled: true
   xpack.security.authc.api_key.enabled: true
   ```

### Issue 9: SSL/TLS Certificate Issues

**Symptoms:**
- Certificate validation errors
- "unable to verify certificate" warnings
- HTTPS connection failures

**Troubleshooting Steps:**

1. **Check Certificate Validity:**
   ```powershell
   openssl x509 -in certificate.crt -text -noout
   ```

2. **Verify Certificate Chain:**
   - Ensure all intermediate certificates are included
   - Check certificate expiration dates

3. **Trust Certificate in Windows:**
   - Import certificate to Trusted Root Certification Authorities
   - Use Certificate Manager (certmgr.msc)

## Service Verification

### Issue 10: Verifying Elasticsearch Health

**Symptoms:**
- Uncertainty about cluster status
- Need to confirm proper operation
- Performance monitoring

**Troubleshooting Steps:**

1. **Check Cluster Health:**
   ```curl
   curl -X GET "localhost:9200/_cluster/health?pretty"
   ```
   
   Expected response for healthy cluster:
   ```json
   {
     "cluster_name": "elasticsearch",
     "status": "green",
     "timed_out": false,
     "number_of_nodes": 3,
     "number_of_data_nodes": 3
   }
   ```

2. **Verify Node Status:**
   ```curl
   curl -X GET "localhost:9200/_cat/nodes?v"
   ```

3. **Check Index Health:**
   ```curl
   curl -X GET "localhost:9200/_cat/indices?v"
   ```

4. **Monitor Performance:**
   ```curl
   curl -X GET "localhost:9200/_nodes/stats?pretty"
   ```

### Issue 11: Service Dependencies Verification

**Symptoms:**
- Elasticsearch starts but related services fail
- Integration issues with Relativity components

**Troubleshooting Steps:**

1. **Verify Required Services:**
   ```powershell
   Get-Service -Name elasticsearch, kibana, apm-server | Format-Table -AutoSize
   ```

2. **Check Service Dependencies:**
   - Ensure Elasticsearch starts before dependent services
   - Verify network connectivity between services

3. **Test API Connectivity:**
   ```powershell
   # Test from Relativity server
   Invoke-RestMethod -Uri "https://your-es-server:9200/" -Method GET
   ```

4. **Validate Configuration:**
   - Review Relativity configuration for correct Elasticsearch endpoints
   - Verify authentication credentials are current
   - Check SSL certificate configuration

## Additional Diagnostic Commands

### General Health Checks

```powershell
# Check service status
Get-Service elasticsearch | Select-Object Status, StartType, Name

# Check process information
Get-Process -Name java | Where-Object {$_.ProcessName -eq "java"}

# Check listening ports
Get-NetTCPConnection | Where-Object {$_.LocalPort -in @(9200,9300)} | Select-Object LocalAddress, LocalPort, State

# Check disk space
Get-WmiObject -Class Win32_LogicalDisk | Select-Object DeviceID, @{Name="Size(GB)";Expression={[math]::Round($_.Size/1GB,2)}}, @{Name="FreeSpace(GB)";Expression={[math]::Round($_.FreeSpace/1GB,2)}}
```

### Log Analysis

```powershell
# View recent Elasticsearch logs
Get-Content "C:\elastic\elasticsearch\logs\elasticsearch.log" -Tail 50

# Search for specific errors
Select-String -Path "C:\elastic\elasticsearch\logs\*.log" -Pattern "ERROR|WARN" | Select-Object -Last 20
```


