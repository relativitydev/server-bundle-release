# One or more Resource Servers are inactive

## Description: 
The alert is true if the 'Status' field = "Inactive" for any Servers.

## Alert Details:
**Alert ID:** 1b3b3a99-5c2e-4b5d-8ee6-a6d8b77de3d9

**Tags:**
Each tag should follow "key:value" format.

- Type:Platform
- Group:Server Health
- PageType:RelativityTab
- PageID:
- CreatedBy:Relativity
- ResolutionText:Go to the Servers tab, locate any server that has a status of Inactive, open it, and edit the Status field to be Active.
- ResolutionURL:/docs/alerts/00002-One-or-more-resource-servers-are-inactive-alert-resolution-sop.md

## Metric/Log/Trace Details:
**Metric Name:** relsvr.server.inactive

**Metric Attributes:**

|Attribute Name|Description|Value|
|-------|---|--|
|labels.application_name|Application Name|Environment Administration & Operations|
|labels.message|Message describes the issue|The [Resource server type] Resource Server (ArtifactID: [Resource Server Artifact Id]) [Resource Server Name] is inactive.|
|labels.name|Name of metric|Server Inactive|
|labels.relsvr_artifact_id|Resource Server Artifact Id||
|labels.relsvr_subsystem|Resource Server Name||
|labels.relsvr_system|System name|Resource Servers|

## Rule details
**Alert Condition Description:** Alert triggers on inactive resource server count greater than 0 for last 30 seconds.

|Name|Value|Description|
|-|-|-|
|Rule Type| Elastic Query||
|Data View| metrics-*||
|Filter Query|relsvr.server.inactive : 1|Resource Server is inactive|
|Group| Count|number of inactive resource servers|
|Threshold| > 0| Count greter than 0, alert triggers|
|Time Window| 1 m| Verified data for last 1 minute|
|Frequency| 30 sec|Checks for each 30 seconds|

## Requires User Intervention
- Yes: alert immediately
  - Min time before the alert is active/inactive: 90 seconds

## Visualization link
Relativity link to Servers Tab

## Related Alerts
- Host Heartbeat alert should not be in active state.
- If resource server is associated with below window services, then "Windows Service is stopped for at least one Resource Server" alert should fire
  - kCura EDDS Agent Manager
  - kCura EDDS Web Processing Manager
  - kCura Service Host Manager
  - QueueManager
  - Relativity Secret Store
  - RabbitMQ
  - W3SVC