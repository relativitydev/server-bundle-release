## Alert Notification Handlers [Preview Feature]

The `alertNotificationHandlers` section configures integrations for sending alerts when monitored resources meet specified conditions. This enables automated notifications to external systems such as Slack.

## Custom JSON Configuration Structure

To monitor Kibana Alerts through Slack notification, configure the Slack details in a custom JSON file. This file should be stored in the `BCPPath` directory within the `EnvironmentWatch` folder and named `environment-watch-configuration.json`. An example of the BCPPath and folder structure is shown below:

![](/resources/sql-cluster-images/bcp-path-custom-json-file-name.png)

The custom JSON file includes the following key sections:
- Monitoring by Instance
- Monitoring by Installed Product
- Monitoring by Host

For details about the custom JSON structure, refer to the [Custom JSON Configuration Structure](./environment_watch_configuration.md) document.

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

Step 1 : Click on 3 horizontal lines on the extreme left side of the slack. Click `File` and then `New Channel` 

![New Channel](../../resources/slackalerts-images/NewChannel.png)

Step 2 : Enter Channel Name 

![New Channel](../../resources/slackalerts-images/createachannel.png)

Step 3 : Make the channel visibility `Public`. click on `Create`.

![Channel Visibility](../../resources/slackalerts-images/CreateChannelVisibility.png)

### Map Channel to Slack App

go to the required channel aand type below command and Enter

/invite @appname 

where appname is slack app name.

### Get Channel Id

Go to required channel and in About copy channel Id and assign it to channel in slack configuration.

![New Channel](../../resources/slackalerts-images/ChannelId.png)

### Slack Notofication Example

![Slack Example](../../resources/slackalerts-images/SlackNotification.png)

