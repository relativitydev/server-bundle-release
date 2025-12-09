# Custom-JSON Troubleshooting

This document provides troubleshooting guidance for custom json  issues encountered during configuration and operation in Relativity Server environments.

> [!NOTE]
> The default custom JSON path is `BCP Path\EnvironmentWatch`, where `BCPPath` is a shared path set in the Relativity application. The configuration file is named `environment-watch-configuration.json`.

## Certificate issues

### 1.  x509: certificate has expired or is not yet valid Alert is active

If you see this alert in the Relativity application, possible causes include:

 *1.1 Thumbprint for the certificate is not configured properly.*

**Check certificate thumbprint is present or not:**

Depending on the Store Location and Store Name, run the following command on the host. For `LocalMachine` and `My`, use:

> ```powershell
> Get-ChildItem Cert:\LocalMachine\My

 *1.2 The host name is not proper for the monitoring by Host or Installed Product.* 

 **Check hostname properly:**

 run this command to get host name.

 > ```powershell
> hostname


Ensure the `hostName` property in your configuration matches the output. Example:

```json
"hosts": [
				{
					"hostName": "SQL01",
					"sources": {
						"certificates": {
							"enabled": true,
							"include": []
						},
						"sqlServers": {
							"enabled": true,
							"include": [ "VIRTSQL\\INSTANCE01" ]
						},
						"windowsServices": {
							"enabled": true,
							"include": [ "MSSQLSERVER" ]
						}
					},
					"otelCollectorYamlFiles": []
				},
```

 *1.3 Certificate thumprint is unique and should not be configured in Monitoring by instance.*

 Example configuration for instance monitoring:


```json
"instance": {
				"sources": {
					"certificates": {
						"enabled": false,
						"include": []
					},
					"windowsServices": {
						"enabled": true,
						"include": [
							"WindowsAzureGuestAgent",
							"mpssvc"
						]
					}
				}
```