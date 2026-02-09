# libsodium-64.dll: Functionality and Impact

## Overview

The `libsodium-64.dll` file is a 64-bit Windows dynamic-link library (DLL) that provides cryptographic functionality for the Environment Watch monitoring infrastructure. This document explains what this DLL is, which components depend on it, and what functionality is affected if it is missing or corrupted.

## What is libsodium?

**libsodium** is a modern, portable, and easy-to-use cryptography library. It provides:

- **Encryption and Decryption**: Secure data transmission and storage
- **Digital Signatures**: Authentication and integrity verification
- **Password Hashing**: Secure credential storage
- **Key Derivation**: Generating cryptographic keys from passwords
- **Random Data Generation**: Cryptographically secure random numbers
- **Public-Key Cryptography**: Secure key exchange and authentication

libsodium is a fork of the NaCl (Networking and Cryptography Library) project and is widely adopted for its focus on security, performance, and ease of use.

## Components That Use libsodium-64.dll

In the Relativity Environment Watch infrastructure, `libsodium-64.dll` is primarily used by:

### 1. OpenTelemetry Collector (otelcol-relativity.exe)

The OpenTelemetry Collector is the core component responsible for collecting, processing, and exporting telemetry data (metrics, logs, and traces) from each Relativity server to the Elastic Stack. The collector uses libsodium for:

- **Secure Communication**: Encrypting data transmitted to Elasticsearch and APM Server
- **TLS/SSL Operations**: Supporting secure HTTPS connections to Elastic Stack endpoints
- **Authentication**: Managing secure tokens and credentials
- **Certificate Validation**: Verifying X.509 certificates for trusted communication

### 2. Relativity Environment Watch Service (rel-envwatch-service.exe)

The Environment Watch Windows service manages the OpenTelemetry Collector and may use libsodium for:

- **Secret Storage**: Securely handling credentials and API keys
- **Configuration Encryption**: Protecting sensitive configuration data
- **Secure Inter-Process Communication**: Communicating with other Relativity services

## Affected Functionality

If `libsodium-64.dll` is missing, corrupted, or incompatible, the following functionality will be affected:

### Environment Watch Monitoring (Primary Impact)

- **Metric Collection Failures**: Host infrastructure metrics (CPU, RAM, Disk, Network) will not be collected or sent to Elasticsearch
- **Dashboard Data Loss**: Kibana dashboards will show no data or stale data for affected hosts:
  - [Relativity] Host Infrastructure Overview
  - [Relativity] Monitoring Agent
  - [Relativity] Windows Services
  - [Relativity] IIS Performance
  - [Relativity] SQL Server Performance
  - All custom monitoring dashboards

### Relativity Alerts (Secondary Impact)

- **Alert Evaluation Failures**: Relativity Alerts that depend on Environment Watch metrics will not trigger
- **Notification Failures**: Alert notifications may fail if they rely on secure communication channels

### Data Grid Audit (Minimal Impact)

- **Audit Log Transmission**: If Data Grid Audit uses the same monitoring infrastructure for health metrics, those metrics may be affected
- **Core Audit Functionality**: Data Grid Audit's primary audit logging functionality should not be directly affected, as it uses Elasticsearch's native indexing APIs

### Distributed Tracing

- **APM Data Loss**: Application Performance Monitoring (APM) traces from Relativity services will not be collected
- **Performance Analysis**: Performance troubleshooting and analysis capabilities will be unavailable

## Symptoms of Missing or Corrupted libsodium-64.dll

If `libsodium-64.dll` is missing or not functioning correctly, you may observe:

### Service and Process Symptoms

1. **OpenTelemetry Collector Fails to Start**:
   - `otelcol-relativity.exe` process is not running in Task Manager
   - Port 4318 is not listening

2. **Environment Watch Service Errors**:
   - "Relativity Environment Watch" Windows service status shows errors or repeatedly restarts
   - Windows Event Viewer shows errors in the `Relativity.EnvironmentWatch` log

3. **Missing "Everything is ready" Message**:
   - The critical log message "Everything is ready" does not appear in:
     - `otelcol-relativity-stderr.log`
     - `otelcol-relativity-stdout.log`
     - Windows Event Viewer → Relativity.EnvironmentWatch

### Error Messages

You may see error messages such as:

```
Error loading libsodium-64.dll
The specified module could not be found
```

Or:

```
Failed to initialize cryptographic library
Secure connection to Elasticsearch failed
TLS handshake error
Certificate validation failed
```

### Dashboard and Data Symptoms

1. **No Recent Data in Kibana**:
   - "Last Check-In" timestamp in the **Monitoring Agents** dashboard is outdated or missing
   - Host Infrastructure Overview shows no CPU, RAM, or Disk metrics

2. **Discover Query Returns No Results**:
   ```
   service.name: "relsvr_infrawatch_agent" and host.hostname: "<hostname>"
   ```
   Returns no data or outdated data

