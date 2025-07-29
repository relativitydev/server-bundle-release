# Relativity Server CLI Troubleshooting

This document provides troubleshooting guidance for common Relativity Server CLI issues encountered during Environment Watch and Data Grid Audit setup, configuration, and operation.

> [!NOTE]
> This guide assumes the Relativity Server bundle was extracted to `C:\server-bundle` or a similar directory chosen by the user.

## Table of Contents

- [Elastic APM Integration Package Issues](#elastic-apm-integration-package-issues)
- [Data View Configuration Issues](#data-view-configuration-issues)
- [Kibana Encryption Keys Issues](#kibana-encryption-keys-issues)
- [Prerequisite Access Verification](#prerequisite-access-verification)

---

## Elastic APM Integration Package Issues

The Elastic APM integration package must be installed and configured before running the CLI setup.  
If you encounter errors such as "Package not found" or installation timeouts during APM integration package installation, refer to the official [Elastic APM Integration Setup Guide](../elasticsearch_setup_development.md#elastic-apm-integration-package).

To verify connectivity, always use the following format for verification commands:
```powershell
curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/"
```
<details>
<summary>Expected Output</summary>

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

---

## Data View Configuration Issues

Data views must be created and managed through the Kibana frontend.  
For step-by-step instructions, see [Data View Setup](../elasticsearch_setup_development.md#data-view-setup).

---

## Kibana Encryption Keys Issues

Kibana encryption keys must be added to `C:\elastic\kibana\config\kibana.yml` before running CLI setup.  
**If encryption keys are missing or invalid, the CLI will display errors such as:**
```
[ERROR] Missing required Kibana encryption key: xpack.encryptedSavedObjects.encryptionKey
[ERROR] Missing required Kibana encryption key: xpack.reporting.encryptionKey
[ERROR] Missing required Kibana encryption key: xpack.security.encryptionKey
```
If you encounter encryption key validation errors or warnings in the CLI, follow the instructions in [Kibana Encryption Key Configuration](kibana.md#missing-or-invalid-encryption-keys).

---

## Prerequisite Access Verification

Before running the CLI, you must have access to all of the following:

- **Relativity Admin account**  
  To verify Relativity admin account credentials, run:
  ```powershell
  curl.exe -k -u <username>:<password> -X GET "https://<hostname_or_ip>/Relativity.REST/API/Relativity.Services.InstanceDetails.IInstanceDetailsModule/InstanceDetailsService/GetRelativityVersionAsync"
  ```
  <details>
  <summary>Expected Output</summary>

  ```json
  {
    "Version": "24.0.0.0",
    ...
  }
  ```
  </details>

- **Secret Server**  
  See [Secret Store Troubleshooting](../elasticsearch_setup_development.md#secret-store-access-verification) for verification steps.

- **Kepler (SSL certificate)**  
  See [Kepler Troubleshooting](../elasticsearch_setup_development.md#kepler-ssl-certificate-access) for verification steps.

- **Elasticsearch**  
  See [Elasticsearch Troubleshooting](elasticsearch.md) for verification steps.

- **Kibana**  
  See [Kibana Troubleshooting](kibana.md) for verification steps.

- **APM Server**  
  See [APM-Server Troubleshooting](apm-server.md) for verification steps.

---

For full setup instructions, see [Relativity Server CLI Setup](relativity_server_cli_setup.md).

