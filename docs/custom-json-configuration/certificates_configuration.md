# Certificates Configuration

This section describes the configuration of Certificates in the `environmentWatchConfiguration` JSON object for monitoring Certificates.

---

## Custom JSON Configuration Structure

To monitor Certificates, configure the Certificate details in a Custom JSON file. This file should be stored in the `BCPPath` directory within the `EnvironmentWatch` folder and named `environment-watch-configuration.json`. An example of the BCPPath and folder structure is shown below:

![](/resources/sql-cluster-images/bcp-path-custom-json-file-name.png)

To identify the BCP path for the environment, execute the following SQL query against the '**EDDS**' database:

```sql
SELECT 
[TemporaryDirectory]
FROM [EDDS].[eddsdbo].[ResourceServer] AS rs WITH(NOLOCK)
INNER JOIN [EDDS].[eddsdbo].[ExtendedArtifact] AS ea WITH(NOLOCK)
	ON ea.[ArtifactID] = rs.[ArtifactID]
INNER JOIN [EDDS].[eddsdbo].[Code] AS c WITH(NOLOCK)
	ON c.[ArtifactID] = rs.[Type]
INNER JOIN [EDDS].[eddsdbo].[ResourceGroupSQLServers] AS rgss WITH(NOLOCK)
	ON rgss.[SQLServerArtifactID] = rs.[ArtifactID]
INNER JOIN [EDDS].[eddsdbo].[ResourceGroup] AS rg WITH(NOLOCK)
	ON rg.[ArtifactID] = rgss.[ResourceGroupArtifactID]
WHERE c.[Name] = 'SQL - Primary'
```
![](/resources/custom-json-images/sql-bcp-path-query.png)

The Custom JSON file includes the following key sections:
- Monitoring by Instance
- Monitoring by Installed Product
- Monitoring by Host

For more information about the Custom JSON structure, refer to the [Custom JSON Configuration Structure](./environment_watch_configuration.md) document.

## Overview

Monitors the presence and validity of specified Certificates in Windows Certificate stores. There are Certificates by default monitored without configuration and they are based on installed product.

**Default Certificates**
| Certificate Name                  | Description                                      |
|-----------------------------------|--------------------------------------------------|
| Relativity Secret Store           | Certificate for Relativity Secret Store.         |

**Properties In Custom JSON Related to Certificates**

| Property       | Type     | Description                                                      |
|----------------|----------|------------------------------------------------------------------|
| `enabled`      | boolean  | Enables or disables monitoring for Certificates.                 |
| `include`      | array    | List of Certificate objects to monitor.                          |
| `StoreName`    | string   | Name of the Certificate store (e.g., `"My"`).                    |
| `StoreLocation`| string   | Location of the store (e.g., `"LocalMachine"`).                  |
| `Thumbprint`   | string   | Certificate thumbprint to identify the Certificate.              |

#### StoreLocation Enum Values

The `StoreLocation` field specifies the location of the X.509 Certificate store to use.

**Possible Values**

| Value         | Description                                                    |
|---------------|----------------------------------------------------------------|
| CurrentUser   | The X.509 Certificate store is located in the current user's profile. |
| LocalMachine  | The X.509 Certificate store is located in the local computer's profile. |

#### StoreName Enum Values

The `StoreName` field specifies the name of the Windows Certificate store where the X.509 Certificate is located.

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

The command outputs a table with `Thumbprint` and `Subject`. Select the `Thumbprint` for the required Certificate and assign it as shown in the example. Adjust the command based on the `StoreName` and `StoreLocation`, and update the values in the JSON accordingly.


## Configure Certificates

Certificates can be monitored by "**hosts**", "**instance**", or "**installedProducts**". For Certificates to monitor, locate "**certificates**" under the desired section and update the configuration as below.

- `enabled` : Set to `true` to enable Certificate monitoring.
- When configuring the `include` array, each Certificate object must specify the `StoreName`, `StoreLocation`, and `Thumbprint` fields.

**Example 1**: Monitoring one Certificate where `StoreName` is `My`, `StoreLocation` is `LocalMachine`, and `Thumbprint` is obtained from the following PowerShell command `Get-ChildItem Cert:\LocalMachine\My`:

```json
{
	"certificates": {
		"enabled": true,
		"include": [
			{
				"StoreName": "My",
				"StoreLocation": "LocalMachine",
				"Thumbprint": "A54225760344699530649239D175BAA73C70DC1B"
			}
		]
	}
}
```

**Example 2**: Monitoring multiple Certificates (3 in this case) with `StoreName` as `My`, `StoreLocation` as `LocalMachine`, and `Thumbprint` obtained from the following PowerShell command `Get-ChildItem Cert:\LocalMachine\My`:

```json
{
	"certificates": {
		"enabled": true,
		"include": [
			{
				"StoreName": "My",
				"StoreLocation": "LocalMachine",
				"Thumbprint": "F8809D2677E010477847C92C5A1A673784537CBC"
			},
			{
				"StoreName": "My",
				"StoreLocation": "LocalMachine",
				"Thumbprint": "984812C68F059EB19A346D538ECFB072968C11C3"
			},
			{
				"StoreName": "My",
				"StoreLocation": "LocalMachine",
				"Thumbprint": "984812C68F059EB19A346D538ECFB072968C11C3"
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
- Navigate to the Kibana Certificates Dashboard.
- Ensure that the Certificates defined in the Custom JSON configuration appear on the Kibana Certificates Dashboard. The example below demonstrates how a Certificate specified in the Custom JSON is successfully monitored and displayed on the Certificates Dashboard.

![](/resources/custom-json-images/certificate-json-example.png)

![](/resources/custom-json-images/certificate-dashboard-example.png)