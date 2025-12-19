# Custom-JSON Troubleshooting

This document provides guidance for troubleshooting issues related to custom JSON configurations in Relativity Server environments.

## Common Issues

If the log "The Environment Watch shared configuration object is not empty" does not appear in Kibana after configuring the custom JSON file, the potential causes are:

![](/resources/custom-json-troubleshooting-images/environment-watch-shared-settings-not-empty.png)

 **1.1 The custom JSON file is not placed in the correct BCP path.**

Check the custom JSON file path:

Base path: `BCPPath`

Folder: `EnvironmentWatch`

File name: `environment-watch-configuration.json`

Environment Watch automatically reads this file and applies the defined monitoring rules to the relevant instances, products, and hosts.

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

**1.2 The custom JSON file is in the correct BCP path, but the BCP path is not active.**

Check if the BCP path is active:

- Login to the Relativity application.
- Navigate to **Servers**.
- Filter by `SQL - Primary` in the 'Type' column.

![](/resources/custom-json-troubleshooting-images/bcp-path-relativity-ui.png)

- Ensure that the BCP path is active and 'Visible in Dropdown' is set to 'Yes'.

![](/resources/custom-json-troubleshooting-images/bcp-path-active.png)

- Click on 'Edit' and toggle the **'Status'** to 'Inactive' and save it.
- Again, click on 'Edit' and toggle the **'Status'** to 'Active' and save it.
- Restart the Environment Watch Windows Service to apply the changes.

**1.3 The custom JSON file has syntax errors.**

Verify the custom JSON file for syntax errors:

- Please download the sample JSON file from the link below and compare it with your custom JSON file to identify any syntax errors.

[Sample Custom JSON File](/resources/custom-json-troubleshooting-images/environment-watch-configuration.json)

- Restart the Environment Watch Windows Service to apply the changes.

**1.4 The Environment Watch Windows Service is not running the latest version.**

- Verify the version of the Environment Watch Windows Service.
- Ensure it is updated to version **100.0.21** or later, as custom JSON support was introduced in this version.
- If it is not updated, upgrade the Environment Watch Windows Service to the latest version.

> [!NOTE]
> Ensure the `enabled` flag is set to `true` in the custom JSON file for the relevant monitoring section (hosts, instance, or installedProducts).

## Certificates

Possible causes for the following alert in the Relativity application include:

![](/resources/custom-json-troubleshooting-images/certificate-alert.png)

 **1.1 Thumbprint for the certificate is not configured properly.**

Depending on the Store Location and Store Name, run the following command on the host. For `LocalMachine` and `My`, use:

> ```powershell
> Get-ChildItem Cert:\LocalMachine\My

 **1.2 The host name is not proper for the monitoring by Host or Installed Product.** 

- Run below powershell command to get host name.

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
							"include": [
                            {
                                "clusterVirtualName": "SQLCLUSTER",
                                "instanceName": "SQL_INSTANCE"
                            }]
						},
						"windowsServices": {
							"enabled": true,
							"include": [ "MSSQLSERVER" ]
						}
					},
					"otelCollectorYamlFiles": []
				},
        ]
```

 **1.3 Certificate thumbprint is unique and should not be configured in Monitoring by instance.**

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
             }
```

## Windows Services

If Windows services do not appear in the Kibana dashboard after configuring the custom JSON file, the potential causes are:

![](/resources/custom-json-troubleshooting-images/widows-service-dashboard-json.png)

**1.1 The custom JSON file does not include the Windows services configuration.**

- Ensure that the custom JSON file contains the correct configuration for the Windows services to monitor.
- The configuration should be under the `windowsServices` section for the relevant hosts, instance, or installedProducts.

Example:

```json
"hosts": [
				{
					"hostName": "SQL01",
					"sources": {
						"windowsServices": {
							"enabled": true,
							"include": [ "MSSQLSERVER", "AnotherService" ]
						}
					}
				},
        ]
