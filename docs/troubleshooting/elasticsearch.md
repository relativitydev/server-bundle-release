# Elasticsearch Troubleshooting

This document provides troubleshooting guidance for common Elasticsearch issues encountered during installation, configuration, and operation in Relativity Server environments.

> [!NOTE]
> This guide assumes a default Elasticsearch installation path of `C:\elastic\elasticsearch`. Adjust paths according to your actual installation directory.

## Table of Contents

- [Windows Service Issues](#windows-service-issues)
  - [Elasticsearch Service Not Starting](#elasticsearch-service-not-starting)
  - [Service Crashes or Stops Unexpectedly](#service-crashes-or-stops-unexpectedly)
- [Port Configuration Issues](#port-configuration-issues)
  - [Port Conflicts](#port-conflicts)
  - [Network Connectivity Problems](#network-connectivity-problems)
- [Memory Issues](#memory-issues)
  - [Insufficient Memory Allocation](#insufficient-memory-allocation)
- [Authentication Issues](#authentication-issues)
  - [Username/Password Authentication Problems](#usernamepassword-authentication-problems)
  - [SSL/TTLS Certificate Issues](#ssltls-certificate-issues)
- [Service Verification](#service-verification)
  - [Verifying Elasticsearch Health](#verifying-elasticsearch-health)
  - [Service Dependencies Verification](#service-dependencies-verification)
- [Additional Diagnostic Commands](#additional-diagnostic-commands)

## Windows Service Issues

### Elasticsearch Service Not Starting

**Symptoms:**
- Elasticsearch service fails to start
- Service stops immediately after starting
- Error messages in Elasticsearch logs

**Troubleshooting Steps:**

- **Check Service Status:**
   ```powershell
   Get-Service -Name elasticsearch
   ```

- **Verify Service Configuration:**
   ```powershell
   (Get-CimInstance Win32_Service -Filter "Name = 'elasticsearch'").StartName
   ```

- **Check Elasticsearch Logs:**
   - Navigate to `C:\elastic\elasticsearch\logs\`
   - Review the cluster log file (`elasticsearch.log`) for error messages
   - Check the slow logs and garbage collection logs if present
   - Look for Java heap space issues or configuration errors
   
> [!TIP]
> For detailed logging information, refer to the [official Elasticsearch logging documentation](https://www.elastic.co/guide/en/elasticsearch/reference/8.17/logging.html)

- **Verify Java Installation:**
   ```powershell
   java -version
   ```
   - Elasticsearch includes a bundled Java runtime, so a separate Java installation is not required.
   - If the `JAVA_HOME` environment variable is defined, Elasticsearch will use the specified Java version instead of the bundled one.
   - If you want to use a specific Java version, ensure `JAVA_HOME` is set correctly.

- **Start Service Manually:**
   ```powershell
   Start-Service elasticsearch
   ```

### Service Crashes or Stops Unexpectedly

**Symptoms:**
- Elasticsearch service starts but stops after a short period
- Service status shows "Stopped" unexpectedly

**Troubleshooting Steps:**

1. **Check Elasticsearch Logs:**
   - Navigate to `C:\elastic\elasticsearch\logs\`
   - Review the cluster log file (`elasticsearch.log`) for errors
   - Look for OutOfMemoryError or service crash indicators

3. **Verify Disk Space:**
   - Ensure sufficient disk space on data and log directories (minimum 15% free)
   - Verify data and log files are on separate drives from the Operating System drive
   ```powershell
   # Check disk space
   Get-WmiObject -Class Win32_LogicalDisk | Select-Object DeviceID, @{Name="FreeSpace(%)";Expression={[math]::Round(($_.FreeSpace/$_.Size)*100,2)}}
   ```

## Port Configuration Issues

### Port Conflicts

**Symptoms:**
- Elasticsearch fails to bind to default ports
- "Address already in use" errors in logs
- Cannot access Elasticsearch via HTTP/HTTPS

> [!NOTE]
> Click here to view the latest port diagram that includes the Elastic Stack [Port Diagram](../environment-watch/port-diagram.md)

**Troubleshooting Steps:**

- **Check Default Ports:**
   - HTTP: 9200
   - Transport: 9300
   - Verify these ports are available:
   ```powershell
   netstat -an | findstr ":9200"
   netstat -an | findstr ":9300"
   ```

- **Test Elasticsearch Connectivity:**
   ```bash
   # Test Elasticsearch HTTP port
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/"
   ```

- **Identify Port Conflicts:**
   ```powershell
   Get-NetTCPConnection -LocalPort 9200 -State Listen
   Get-NetTCPConnection -LocalPort 9300 -State Listen
   ```


- **Update Firewall Rules:**
   ```powershell
   New-NetFirewallRule -DisplayName "Elasticsearch HTTP" -Direction Inbound -Protocol TCP -LocalPort 9200 -Action Allow
   New-NetFirewallRule -DisplayName "Elasticsearch Transport" -Direction Inbound -Protocol TCP -LocalPort 9300 -Action Allow
   ```

### Network Connectivity Problems

**Symptoms:**
- Cannot connect to Elasticsearch from remote hosts
- Connection timeouts
- "No route to host" errors

**Troubleshooting Steps:**

- **Verify Network Binding:**
   - Check `C:\elastic\elasticsearch\config\elasticsearch.yml` configuration:
   ```yaml
   network.host: 0.0.0.0  # For all interfaces
   # or
   network.host: ["_local_", "_site_"]
   ```

- **Test Local Connectivity:**
   ```bash
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/"
   ```

- **Test Remote Connectivity:**
   ```powershell
   Test-NetConnection -ComputerName <hostname_or_ip> -Port 9200
   ```

## Memory Issues

### Insufficient Memory Allocation

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
   - If the file does not exist, create it.
   ```
   # Recommended: Set Xms and Xmx to same value
   # Example for system with 8GB+ RAM:
   -Xms4g
   -Xmx4g
   ```
> [!IMPORTANT]
> Rule of thumb: Set heap to 50% of available RAM, maximum 32GB. Monitor current memory usage before making changes.

- **Monitor Memory Usage:**
   ```bash
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/_nodes/stats/jvm?pretty"
   ```

4. **Check for Memory Leaks:**
   - Monitor heap usage over time
   - Look for continuously increasing memory consumption
   - Review application logs for memory-related warnings

## Authentication Issues

### Username/Password Authentication Problems

**Symptoms:**
- Login failures
- "authentication failed" errors
- Cannot access Elasticsearch with credentials

**Troubleshooting Steps:**

- **Verify User Exists:**
   ```bash
   curl.exe -k -X GET "https://<hostname_or_ip>:9200/_security/user/<username>" \
        -u <username>:<password>
   ```

- **Reset Password:**
   ```powershell
   # Navigate to Elasticsearch bin directory
   C:\elastic\elasticsearch\bin\elasticsearch-reset-password.bat -u elastic
   ```

- **Check User Roles:**
   ```bash
   curl.exe -k -X GET "https://<hostname_or_ip>:9200/_security/user/<username>" \
        -u <username>:<password>
   ```

- **Verify Security Configuration:**
   - Check `C:\elastic\elasticsearch\config\elasticsearch.yml`:
   ```yaml
   xpack.security.enabled: true
   ```


## Service Verification

### Verifying Elasticsearch Health

**Symptoms:**
- Uncertainty about cluster status
- Need to confirm proper operation
- Performance monitoring

**Troubleshooting Steps:**

- **Check Cluster Health:**
   ```bash
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/_cluster/health?pretty"
   ```
   
   <details>
   <summary>Expected response for healthy cluster</summary>

   ```json
   {
     "cluster_name": "elasticsearch",
     "status": "green",
     "timed_out": false,
     "number_of_nodes": 3,
     "number_of_data_nodes": 3
   }
   ```
   </details>

- **Verify Node Status:**
   ```bash
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/_cat/nodes?v"
   ```

- **Check Index Health:**
   ```bash
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/_cat/indices?v"
   ```

- **Monitor Performance:**
   ```bash
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/_nodes/stats?pretty"
   ```

### Service Dependencies Verification

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
   ```bash
   # Test from Relativity server
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/"
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


