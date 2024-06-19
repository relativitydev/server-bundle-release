# Windows Service is stopped for at least one Resource Server

## Description: 
Alert is true if the Windows Service is stopped for at least one agent server, web background processing server, worker manager server or analytics server.

## Alert Details:
**Alert ID:** d14f3e28-fa2b-4df8-8b02-4d35d45bb800

**Tags:**
Each tag should follow "key:value" format.

- Type:Infrastructure
- Group:System Health
- PageType:Dashboard
- PageID:cd200ee0-1e61-4645-8220-83ce82914a71
- CreatedBy:Relativity
- ResolutionText:For any Resource Server where the Windows Service is stopped, go to that Server page and click 'Restart Service'
- ResolutionURL:/docs/alerts/00006-Windows-service-is-stopped-for-at-least-one-resource-server-alert-resolution-sop.md

## Metric Details:
**Metric Name:** relsvr.windows_service.running

**Metric Attributes:**

|Attribute Name|Description|Value|
|-------|---|--|
|labels.name|Name of Service||
|labels.startup_mode|||
|labels.state||accessible/inaccessible|

## Rule details
**Alert Condition Description:** Alert triggers on at least one windows service stopped for at least 90 seconds.

|Name|Value|Description|
|-|-|-|
|Rule Type| Elastic Query||
|Data View| metrics-*||
|Filter Query|relsvr.windows_service.running : 0|Windows service is stopped|
|Group| Count|number of Windows service is stopped|
|Threshold| > 0| Count greter than 0, alert triggers|
|Time Window| 90 sec| Verified data for last 90 sec|
|Frequency| 30 sec |Checks for each 30 seconds|

## Requires User Intervention
- Yes: alert immediately
  - Min time before the alert is active/inactive: 90 seconds

## Visualization link
Kibana dashboard link

## Related Alerts
- Host Heartbeat alert should not be in active state.
- If windows service is associated with any resource server, then "One or more Resource Servers are inactive" alert should fire.
