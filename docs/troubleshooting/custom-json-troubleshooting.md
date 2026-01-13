# Custom JSON Configuration Troubleshooting

This document provides guidance for troubleshooting issues related to custom JSON configuration file in Relativity Server environments.

## Common Issues

If the log message “The Environment Watch shared configuration object is not empty” does not appear in Kibana, review the following potential causes:

![](/resources/custom-json-troubleshooting-images/environment-watch-shared-settings-not-empty.png)

 **1.1 The custom JSON configuration file is not placed in the correct BCP path.**

Verify that the custom JSON configuration file is located in the correct BCP path.

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

**1.2 Activate the BCP path if it is not active.**

- Log in to the Relativity application.
- Navigate to **Servers**.
- Filter by `SQL - Primary` in the 'Type' column.

![](/resources/custom-json-troubleshooting-images/bcp-path-relativity-ui.png)

- Ensure the BCP path is active and 'Visible in Dropdown' is set to 'Yes'.

![](/resources/custom-json-troubleshooting-images/bcp-path-active.png)

- Click on 'Edit' and toggle the **'Status'** to 'Inactive' and save it.
- Again, click on 'Edit' and toggle the **'Status'** to 'Active' and save it.
- Restart the Environment Watch Windows Service to apply the changes.

**1.3 Check the custom JSON configuration file for syntax errors.**

- Download the sample JSON file from the link below and compare it with your custom JSON configuration file to identify any syntax errors.

[Sample Custom JSON Configuration File](/resources/custom-json-troubleshooting-images/environment-watch-configuration.json)

- Restart the Environment Watch Windows Service to apply the changes.

**1.4 Update the Environment Watch Windows Service to the latest version.**

- Verify the installed version of the Environment Watch Windows Service.
- Ensure the service is updated to version 100.0.21 or later, as custom JSON configuration file support was introduced in this version.
- If the service is running an earlier version, upgrade it to the latest available version.

> [!NOTE]
> Ensure the `enabled` flag is set to `true` in the custom JSON configuration file for the relevant monitoring section (hosts, instance, or installedProducts).
## Certificates

If certificates do not appear in the Kibana dashboard after configuring the custom JSON configuration file, review the following potential causes:

