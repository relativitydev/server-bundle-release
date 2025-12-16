## Alert Notification Handlers [Preview Feature]

The `alertNotificationHandlers` section configures integrations for sending alerts when monitored resources meet specified conditions. This enables automated notifications to external systems such as Slack.

## Custom JSON Configuration Structure

To monitor Kibana Alerts through Slack Notification, configure the Slack details in a Custom JSON file. This file should be stored in the `BCPPath` directory within the `EnvironmentWatch` folder and named `environment-watch-configuration.json`. An example of the BCPPath and folder structure is shown below:

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

### Slack Handler

The Slack handler allows alerts to be sent to a designated Slack channel. Configuration options include:

| Property                   | Description                                                                 |
|----------------------------|-----------------------------------------------------------------------------|
| `accessToken`              | The Slack API token used for authentication.                                |
| `acknowledgeAlertEnabled`  | Boolean flag to enable/disable alert acknowledgment in Slack. This is by dafault false since implementation is not done.               |
| `channel`                  | The Slack channel ID where alerts will be posted.                         |
| `enabled`                  | Boolean flag to enable/disable Slack notifications.                         |
| `messageIntervalSeconds`   | Interval (in seconds) between alert messages sent to Slack. It should be more than or equal to  min slack interval in seconds i.e. 180               |


---

## Configure Slack Channel

### Create a Slack App

Step 1 : Go to Slack API website:
Open your browser and go to: https://api.slack.com/apps


Step 2 : Click "Create New App"

![Create Slack App](../../resources/slackalerts-images/CreateSlackApp.png)

Step 3 : Choose "From scratch"

Step 4 : Name the AppName and Pick your workspace from the dropdown.Click `Create App`

![Pick Workspace](../../resources/slackalerts-images/PickAworkspace.png)

### Get Access Token

Go To Settings of Channel. Select `OAuth & Permissions`. Copy Bot User Auth Token and assign it to access token.

![OAuth Token](../../resources/slackalerts-images/OAuthToken.png)


### Create Slack Channel

Step 1 : Click on 3 horizontal lines on the extreme left side of the Slack. Click `File` and then `New Channel` 

![New Channel](../../resources/slackalerts-images/NewChannel.png)

Step 2 : Enter Channel Name 

![New Channel](../../resources/slackalerts-images/createachannel.png)

Step 3 : Make the channel visibility `Public`. click on `Create`.

![Channel Visibility](../../resources/slackalerts-images/CreateChannelVisibility.png)

### Map Channel to Slack App

Navigate to the required channel and type the below command and press Enter

/invite @appname 

where appname is Slack app name.

### Get Channel Id

Navigate to required channel and in About copy channel Id and assign it to channel in Slack configuration.

![New Channel](../../resources/slackalerts-images/ChannelId.png)

### Configure Slack in Custom JSON

To configure Slack Notification in the Custom JSON file, locate the `alertNotificationHandlers` section and update the configuration as below.

- Provide OAuth Token in `accessToken`.
- Set `channel` to the Slack channel ID where alerts will be sent.
- Set `enabled` to `true` to enable Slack notifications.
- Set `messageIntervalSeconds` to define the interval at which messages are sent to Slack. By default, it is set to 180 seconds from the code base. If, we set it to less than 180 seconds, it will be overridden to 180 seconds.

```json
"alertNotificationHandlers": {
			"slack": {
				"accessToken": "slack-access-token",
				"acknowledgeAlertEnabled": false,
				"channel": "slack-channel-id",
				"enabled": true,
				"messageIntervalSeconds": 60
			}
		}
```

### Verification in Kibana

- Navigate to Kibana Discover.
- Select `logs-*` Data View.
- Search for "The Environment Watch shared configuration object is not empty" which indicates that the EW Windows Service fetching values from the Custom JSON configuration successfully.
![](/resources/custom-json-images/environment-watch-shared-settings-not-empty-generic.png)

### Slack Notification Example

![Slack Example](../../resources/slackalerts-images/SlackNotification.png)