## Troubleshooting libsodium-64.dll Issues

### Step 1: Verify libsodium-64.dll is Present

1. Check if the DLL exists in the installation directory:
   ```powershell
   Test-Path "C:\Program Files\Relativity\EnvironmentWatch\libsodium-64.dll"
   ```

   Or check in the OpenTelemetry Collector directory:
   ```powershell
   Get-ChildItem "C:\Program Files\Relativity\EnvironmentWatch" -Recurse -Filter "libsodium*.dll"
   ```

2. If the file is missing, the Environment Watch installer package may be corrupted or incomplete.

### Step 2: Verify DLL Version and Integrity

1. Check the DLL properties:
   - Right-click the DLL file → Properties → Details
   - Verify the file version matches the expected version for your Environment Watch release

2. Check the file size and hash:
   ```powershell
   Get-FileHash "C:\Program Files\Relativity\EnvironmentWatch\libsodium-64.dll" -Algorithm SHA256
   ```

### Step 3: Reinstall Environment Watch Monitoring Agent

If the DLL is missing or corrupted, reinstall the monitoring agent:

1. Uninstall the existing Environment Watch monitoring agent:
   - Go to **Control Panel** → **Programs and Features**
   - Uninstall "Relativity Environment Watch"

2. Reinstall from the Server Bundle:
   - Run `Relativity.EnvironmentWatch.Installer.xx.x.xxxx.exe`
   - Follow the [installation instructions](../elastic-stack-setup-02-environment-watch/ew-01-install-monitoring-agents.md)

3. Verify the DLL is present after installation

### Step 4: Check for Conflicting or Outdated Versions

1. Search for other copies of libsodium DLLs on the system:
   ```powershell
   Get-ChildItem "C:\" -Recurse -Filter "libsodium*.dll" -ErrorAction SilentlyContinue
   ```

2. If multiple versions exist, ensure the correct version is in the application directory or that the PATH environment variable points to the correct location

3. Check if other applications on the server have installed conflicting versions of libsodium

### Step 5: Verify Windows System Dependencies

1. Ensure the Visual C++ Redistributable is installed:
   - libsodium-64.dll may depend on Microsoft Visual C++ Redistributable packages
   - Download and install the latest supported Visual C++ redistributable from Microsoft

2. Check Windows system DLLs:
   ```powershell
   sfc /scannow
   ```

### Step 6: Check Windows Event Logs

1. Open Event Viewer and check for DLL loading errors:
   ```powershell
   Get-EventLog Application | Where-Object { $_.Message -like "*libsodium*" }
   ```

2. Review the Relativity.EnvironmentWatch event log:
   ```powershell
   Get-EventLog Relativity.EnvironmentWatch | Select-Object -First 20
   ```

### Step 7: Verify File Permissions

1. Ensure the Relativity Service Account has read and execute permissions on the DLL:
   ```powershell
   Get-Acl "C:\Program Files\Relativity\EnvironmentWatch\libsodium-64.dll" | Format-List
   ```

2. If permissions are incorrect, grant the service account appropriate access

## Security Considerations

### Keep libsodium Updated

- Always use the version of `libsodium-64.dll` provided with the Relativity Environment Watch installer
- Do not manually replace or update the DLL unless directed by Relativity support
- Outdated versions may contain security vulnerabilities

### Protect the DLL from Tampering

- Ensure the Environment Watch installation directory has appropriate file system permissions
- Only administrators and the Relativity Service Account should have write access
- Consider using file integrity monitoring to detect unauthorized changes

### Antivirus and Security Software

- Some antivirus or endpoint protection software may quarantine or block `libsodium-64.dll`
- Add the Environment Watch installation directory to your security software's exclusion list if necessary
- Verify the DLL's digital signature to confirm it's legitimate

## Related Documentation

- [Environment Watch Monitoring Agent Installation](../elastic-stack-setup-02-environment-watch/ew-01-install-monitoring-agents.md)
- [Monitoring Agent and OpenTelemetry Collector Troubleshooting](monitoring-agent-and-otel-collector.md)
- [Environment Watch Product Overview](../environment_watch_product_overview.md)
- [Pre-requisite Troubleshooting](pre-requisite-troubleshooting.md)

## Additional Resources

- [libsodium Official Documentation](https://doc.libsodium.org/)
- [OpenTelemetry Collector Documentation](https://opentelemetry.io/docs/collector/)
- [Elastic APM OpenTelemetry Integration](https://www.elastic.co/guide/en/apm/guide/current/open-telemetry.html)

---

**Note**: If you continue to experience issues after following these troubleshooting steps, contact Relativity Support with the following information:
- Environment Watch installer version
- Windows Server version and patch level
- Event Viewer logs from Relativity.EnvironmentWatch
- OpenTelemetry Collector log files from `C:\ProgramData\Relativity\EnvironmentWatch\Services\InfraWatchAgent\Logs`