- Refer to [Common Issues](#common-issues) section 1.1 to 1.4 to troubleshoot why the Environment Watch Windows service is not picking up the custom JSON configuration file changes.

After fixing the common issues, if certificate alert is triggerred in Kibana, review the following potential causes:

![](/resources/custom-json-troubleshooting-images/certificate-alert.png)

 **2.1 Configure the certificate thumbprint properly.**

Run the following PowerShell command based on the certificate store location and name. For `LocalMachine` and `My`, use:

> ```powershell
> Get-ChildItem Cert:\LocalMachine\My

 **2.2 Certificate thumbprint configured correctly, but the certificate is expired.**

 - Verify the certificate is expirated or not from certificate dashboard.
 - If expired, replace it with a valid certificate by running the following PowerShell command: For `LocalMachine` and `My`, use:
 
 > ```powershell
> Get-ChildItem Cert:\LocalMachine\My

 **2.3 Ensure the host name is correct for monitoring by Host or Installed Product.** 

- Run the following PowerShell command.

 > ```powershell
> hostname

- Ensure the `hostName` property in your configuration matches the output. Example:

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

 **2.4 Avoid configuring certificate thumbprints in Monitoring by instance.**

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

If Windows services do not appear in the Kibana dashboard after configuring the custom JSON configuration file, review the following potential causes:

- Refer to [Common Issues](#common-issues) section 1.1 to 1.4 to troubleshoot why the Environment Watch Windows service is not picking up the custom JSON configuration file changes.

After fixing the common issues, if Windows services do not appear in the Kibana dashboard, review the following potential causes:

![](/resources/custom-json-troubleshooting-images/windows-service-dashboard-json.png)

**3.1 Include the Windows services configuration in the custom JSON configuration file.**

- Ensure the custom JSON configuration file contains the correct configuration for the Windows services to monitor.
- Place the configuration under the `windowsServices` section for the relevant hosts, instance, or installedProducts.

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

**3.2 Verify that the Windows services are running and exist on the host.**

- Verify that the Windows services to be monitored are running and exist on the host machine.
- Use the Services management console (`services.msc`) or the following PowerShell command:

> ```powershell
> Get-Service -Name MSSQLSERVER, AnotherService

**3.3 Always include service name in the custom JSON configuration file**

- Ensure that the correct service names (not display names) are used in the include section of the custom JSON configuration file.

## SQL Cluster Instances

When monitoring SQL cluster instances using the custom JSON configuration file, ensure the following:

- All nodes in the cluster are properly configured.
- The Environment Watch Windows service picks up the configuration changes.
- The log message below appears in Kibana to confirm that SQL cluster instance details are picked up from the custom JSON configuration file.

- Log Message: "Processed SQL instance details"
- Attribute in Kibana -> Discover: "labels.IsProvidedByCustomConfiguration" set to true under logs-\* data view.

![](/resources/sql-cluster-images/processed-sql-details-true.png)

**4.1 If the log message "Processed SQL instance details" does not appear in Kibana, and the "labels.IsProvidedByCustomConfiguration" attribute is not set to true in Kibana->Discover under logs-`*` data view:**

- Refer to [Common Issues](#common-issues) section 1.1 to 1.4 to troubleshoot why the Environment Watch Windows service is not picking up the custom JSON configuration file changes.

**4.2 Ensure all instances/nodes in the SQL cluster are monitored.**

- Include each node of the SQL cluster in the `hosts` section of the custom JSON configuration file with the correct `hostName`, `clusterVirtualName`, and `instanceName`.
- Set `sqlServers` -> `enabled` to `true` for each host entry.

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
> Always specify SQL cluster configuration within the "**hosts**" section of the custom JSON configuration file.

## Slack Notifications

When troubleshooting Slack notification issues, first step is to verify whether the following log message appears in Kibana Discover for the triggered alert.

- Log Message: "Message successfully sent alert {AlertId} to Slack channel {SlackChannel}"

**5.1  When Kibana Alert is triggered, and unable to see: "Message successfully sent alert {AlertId} to Slack channel {SlackChannel}" log in Kibana Discover**

- Pass the Kibana Alert ID (`{AlertId}`) and Slack Channel ID (`{SlackChannel}`) in Kibana Discover.
- If message is not found, can mean custom JSON configuration file is not configured correctly. 
- Refer to [Common Issues](#common-issues) section 1.1 to 1.4 to troubleshoot why the Environment Watch Windows service is not picking up the custom JSON configuration file changes.

**5.2 Above message appears in Kibana Discover after fixing common issues, but Slack notifications are still not being sent**

- Verify that the Slack handler is correctly configured in the `alertNotificationHandlers` section of the custom JSON configuration file.
- Ensure that the `accessToken`, `channel`, `enabled`, and `messageIntervalSeconds` properties are set correctly.
- See [Common Issues](#common-issues) section 1.1 to 1.4 to troubleshoot why the Environment Watch Windows service is not picking up the custom JSON configuration file changes.
- It can also occur if the Slack API token is invalid or does not have the necessary permissions to post messages to the specified channel. 
- Verify the token and permissions.
- Once the above issues are resolved, Slack notifications should be successfully sent. 

![](/resources/custom-json-troubleshooting-images/slack-message-in-kibana-discover.png)

![](/resources/custom-json-troubleshooting-images/triggerred-alert-in-kibana.png)

![](/resources/custom-json-troubleshooting-images/slack-notification.png)

**If users need to adjust the frequency of Slack messages, follow these steps:**

- Set the `messageIntervalSeconds` property in the Slack handler configuration to a value greater than or equal to 180 seconds to avoid rate limiting by Slack.
- Restart the Environment Watch Windows Service to apply the changes.



