# Certificates Configuration

This section describes the configuration of certificates in the `environmentWatchConfiguration` JSON object for monitoring certificates.

---

## Overview

Monitors the presence and validity of specified certificates in Windows certificate stores. There are certificates by default monitored without configuration and they are based on installed product.

**Default Certificates**
| Certificate Name                  | Description                                      |
|-----------------------------------|--------------------------------------------------|
| Relativity Secret Store           | certificate for Relativity Secret Store.         |

**Properties In Custom JSON Related to Certificates**

| Property       | Type     | Description                                                      |
|----------------|----------|------------------------------------------------------------------|
| `enabled`      | boolean  | Enables or disables monitoring for certificates.                 |
| `include`      | array    | List of certificate objects to monitor.                          |
| `StoreName`    | string   | Name of the certificate store (e.g., `"My"`).                    |
| `StoreLocation`| string   | Location of the store (e.g., `"LocalMachine"`).                  |
| `Thumbprint`   | string   | certificate thumbprint to identify the certificate.              |

#### StoreLocation Enum Values

The `StoreLocation` field specifies the location of the X.509 certificate store to use.

**Possible Values**

| Value         | Description                                                    |
|---------------|----------------------------------------------------------------|
| CurrentUser   | The X.509 certificate store is located in the current user's profile. |
| LocalMachine  | The X.509 certificate store is located in the local computer's profile. |

#### StoreName Enum Values

The `StoreName` field specifies the name of the Windows certificate store where the X.509 certificate is located.

**Possible Values**

| Value                | Description                                   |
|----------------------|-----------------------------------------------|
| AddressBook          | Other people                                  |
| AuthRoot             | Third party trusted roots                     |
| CertificateAuthority | Intermediate CAs                              |
| Disallowed           | Revoked certificates                          |
| My                   | Personal certificates                         |
| Root                 | Trusted root CAs                              |
| TrustedPeople        | Trusted people (used in EFS)                  |
| TrustedPublisher     | Trusted publishers (used in Authenticode)     |

**Get Certificate Thumbprint**

Depending on the Store Location and Store Name, run the following command on the host. For `LocalMachine` and `My`, use:

```powershell
Get-ChildItem Cert:\LocalMachine\My
```

The command outputs a table with `Thumbprint` and `Subject`. Select the `Thumbprint` for the required certificate and assign it as shown in the example. Adjust the command based on the `StoreName` and `StoreLocation`, and update the values in the JSON accordingly.


## Configure Certificates

certificates can be monitored by "**hosts**", "**instance**", or "**installedProducts**". For certificates to monitor, locate "**certificates**" under the desired section and update the configuration as below.

- `enabled` : Set to `true` to enable certificate monitoring.
- When configuring the `include` array, each certificate object must specify the `StoreName`, `StoreLocation`, and `Thumbprint` fields.

**Example 1**: Monitoring one certificate where `StoreName` is `My`, `StoreLocation` is `LocalMachine`, and `Thumbprint` is obtained from the following PowerShell command `Get-ChildItem Cert:\LocalMachine\My`:

```json
{
	"certificates": {
		"enabled": true,
		"include": [
			{
				"StoreName": "My",
				"StoreLocation": "LocalMachine",
				"Thumbprint": "005501F9BA68A2ED7D9BD515B256F6298AEF7E5A"
			},
			{
				"StoreName": "My",
				"StoreLocation": "LocalMachine",
				"Thumbprint": "E62D7D4DD8D054072A7A58A577D500753A586C75"
			}
		]
	}
}
```

### Verification in Kibana

- Navigate to Kibana Discover.
- Select `logs-*` Data View.
- Search for "The Environment Watch shared configuration object is not empty" which indicates that the EW Windows Service fetching values from the Custom JSON configuration successfully.

![](/resources/custom-json-images/environment-watch-shared-settings-not-empty-generic.png)
- Navigate to the Kibana certificates dashboard.
- Ensure that the certificates defined in the custom JSON configuration appear on the Kibana certificates dashboard. The example below demonstrates how a certificate specified in the custom JSON is successfully monitored and displayed on the certificates dashboard.

![](/resources/custom-json-images/certificate-json-example.png)

![](/resources/custom-json-images/certificate-dashboard-example.png)