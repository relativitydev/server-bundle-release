# Relativity Server CLI Troubleshooting

This document provides troubleshooting guidance for common Relativity Server CLI issues encountered during Environment Watch and Data Grid Audit setup, configuration, and operation.

> [!NOTE]
> This guide assumes the Relativity Server CLI is installed in the default location. Adjust paths according to your actual installation directory.

## Table of Contents

- [Elastic APM Integration Package Issues](#elastic-apm-integration-package-issues)
  - [APM Integration Package Installation Failures](#apm-integration-package-installation-failures)
  - [APM Integration Package Configuration Problems](#apm-integration-package-configuration-problems)
- [Data View Configuration Issues](#data-view-configuration-issues)
  - [Data View Must Be Triggered Through Frontend](#data-view-must-be-triggered-through-frontend)
- [Kibana Encryption Keys Issues](#kibana-encryption-keys-issues)
  - [Missing Kibana Encryption Keys](#missing-kibana-encryption-keys)
- [Prerequisite Access Verification](#prerequisite-access-verification)
  - [Relativity Admin Account Access](#relativity-admin-account-access)
  - [Secret Server Access Verification](#secret-server-access-verification)
  - [Kepler SSL Certificate Access](#kepler-ssl-certificate-access)
  - [Elasticsearch Access Verification](#elasticsearch-access-verification)
  - [Kibana Access Verification](#kibana-access-verification)
  - [APM Server Access Verification](#apm-server-access-verification)
- [Additional Diagnostic Commands](#additional-diagnostic-commands)

## Elastic APM Integration Package Issues

### APM Integration Package Installation Failures

**Symptoms:**
- CLI fails during APM integration package installation
- "Package not found" errors
- Integration package installation timeouts

**Troubleshooting Steps:**

1. **Verify Elasticsearch Connectivity:**
   ```powershell
   curl -X GET "https://your-elasticsearch-server:9200/"
   ```

2. **Check Integration Package Availability:**
   ```powershell
   # Test CLI connectivity to package repository
   .\RelativityServerCLI.exe test-elasticsearch-connection --elasticsearch-url https://your-elasticsearch:9200
   ```

3. **Manual Package Installation Verification:**
   ```curl
   curl -X GET "https://your-elasticsearch:9200/_cat/indices/.fleet-*?v"
   ```

4. **Verify Fleet Server Configuration:**
   ```curl
   curl -X GET "https://your-elasticsearch:9200/.fleet-policies/_search" -H "Content-Type: application/json"
   ```

5. **Check Package Registry Access:**
   - Ensure CLI can access Elastic Package Registry
   - Verify network connectivity to registry.elastic.co
   - Check firewall rules for outbound HTTPS connections

**Common Error Messages:**
```
Error: Failed to install APM integration package
Cause: Unable to connect to Elastic Package Registry
Solution: Verify internet connectivity and firewall settings
```

### APM Integration Package Configuration Problems

**Symptoms:**
- Package installs but configuration fails
- APM data not flowing to Elasticsearch
- Integration package shows as installed but inactive

**Troubleshooting Steps:**

1. **Verify Integration Package Status:**
   ```curl
   curl -X GET "https://your-elasticsearch:9200/.fleet-packages/_search" \
        -H "Content-Type: application/json" \
        -d '{"query": {"term": {"name": "apm"}}}'
   ```

2. **Check Integration Policy:**
   ```curl
   curl -X GET "https://your-kibana:5601/api/fleet/package_policies" \
        -H "kbn-xsrf: true" \
        -H "Content-Type: application/json"
   ```

3. **Validate APM Server Configuration:**
   ```powershell
   # Verify APM Server is receiving configuration from Fleet
   curl -X GET "http://your-apm-server:8200/config"
   ```

4. **Test Data Flow:**
   ```powershell
   # Send test APM data
   curl -X POST "http://your-apm-server:8200/intake/v2/events" \
        -H "Content-Type: application/x-ndjson" \
        -d '{"metadata":{"service":{"name":"test-service"}}}'
   ```

## Data View Configuration Issues

### Data View Must Be Triggered Through Frontend

**Symptoms:**
- CLI completes successfully but data views are not available
- Kibana dashboards show "No data view" errors
- Index patterns exist but are not properly configured as data views

**Troubleshooting Steps:**

1. **Verify Index Patterns Creation:**
   ```curl
   curl -X GET "https://your-kibana:5601/api/saved_objects/_find?type=index-pattern" \
        -H "kbn-xsrf: true"
   ```

2. **Check Data View Status:**
   ```curl
   curl -X GET "https://your-kibana:5601/api/data_views" \
        -H "kbn-xsrf: true" \
        -H "Content-Type: application/json"
   ```

3. **Manual Data View Creation via Frontend:**
   - Navigate to Kibana → Stack Management → Data Views
   - Click "Create data view"
   - Enter index pattern (e.g., `apm-*`, `logs-*`, `metrics-*`)
   - Set timestamp field (@timestamp)
   - Save data view

4. **Verify Data View Accessibility:**
   ```curl
   curl -X GET "https://your-kibana:5601/api/data_views/default" \
        -H "kbn-xsrf: true"
   ```

**CLI Configuration Note:**
```yaml
# The CLI creates index patterns but data views must be manually created
# or triggered through Kibana frontend for proper dashboard functionality
```

**Resolution Steps:**
1. Complete CLI setup process
2. Access Kibana web interface
3. Navigate to Stack Management → Data Views
4. Create data views for the following patterns:
   - `apm-*` (for APM data)
   - `logs-apm*` (for APM logs)
   - `metrics-apm*` (for APM metrics)

## Kibana Encryption Keys Issues

### Missing Kibana Encryption Keys

**Symptoms in CLI:**
- CLI displays encryption key validation errors
- "Kibana encryption configuration incomplete" warnings
- Setup process fails with encryption-related errors

**Common CLI Error Messages:**
```
ERROR: Kibana encryption keys validation failed
DETAILS: One or more required encryption keys are missing from kibana.yml
IMPACT: Saved objects, reporting, and security features will not function properly
SOLUTION: Configure encryption keys in Kibana configuration file
```

**CLI Warning Examples:**
```
WARNING: xpack.encryptedSavedObjects.encryptionKey not found in Kibana configuration
WARNING: xpack.reporting.encryptionKey not found in Kibana configuration  
WARNING: xpack.security.encryptionKey not found in Kibana configuration
```

**Troubleshooting Steps:**

1. **CLI Encryption Key Validation:**
   ```powershell
   .\RelativityServerCLI.exe validate-kibana-config --kibana-url https://your-kibana:5601
   ```

2. **Check Current Kibana Configuration:**
   ```powershell
   # Verify if encryption keys are present
   Select-String -Path "C:\kibana\config\kibana.yml" -Pattern "encryptionKey"
   ```

3. **Generate Required Encryption Keys:**
   ```powershell
   # Navigate to Kibana bin directory
   .\kibana-encryption-keys.bat generate
   ```

4. **Configure Missing Encryption Keys:**
   
   **For detailed encryption key configuration, see: [Kibana Troubleshooting - Encryption Keys](kibana.md#kibana-encryption-keys-configuration)**

   Required keys in `kibana.yml`:
   ```yaml
   # Saved Objects encryption key (required)
   xpack.encryptedSavedObjects.encryptionKey: "32-character-encryption-key"
   
   # Reporting encryption key (required for reporting features)
   xpack.reporting.encryptionKey: "32-character-encryption-key"
   
   # Security session encryption key (required for security features)
   xpack.security.encryptionKey: "32-character-encryption-key"
   ```

5. **Restart Kibana and Re-run CLI:**
   ```powershell
   Restart-Service kibana
   .\RelativityServerCLI.exe setup-environment-watch
   ```

**Error Resolution Process:**
1. When CLI reports encryption key errors, stop the setup process
2. Follow the [Kibana encryption key configuration guide](kibana.md#issue-10-missing-or-invalid-encryption-keys)
3. Generate and configure all required encryption keys
4. Restart Kibana service
5. Re-run the Relativity Server CLI setup command

## Prerequisite Access Verification

### Relativity Admin Account Access

**Symptoms:**
- CLI authentication failures with Relativity
- "Insufficient permissions" errors during setup
- Unable to create or configure Relativity applications

**Troubleshooting Steps:**

1. **Verify Admin Account Credentials:**
   ```powershell
   # Test Relativity API connectivity
   $headers = @{
       'X-CSRF-Header' = '-'
       'Content-Type' = 'application/json'
   }
   $credential = Get-Credential
   Invoke-RestMethod -Uri "https://your-relativity/Relativity.REST/API/Relativity.Services.InstanceDetails.IInstanceDetailsModule/InstanceDetailsService/GetRelativityVersionAsync" -Credential $credential -Headers $headers
   ```

2. **Check Required Permissions:**
   - System Administrator rights in Relativity
   - Application installation permissions
   - Workspace creation/modification rights

3. **Validate Account Status:**
   ```powershell
   # Check if account is enabled and not locked
   Get-ADUser -Identity "relativity-admin" -Properties Enabled, LockedOut, AccountExpirationDate
   ```

4. **Test CLI Authentication:**
   ```powershell
   .\RelativityServerCLI.exe test-relativity-connection --relativity-url https://your-relativity --username admin@domain.com
   ```

### Secret Server Access Verification

**Symptoms:**
- Cannot retrieve secrets during CLI setup
- Authentication failures with Secret Server API
- Missing or expired API tokens

**Troubleshooting Steps:**

1. **Test Secret Server Connectivity:**
   ```powershell
   Test-NetConnection -ComputerName secret-server.domain.com -Port 443
   ```

2. **Verify API Token:**
   ```powershell
   $headers = @{
       'Authorization' = 'Bearer your-secret-server-token'
       'Content-Type' = 'application/json'
   }
   try {
       Invoke-RestMethod -Uri "https://secret-server.domain.com/api/v1/secrets" -Headers $headers -Method GET
       Write-Host "Secret Server access successful"
   } catch {
       Write-Error "Secret Server access failed: $($_.Exception.Message)"
   }
   ```

3. **Check Required Permissions:**
   - Read access to Relativity secrets
   - API access permissions
   - Token expiration status

4. **CLI Secret Server Configuration:**
   ```powershell
   .\RelativityServerCLI.exe configure-secret-server --server-url https://secret-server.domain.com --token your-token
   ```

### Kepler SSL Certificate Access

**Symptoms:**
- SSL certificate validation errors during setup
- Cannot establish secure connections to Kepler
- Certificate chain validation failures

**Troubleshooting Steps:**

1. **Verify Certificate Store Access:**
   ```powershell
   Get-ChildItem -Path Cert:\LocalMachine\My | Where-Object {$_.Subject -like "*kepler*"}
   ```

2. **Test Kepler SSL Connectivity:**
   ```powershell
   Test-NetConnection -ComputerName kepler.domain.com -Port 443
   ```

3. **Validate Certificate Chain:**
   ```powershell
   $cert = Get-ChildItem -Path Cert:\LocalMachine\My | Where-Object {$_.Subject -like "*kepler*"}
   if ($cert) {
       Test-Certificate -Cert $cert -AllowUntrustedRoot
   } else {
       Write-Error "Kepler certificate not found in certificate store"
   }
   ```

4. **CLI Certificate Configuration:**
   ```powershell
   .\RelativityServerCLI.exe configure-kepler --kepler-url https://kepler.domain.com --cert-path C:\certs\kepler.crt
   ```

### Elasticsearch Access Verification

**Symptoms:**
- Cannot connect to Elasticsearch during CLI setup
- Authentication failures with Elasticsearch
- Index creation or configuration errors

**Troubleshooting Steps:**

1. **Test Elasticsearch Connectivity:**
   ```powershell
   Test-NetConnection -ComputerName elasticsearch.domain.com -Port 9200
   ```

2. **Verify Authentication:**
   ```powershell
   # Test with API key
   $headers = @{
       'Authorization' = 'ApiKey your-elasticsearch-api-key'
       'Content-Type' = 'application/json'
   }
   try {
       $response = Invoke-RestMethod -Uri "https://elasticsearch.domain.com:9200/_cluster/health" -Headers $headers
       Write-Host "Elasticsearch cluster status: $($response.status)"
   } catch {
       Write-Error "Elasticsearch access failed: $($_.Exception.Message)"
   }
   ```

3. **Check Required Indices:**
   ```curl
   curl -X GET "https://elasticsearch.domain.com:9200/_cat/indices/apm-*,logs-*,metrics-*?v"
   ```

4. **CLI Elasticsearch Testing:**
   ```powershell
   .\RelativityServerCLI.exe test-elasticsearch --elasticsearch-url https://elasticsearch.domain.com:9200 --api-key your-key
   ```

### Kibana Access Verification

**Symptoms:**
- Cannot access Kibana during dashboard setup
- Dashboard import failures
- Visualization configuration errors

**Troubleshooting Steps:**

1. **Test Kibana Connectivity:**
   ```powershell
   Test-NetConnection -ComputerName kibana.domain.com -Port 5601
   ```

2. **Verify Kibana API Access:**
   ```powershell
   try {
       $response = Invoke-WebRequest -Uri "https://kibana.domain.com:5601/api/status" -Method GET
       $status = ($response.Content | ConvertFrom-Json).status.overall.level
       Write-Host "Kibana status: $status"
   } catch {
       Write-Error "Kibana access failed: $($_.Exception.Message)"
   }
   ```

3. **Check Space Configuration:**
   ```curl
   curl -X GET "https://kibana.domain.com:5601/api/spaces/space" \
        -H "kbn-xsrf: true"
   ```

4. **CLI Kibana Testing:**
   ```powershell
   .\RelativityServerCLI.exe test-kibana --kibana-url https://kibana.domain.com:5601
   ```

### APM Server Access Verification

**Symptoms:**
- Cannot configure APM Server during setup
- APM data collection failures
- Agent configuration errors

**Troubleshooting Steps:**

1. **Test APM Server Connectivity:**
   ```powershell
   Test-NetConnection -ComputerName apm-server.domain.com -Port 8200
   ```

2. **Verify APM Server Status:**
   ```powershell
   try {
       $response = Invoke-WebRequest -Uri "https://apm-server.domain.com:8200/" -Method GET
       $apmInfo = $response.Content | ConvertFrom-Json
       Write-Host "APM Server version: $($apmInfo.version)"
   } catch {
       Write-Error "APM Server access failed: $($_.Exception.Message)"
   }
   ```

3. **Test APM Data Intake:**
   ```curl
   curl -X POST "https://apm-server.domain.com:8200/intake/v2/events" \
        -H "Authorization: Bearer your-secret-token" \
        -H "Content-Type: application/x-ndjson"
   ```

4. **CLI APM Server Testing:**
   ```powershell
   .\RelativityServerCLI.exe test-apm-server --apm-url https://apm-server.domain.com:8200 --secret-token your-token
   ```

## Additional Diagnostic Commands

### CLI-Specific Diagnostics

```powershell
# Validate complete environment configuration
.\RelativityServerCLI.exe validate-environment

# Test all prerequisite connections
.\RelativityServerCLI.exe test-prerequisites

# Check CLI configuration file
.\RelativityServerCLI.exe show-config

# Verbose logging for troubleshooting
.\RelativityServerCLI.exe setup-environment-watch --verbose --log-level debug

# Export configuration for review
.\RelativityServerCLI.exe export-config --output-file config-export.json
```

### Comprehensive Prerequisite Testing

```powershell
# Test all required system access
$prerequisites = @(
    @{Name="Relativity"; Command=".\RelativityServerCLI.exe test-relativity-connection"},
    @{Name="Secret Server"; Command=".\RelativityServerCLI.exe test-secret-server"},
    @{Name="Kepler"; Command=".\RelativityServerCLI.exe test-kepler-connection"},
    @{Name="Elasticsearch"; Command=".\RelativityServerCLI.exe test-elasticsearch-connection"},
    @{Name="Kibana"; Command=".\RelativityServerCLI.exe test-kibana-connection"},
    @{Name="APM Server"; Command=".\RelativityServerCLI.exe test-apm-server-connection"}
)

foreach ($prereq in $prerequisites) {
    Write-Host "Testing $($prereq.Name)..." -ForegroundColor Yellow
    try {
        Invoke-Expression $prereq.Command
        Write-Host "$($prereq.Name): PASS" -ForegroundColor Green
    } catch {
        Write-Host "$($prereq.Name): FAIL - $($_.Exception.Message)" -ForegroundColor Red
    }
}
```

### Configuration Validation

```powershell
# Validate Kibana encryption keys before CLI setup
$kibanaConfig = "C:\kibana\config\kibana.yml"
$requiredKeys = @(
    "xpack.encryptedSavedObjects.encryptionKey",
    "xpack.reporting.encryptionKey", 
    "xpack.security.encryptionKey"
)

foreach ($key in $requiredKeys) {
    if (Select-String -Path $kibanaConfig -Pattern $key -Quiet) {
        Write-Host "$key: CONFIGURED" -ForegroundColor Green
    } else {
        Write-Host "$key: MISSING" -ForegroundColor Red
        Write-Host "Please configure encryption keys before running CLI setup" -ForegroundColor Yellow
    }
}
```

