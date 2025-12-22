# Elasticsearch Troubleshooting

This document provides troubleshooting guidance for common Elasticsearch issues encountered during installation, configuration, and operation in Relativity Server environments.

> [!NOTE]
> This guide assumes a default Elasticsearch installation path of `C:\elastic\elasticsearch`. Adjust paths according to your actual installation directory.

## Windows service issues


> [!IMPORTANT]
> Before troubleshooting Elasticsearch, verify that all required Elastic Stack services are running. If any of these are not running, other troubleshooting steps may be irrelevant.
>
> **Check all required services:**
> ```powershell
> Get-Service -Name elasticsearch-service-x64, kibana, apm-server | Format-Table -AutoSize
> ```
> **OR**
> ```powershell
> Get-Service -Name elasticsearch, kibana, apm-server | Format-Table -AutoSize
> ```
> Expected output:
>
> ```
> Status   Name           DisplayName
> ------   ----           -----------
> Running  elasticsearch  elasticsearch
> Running  kibana         kibana
> Running  apm-server     apm-server
> ```
>
> - The `kibana` Windows service may not exist if Kibana was not installed as a Windows service (e.g., via NSSM). NSSM is not required; Kibana can be run manually or as a scheduled task. Only check for the `kibana` service if you have installed it as a service.
> - If Kibana is not in running state, [click here for Kibana troubleshooting](../troubleshooting/kibana.md).
> - If APM is not in running state, [click here for APM troubleshooting](apm-server.md)

### Elasticsearch service not starting

**Symptoms:**
- Elasticsearch service fails to start
- Service stops immediately after starting
- Error messages in Elasticsearch logs

**Troubleshooting Steps:**

1. Check Service Status:
   ```powershell
   Get-Service -Name elasticsearch-service-x64 | Select-Object Status, StartType, Name
   ```
   **OR**
   ```powershell
   Get-Service -Name elasticsearch | Select-Object Status, StartType, Name
   ```
   
   Expected output:
   ```
   Status   StartType Name
   ------   --------- ----
   Running  Automatic elasticsearch
   ```

2. Verify Service Configuration:
   ```powershell
   (Get-CimInstance Win32_Service -Filter "Name = 'elasticsearch-service-x64'").StartName
   ```
   **OR**
   ```powershell
   (Get-CimInstance Win32_Service -Filter "Name = 'elasticsearch'").StartName
   ```
   
   Expected output:
   ```
   LocalSystem
   ```

