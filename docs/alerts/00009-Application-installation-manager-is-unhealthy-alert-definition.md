# Application Installation Manager is unhealthy

## Description: 
Alert will be true when Application Installation Manager is unhealthy.

## Alert Details:
**Alert ID:** d212d142-8150-4f59-87c6-fc8f06fe41c8

**Tags:**
Each tag should follow "key:value" format.

- Type:Application
- Group:ADS
- PageType:
- PageID:
- CreatedBy:Relativity
- ResolutionText:
- ResolutionURL:/docs/alerts/00009-Application-installation-manager-is-unhealthy-alert-resolution-sop.md

## Log Details:

**Log Attributes:**

|Attribute Name|Description|Value|
|-------|---|--|
|labels.event_source|Event Source Name|relsvr.ads.application.install|
|labels.event_type|Event Type|health_check|
|labels.name|Health Check Name|relsvr.ads.application.install.failed|
|labels.status_state|unhealthy/healthy|unhealthy|

## Rule details
**Alert Condition Description:** Alert triggers on Application installation failure.

|Name|Value|Description|
|-|-|-|
|Rule Type| Log threshold||
|Data View| logs-*||
|Filter Query|labels.event_type : "health_check" AND labels.event_source : "relsvr.ads.application.install" AND labels.name : "relsvr.ads.application.install.failed"|Application installation failure|
|Group| Count| Count of "relsvr.ads.application.install.failed" health check records|
|Threshold| > 0| Count > 0, alert triggers|
|Time Window| 1 hour| Verified data for last 1 hour|
|Frequency| 1 min|Checks for each 1 minute|

## Requires User Intervention
- Yes: alert immediately
  - Min time before the alert is active: 90 seconds
  - Min time before the alert is inactive: 1 hour

## Visualization link
No Link

## Related Alerts
Host Heartbeat alert should not be in active state.

