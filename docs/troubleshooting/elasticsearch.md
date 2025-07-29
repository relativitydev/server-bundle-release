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
   <details>
   <summary>Expected output</summary>

   ```
   Status   Name           DisplayName
   ------   ----           -----------
   Running  elasticsearch  elasticsearch
   ```
   </details>

- **Verify Service Configuration:**
   ```powershell
   (Get-CimInstance Win32_Service -Filter "Name = 'elasticsearch'").StartName
   ```
   <details>
   <summary>Expected output</summary>

   ```
   LocalSystem
   ```
   </details>

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
   <details>
   <summary>Expected output</summary>

   ```
   java version "17.0.8" 2023-07-18 LTS
   Java(TM) SE Runtime Environment (build 17.0.8+9-LTS-211)
   Java HotSpot(TM) 64-Bit Server VM (build 17.0.8+9-LTS-211, mixed mode, sharing)
   ```
   </details>

> [!NOTE]
> - Elasticsearch includes a bundled Java runtime, so a separate Java installation is not required.
> - If the `JAVA_HOME` environment variable is defined, Elasticsearch will use the specified Java version instead of the bundled one.
> - If you want to use a specific Java version, ensure `JAVA_HOME` is set correctly.

- **Start Service Manually:**
   ```powershell
   Start-Service elasticsearch
   ```
   <details>
   <summary>Expected output</summary>

   ```
   (No output if successful. Service status will be "Running" after execution.)
   ```
   </details>

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
   <details>
   <summary>Expected output</summary>

   ```
   DeviceID FreeSpace(%)
   -------- ------------
   C:       22.15
   D:       48.92
   ```
   </details>

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
   <details>
   <summary>Expected output</summary>

   ```
   TCP    0.0.0.0:9200           0.0.0.0:0              LISTENING
   TCP    0.0.0.0:9300           0.0.0.0:0              LISTENING
   ```
   </details>

- **Test Elasticsearch Connectivity:**
   ```powershell
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/"
   ```
   <details>
   <summary>Expected output</summary>

   ```json
   {
     "name" : "EMTTEST",
     "cluster_name" : "elasticsearch",
     "cluster_uuid" : "",
     "version" : {
       "number" : "8.17.3",
       "build_flavor" : "default",
       "build_type" : "zip",
       "build_hash" : "",
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

- **Identify Port Conflicts:**
   ```powershell
   Get-NetTCPConnection -LocalPort 9200 -State Listen
   Get-NetTCPConnection -LocalPort 9300 -State Listen
   ```
   <details>
   <summary>Expected output</summary>

   ```
   LocalAddress LocalPort State
   ----------- --------- -----
   0.0.0.0     9200      Listen
   0.0.0.0     9300      Listen
   ```
   </details>

- **Update Firewall Rules:**
   ```powershell
   New-NetFirewallRule -DisplayName "Elasticsearch HTTP" -Direction Inbound -Protocol TCP -LocalPort 9200 -Action Allow
   New-NetFirewallRule -DisplayName "Elasticsearch Transport" -Direction Inbound -Protocol TCP -LocalPort 9300 -Action Allow
   ```
   <details>
   <summary>Expected output</summary>

   ```
   (No output if successful. Rule will appear in Windows Firewall.)
   ```
   </details>

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
   ```powershell
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/"
   ```
   <details>
   <summary>Expected output</summary>

   ```json
   {
     "name" : "EMTTEST",
     "cluster_name" : "elasticsearch",
     ...
   }
   ```
   </details>

- **Test Remote Connectivity:**
   ```powershell
   Test-NetConnection -ComputerName <hostname_or_ip> -Port 9200
   ```
   <details>
   <summary>Expected output</summary>

   ```
   ComputerName     : <hostname_or_ip>
   RemoteAddress    : <ip>
   RemotePort       : 9200
   TcpTestSucceeded : True
   ```
   </details>

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
   <details>
   <summary>Expected output</summary>

   ```
   Count    : 2
   Average  :
   Sum      : 34359738368
   Property : Capacity
   ```
   </details>
> [!NOTE]
> `Sum` is the total RAM in bytes (e.g., 34359738368 bytes = 32 GB).

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
   ```powershell
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/_nodes/stats/jvm?pretty"
   ```
   <details>
   <summary>Expected output</summary>

   ```json
   {
     "nodes": {
       "node_id": {
         "jvm": {
           "mem": {
             "heap_used_in_bytes": 123456789,
             "heap_committed_in_bytes": 2147483648,
             "heap_max_in_bytes": 2147483648,
             ...
           }
         }
       }
     }
   }
   ```
   </details>

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
   ```powershell
   curl.exe -k -X GET "https://<hostname_or_ip>:9200/_security/user/<username>" -u <username>:<password>
   ```
   <details>
   <summary>Expected output</summary>

   ```json
   {
     "<username>": {
       "username": "<username>",
       "roles": [
         "superuser"
       ],
       ...
     }
   }
   ```
   </details>

- **Reset Password:**
   ```powershell
   C:\elastic\elasticsearch\bin\elasticsearch-reset-password.bat -u elastic
   ```
   <details>
   <summary>Expected output</summary>

   ```
   Password for the [elastic] user successfully reset.
   ```
   </details>

- **Check User Roles:**
   ```powershell
   curl.exe -k -X GET "https://<hostname_or_ip>:9200/_security/user/<username>" -u <username>:<password>
   ```
   <details>
   <summary>Expected output</summary>

   ```json
   {
     "<username>": {
       "username": "<username>",
       "roles": [
         "superuser"
       ],
       ...
     }
   }
   ```
   </details>

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
   ```powershell
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
   ```powershell
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/_cat/nodes?v"
   ```
   <details>
   <summary>Expected output</summary>

   ```
   ip         heap.percent ram.percent cpu load_1m load_5m load_15m node.role   master name
   10.0.0.1   23           55          2   0.00    0.01    0.05    cdfhilmrstw *      node-1
   10.0.0.2   19           60          1   0.00    0.01    0.05    cdfhilmrstw -      node-2
   ```
   </details>

