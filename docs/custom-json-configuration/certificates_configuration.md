# Certificates Configuration

This section describes the configuration of certificates in the `environmentWatchConfiguration` JSON object for monitoring certificates.

---

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