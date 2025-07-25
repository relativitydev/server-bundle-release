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
- [Authentication Issues](#authentication-issues)
  - [Username/Password Authentication Issues](#usernamepassword-authentication-issues)
- [Kibana Encryption Keys Configuration](#kibana-encryption-keys-configuration)
  - [Missing or Invalid Encryption Keys](#missing-or-invalid-encryption-keys)
- [Service Verification](#service-verification)
  - [Verifying Kibana Health and Status](#verifying-kibana-health-and-status)
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
   **Expected output:**
   ```
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

4. **Check Memory Usage:**
   - Monitor Kibana process memory consumption using Task Manager or Resource Monitor.
   - Verify sufficient system memory is available.

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
   curl.exe -k -u <username>:<password> -X GET "http://<hostname_or_ip>:5601/"
   ```
   **Expected output:**
   ```
   <!DOCTYPE html>
   <html>
   <head>
     <title>Kibana</title>
     ...
   </head>
   <body>
     ...
   </body>
   </html>
   ```

- **Configure Alternative Port (if needed):**
> [!WARNING]
> At present, you cannot use alternative ports without introducing several side effects. If you must use a different port, ensure the other application using port 5601 is removed before changing the port.
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

- **Test Local and Remote Access:**
   ```bash
   curl.exe -k -u <username>:<password> -X GET "http://<hostname_or_ip>:5601/"
   ```
   **Expected output:**
   ```
   <!DOCTYPE html>
   <html>
   <head>
     <title>Kibana</title>
     ...
   </head>
   <body>
     ...
   </body>
   </html>
   ```
   - Run this command from the server hosting Kibana (local) and from a remote machine (remote).

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

2. **Test User Authentication:**
   ```bash
   curl.exe -k -X GET "https://<hostname_or_ip>:9200/_security/user/kibana_system" -u <username>:<password>
   ```
   **Expected output:**
   ```
   {
     "kibana_system": {
       "username": "kibana_system",
       "roles": [
         "kibana_system"
       ],
       ...
     }
   }
   ```

3. **Reset Kibana System User Password:**
   ```powershell
   # From Elasticsearch bin directory
   C:\elastic\elasticsearch\bin\elasticsearch-reset-password.bat -u kibana_system
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
   cd C:\elastic\kibana\bin
   .\kibana-encryption-keys.bat generate
   ```
   **Expected output:**
   ```
   xpack.encryptedSavedObjects.encryptionKey: "<randomly-generated-key-1>"
   xpack.reporting.encryptionKey: "<randomly-generated-key-2>"
   xpack.security.encryptionKey: "<randomly-generated-key-3>"
   ```

2. **Configure Required Encryption Keys in kibana.yml:**
   - Copy the generated lines above and paste them into your `C:\elastic\kibana\config\kibana.yml` file.

## Service Verification

### Verifying Kibana Health and Status

**Symptoms:**
- Need to confirm Kibana is operating correctly
- Performance monitoring requirements
- Health check automation

**Troubleshooting Steps:**

- **Check Kibana Status:**
   ```bash
   curl.exe -k -u <username>:<password> -X GET "http://<hostname_or_ip>:5601/api/status"
   ```
   **Expected output:**
   ```
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

## Additional Diagnostic Commands

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

