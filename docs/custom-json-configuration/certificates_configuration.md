# Certificates Configuration

This section describes the configuration of certificates in the `environmentWatchConfiguration` JSON object for monitoring certificates.

---

## Custom JSON Configuration Structure

To monitor Certificates, configure the Slack details in a custom JSON file. This file should be stored in the `BCPPath` directory within the `EnvironmentWatch` folder and named `environment-watch-configuration.json`. An example of the BCPPath and folder structure is shown below:

![](/resources/sql-cluster-images/bcp-path-custom-json-file-name.png)

The custom JSON file includes the following key sections:
- Monitoring by Instance
- Monitoring by Installed Product
- Monitoring by Host

For details about the custom JSON structure, refer to the [Custom JSON Configuration Structure](./environment_watch_configuration.md) document.

## Overview

Monitors the presence and validity of specified certificates in Windows certificate stores.There are certificates by default monitored without configuration and they are based on installed product.

**Default Certificates**
| Certificate Name                  | Description                                      |
|-----------------------------------|--------------------------------------------------|
|Relativity Secret Store            | Certificate for Relativity Secret Store.         |

**Properties Table**

| Property      | Type     | Description                                                      |
|---------------|----------|------------------------------------------------------------------|
| `enabled`     | boolean  | Enables or disables monitoring for certificates.                 |
| `include`     | array    | List of certificate objects to monitor.                          |
| `StoreName`   | string   | Name of the certificate store (e.g., `"My"`).                    |
| `StoreLocation`| string  | Location of the store (e.g., `"LocalMachine"`).                  |
| `Thumbprint`  | string   | Certificate thumbprint to identify the certificate.              |


#### StoreLocation Enum Values

The `StoreLocation` Field specifies the location of the X.509 certificate store to use.

**Possible Values**

| Value         | Description                                                    |
|---------------|----------------------------------------------------------------|
| CurrentUser   | The X.509 certificate store is located in the current user's profile. |
| LocalMachine  | The X.509 certificate store is located in the local computer's profile. |

#### StoreName Enum Values

The `StoreName` Field specifies name of the Windows certificate store where the X.509 certificate is located.

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

**Get certificate thumbprint**

Depending on the Store Location and Store Name, run the following command on the host. For `LocalMachine` and `My`, use:

> ```powershell
> Get-ChildItem Cert:\LocalMachine\My

You will get table with Thumprint and Subject. Select the Thumprint for the certificate required and assign it as shown in the example. Based on the Store Name and Store Location command changes and the values in the json too.


**Example**
```json
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
```