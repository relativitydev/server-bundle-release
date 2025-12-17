# Certificates Configuration

This section describes how to configure certificate monitoring using the `environmentWatchConfiguration` JSON object.

---

## Overview

Monitors the presence and validity of specified certificates in Windows certificate stores. By default, the Relativity Secret Store certificate is monitored without requiring additional configuration. Other certificates can be added based on the installed product or specific requirements.

**Default Certificates**
| Certificate Name                  | Description                                      |
|-----------------------------------|--------------------------------------------------|
| Relativity Secret Store           | Certificate for Relativity Secret Store.         |

**Properties In Custom JSON Related to Certificates**

| Property       | Type     | Description                                                      |
|----------------|----------|------------------------------------------------------------------|
| `enabled`      | boolean  | Enables or disables monitoring for certificates.                 |
| `include`      | array    | List of certificate objects to monitor.                          |
| `storeName`    | string   | Name of the certificate store (e.g., `"My"`).                    |
| `storeLocation`| string   | Location of the store (e.g., `"LocalMachine"`).                  |
| `thumbprint`   | string   | Certificate thumbprint to identify the certificate.              |

#### StoreLocation Enum Values

The `storeLocation` field specifies the location of the X.509 certificate store to use.

**Possible Values**

| Value         | Description                                                    |
|---------------|----------------------------------------------------------------|
| CurrentUser   | The X.509 certificate store is located in the current user's profile. |
| LocalMachine  | The X.509 certificate store is located in the local computer's profile. |

#### StoreName Enum Values

The `storeName` field specifies the name of the Windows certificate store where the X.509 certificate is located.

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

The command returns a list of certificates including their `thumbprint` and `subject`. Copy the `thumbprint` value for the certificate you want to monitor and use it in the custom JSON configuration. Adjust the command as needed based on the selected `storeName` and `storeLocation`.

## Configure Certificates

Certificates can be monitored at the "**hosts**", "**instance**", or "**installedProducts**" level.
For certificates to monitor, locate "**certificates**" under the desired section and update the configuration as below.

- `enabled` : Set to `true` to enable certificate monitoring.
- When configuring the `include` section, specify the `storeName`, `storeLocation`, and `thumbprint` for each certificate to be monitored.

**Example 1**: Monitoring two certificates from the LocalMachine\My store. The certificate is identified by its Thumbprint, which you can retrieve using the following PowerShell command:`Get-ChildItem Cert:\LocalMachine\My`

```json
{
	"certificates": {
		"enabled": true,
		"include": [
			{
				"storeName": "My",
				"storeLocation": "LocalMachine",
				"thumbprint": "005501F9BA68A2ED7D9BD515B256F6298AEF7E5A"
			},
			{
				"storeName": "My",
				"storeLocation": "LocalMachine",
				"thumbprint": "E62D7D4DD8D054072A7A58A577D500753A586C75"
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