```

**1.2 The Windows services are not running on the host.**

- Verify that the Windows services you are trying to monitor are actually running on the host machine.
- You can check this by logging into the host machine and using the Services management console (`services.msc`) or by using the following PowerShell command:

> ```powershell
> Get-Service -Name MSSQLSERVER, AnotherService

**1.3 Always include service name in the custom JSON**

- Ensure that you are using the correct service names (not display names) in the `include` section of the custom JSON configuration.


## SQL Cluster Instances

**1.1 All instances/nodes in the SQL cluster are not monitored.**

- Verify that each node of the SQL cluster is included in the `hosts` section of the custom JSON configuration with the correct `hostName`, `clusterVirtualName`, and `instanceName`.
- Ensure that `sqlServers` -> `enabled` is set to `true` for each host entry.

```json
"hosts": [
                {
                    "hostName": "SQLNode1",
                    "sources": {
                        "certificates": {
                            "enabled": false,
                            "include": []
                        },
                        "sqlServers": {
                            "enabled": true,
                            "include": [
                            {
                                "clusterVirtualName": "SQLCLUSTER",
                                "instanceName": "SQL_INSTANCE"
                            }
                          ]
                        },
                        "windowsServices": {
                            "enabled": false,
                            "include": [ "MSSQLSERVER" ]
                        }
                    },
                    "otelCollectorYamlFiles": []
                },
                {
                    "hostName": "SQLNode2",
                    "sources": {
                        "certificates": {
                            "enabled": false,
                            "include": []
                        },
                        "sqlServers": {
                            "enabled": true,
                            "include": [
                            {
                                "clusterVirtualName": "SQLCLUSTER",
                                "instanceName": "SQL_INSTANCE"
                            }
                          ]
                        },
                        "windowsServices": {
                            "enabled": false,
                            "include": [ "Dhcp" ]
                        }
                    },
                    "otelCollectorYamlFiles": []
                }
        ]
```

> [!NOTE]
> SQL cluster configuration in the custom JSON file should always be specified within the "**hosts**" section.

**1.2 The log "Processed SQL instance details" is missing, and the "labels.IsProvidedByCustomConfiguration" attribute is not set to true in Kibana->Discover**

- See [Common Issues](#common-issues) section 1.1 to 1.4 to troubleshoot why the Environment Watch Windows service is not picking up the custom JSON changes.

## Slack Notifications

**1.1 Slack messages are being sent too frequently and require to extend the interval.**

- Ensure that the `messageIntervalSeconds` property in the Slack handler configuration is set to a value greater than or equal to 180 seconds to avoid rate limiting by Slack.
- Restart the Environment Watch Windows Service to apply the changes.

**1.2  Kibana Alert is triggered, still cannot see: "Message successfully sent alert {AlertId} to Slack channel {SlackChannel}" in Kibana->Discover**

- {AlertId}: Pass Kibana Alert ID.
- {SlackChannel}: Pass Slack Channel ID.
- Above issue can occur due to issues mentioned in [Common Issues](#common-issues) section 1.1 to 1.4.

**1.3 Slack notifications are not being sent even though the alert is triggered in Kibana.**

- Verify that the Slack handler is correctly configured in the `alertNotificationHandlers` section of the custom JSON file.
- Ensure that the `accessToken`, `channel`, `enabled`, and `messageIntervalSeconds` properties are set correctly.
- See [Common Issues](#common-issues) section 1.1 to 1.4 to troubleshoot why the Environment Watch Windows service is not picking up the custom JSON changes.
- It can also occur if the Slack API token is invalid or does not have the necessary permissions to post messages to the specified channel. Verify the token and permissions.

![](/resources/custom-json-troubleshooting-images/slack-message-in-kibana-discover.png)

![](/resources/custom-json-troubleshooting-images/triggerred-alert-in-kibana.png)

![](/resources/custom-json-troubleshooting-images/slack-notification.png)