- **Check Index Health:**
   ```powershell
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/_cat/indices?v"
   ```
   <details>
   <summary>Expected output</summary>

   ```
   health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
   green  open   myindex  1a2b3c4d5e6f7g8h9i0j   1   1   1000       0            1.2mb      600kb
   ```
   </details>

- **Monitor Performance:**
   ```powershell
   curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/_nodes/stats?pretty"
   ```
   <details>
   <summary>Expected output</summary>

   ```json
   {
     "nodes": {
       "node_id": {
         "name": "node-1",
         "host": "10.0.0.1",
         ...
       }
     }
   }
   ```
   </details>

### Service Dependencies Verification

**Symptoms:**
- Elasticsearch starts but related services fail

**Troubleshooting Steps:**

1. **Verify Required Services:**
   ```powershell
   Get-Service -Name elasticsearch, kibana, apm-server | Format-Table -AutoSize
   ```
   <details>
   <summary>Expected output</summary>

   ```
   Status   Name           DisplayName
   ------   ----           -----------
   Running  elasticsearch  elasticsearch
   Running  kibana         kibana
   Running  apm-server     apm-server
   ```
   </details>

## Additional Diagnostic Commands

### General Health Checks

```powershell
# Check service status
Get-Service elasticsearch | Select-Object Status, StartType, Name
```
<details>
<summary>Expected output</summary>

```
Status   StartType Name
------   --------- ----
Running  Automatic elasticsearch
```
</details>

```powershell
# Check process information
Get-Process -Name java | Where-Object {$_.ProcessName -eq "java"}
```
<details>
<summary>Expected output</summary>

```
Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
...      ...       ...        ...       ...        ... ... java
```
</details>

```powershell
# Check listening ports
Get-NetTCPConnection | Where-Object {$_.LocalPort -in @(9200,9300)} | Select-Object LocalAddress, LocalPort, State
```
<details>
<summary>Expected output</summary>

```
LocalAddress LocalPort State
----------- --------- -----
0.0.0.0     9200      Listen
0.0.0.0     9300      Listen
```
</details>

```powershell
# Check disk space
Get-WmiObject -Class Win32_LogicalDisk | Select-Object DeviceID, @{Name="Size(GB)";Expression={[math]::Round($_.Size/1GB,2)}}, @{Name="FreeSpace(GB)";Expression={[math]::Round($_.FreeSpace/1GB,2)}}
```
<details>
<summary>Expected output</summary>

```
DeviceID Size(GB) FreeSpace(GB)
-------- -------- -------------
C:       100      22.15
D:       200      48.92
```
</details>