3. Check Elasticsearch Logs:
    1. Navigate to the log directory (default: `C:\elastic\elasticsearch-{version}\logs\`).
    2. Review the Elasticsearch log file (`elasticsearch.log`) for error messages.
    3. Check the slow logs and garbage collection logs if present.
    4. For every error in the Elasticsearch log, provide troubleshooting for that specific error.

    > - For detailed logging information, refer to the [official Elasticsearch logging documentation](https://www.elastic.co/guide/en/elasticsearch/reference/8.17/logging.html)
    > - Elasticsearch includes a bundled Java runtime, so a separate Java installation is not required.
    > - If the `JAVA_HOME` environment variable is defined, Elasticsearch will use the specified Java version instead of the bundled one.
    > - If you want to use a specific Java version, ensure `JAVA_HOME` is set correctly.

4. Start Service Manually:
   ```powershell
   Start-Service elasticsearch-service-x64
   ```
   **OR**
   ```powershell
   Start-Service elasticsearch
   ```
  
   Expected output:
   ```
   (No output if successful. Service status will be "Running" after execution.)
   ```

### Service crashes or stops unexpectedly

**Symptoms:**
- Elasticsearch service starts but stops after a short period
- Service status shows "Stopped" unexpectedly

**Troubleshooting Steps:**

1. Check Elasticsearch Logs:
    1. Navigate to `C:\elastic\elasticsearch\logs\`
    2. Review the Elasticsearch log file (`elasticsearch.log`) for errors
    3. For every error in the Elasticsearch log, provide troubleshooting for that specific error.
       > Always check the latest error in the Elasticsearch log and troubleshoot accordingly. This approach should be followed everywhere.

2. Verify Disk Space:
    1. Ensure sufficient disk space on data and log directories (minimum 15% free)
    2. Verify data and log files are on separate drives from the Operating System drive. If you store Elasticsearch data on a network share, ensure the share is accessible and has sufficient free space. Some environments may not use mapped drives.
       ```powershell
       # Check disk space
       Get-WmiObject -Class Win32_LogicalDisk | Select-Object DeviceID, @{Name="FreeSpace(%)";Expression={[math]::Round(($_.FreeSpace/$_.Size)*100,2)}}
       ```
      
       Expected output:
       ```
       DeviceID FreeSpace(%)
       -------- ------------
       C:       22.15
       D:       48.92
       ```

> For port-related issues, see the [Port Configuration Troubleshooting](pre-requisite-troubleshooting.md) guide.

## Memory issues

### Insufficient memory allocation

**Symptoms:**
- OutOfMemoryError in Elasticsearch logs
- Poor performance or slow queries
- Node becomes unresponsive

**Troubleshooting Steps:**

1. Check Current Memory Usage:
   ```powershell
   Get-WmiObject -Class Win32_PhysicalMemory | Measure-Object -Property Capacity -Sum
   ```
   
   Expected output:
   ```
   Count    : 2
   Average  :
   Sum      : 34359738368
   Property : Capacity
   ```

    > `Sum` is the total RAM in bytes (e.g., 34359738368 bytes = 32 GB).

2. Review JVM Heap Settings:
    1. Edit `C:\elastic\elasticsearch\config\jvm.options` file. (If the file does not exist, create it.)
        ```
        # Recommended: Set Xms and Xmx to same value
        # Example for system with 8GB+ RAM:
        -Xms4g
        -Xmx4g
        ```

    > Set heap to 50% of available RAM, maximum 32GB. Monitor current memory usage before making changes.

3. Check for Memory Leaks:
    1. Monitor heap usage over time
    2. Look for continuously increasing memory consumption
    3. Review application logs for memory-related warnings


## Authentication issues

### Username/Password authentication issues

**Symptoms:**
- Login failures
- "authentication failed" errors
- Cannot access Elasticsearch with credentials

**Troubleshooting Steps:**

1. Verify User Exists:
   ```powershell
   curl.exe -k -X GET "https://<hostname_or_ip>:9200/_security/user/<username>" -u <username>:<password>
   ```

   Expected output:
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

2. Reset Password:
   ```powershell
   C:\elastic\elasticsearch\bin\elasticsearch-reset-password.bat -u <username>
   ```
   
   Expected output:
   ```
   Password for the [<username>] user successfully reset.
   ```  

3. Verify Security Configuration:
    1. Check `C:\elastic\elasticsearch\config\elasticsearch.yml`:
        ```yaml
        xpack.security.enabled: true
        ```
    > Also verify that the URL you are using is `https://<username>:9200/`




## Service verification

### Verifying Elasticsearch health

**Symptoms:**
- Uncertainty about cluster status
- Need to confirm proper operation
- Performance monitoring

**Troubleshooting Steps:**

1. Check Cluster Health:
    ```powershell
    curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/_cluster/health?pretty"
    ```

    Expected response for healthy cluster: :
 
    ```json
    {
      "cluster_name": "elasticsearch",
      "status": "green",
      "timed_out": false,
      "number_of_nodes": 3,
      "number_of_data_nodes": 3
    }
    ```   

2. Verify Node Status:
    ```powershell
    curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/_cat/nodes?v"
    ```

    Expected output:
    ```
    ip            heap.percent ram.percent cpu load_1m load_5m load_15m     node.role      master name
    <ip>          14           95          28                               cdfhilmrstw *  <node name>
    ```

3. Check Index Health:
    ```powershell
    curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/_cat/indices?v"
    ```
   
    Expected output:
    ```
    health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
    green  open   myindex  1a2b3c4d5e6f7g8h9i0j   1   1   1000       0            1.2mb      600kb
    ```

4. Monitor Performance:
    ```powershell
    curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/_nodes/stats?pretty"
    ```
   
    Expected output:
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




