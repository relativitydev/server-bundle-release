# Custom pages failed to install for at least one application

## Description: 
Alert will be true when Custom pages failed to install for at least one application.

## Alert Details:
**Alert ID:** e0122758-1312-412f-8ae4-23c134ecb585

**Tags:**
Each tag should follow "key:value" format.

- Type:Platform
- Group:ADS
- PageType:SavedSearch
- PageID:1bd86ef8-3b9d-4aa8-819d-2cc6a7ca8f3b
- CreatedBy:Relativity
- ResolutionText:
- ResolutionURL:/docs/alerts/00010-Custom-pages-failed-to-install-for-at-least-one-application-sop.md

## Metric Details:
**Metric Name:** relsvr.custom.page.install.failed

**Metric Attributes:**

|Attribute Name|Description|Value|
|-------|---|--|
|labels.application_name|Name of the application||
|labels.name|Failed Custom Page Deployment||
|labels.relsvr_system|Agents||
|labels.relsvr_subsystem|Custom Page Deployment Manager||
|labels.relsvr_server_name|EMTTEST||
|labels.installation_date_time|Installation date of the application||

## Rule details
**Alert Condition Description:** Alert triggers on when custom page of any application fails to install for at least 1 minute.

|Name|Value|Description|
|-|-|-|
|Rule Type| Elastic Query||
|Data View| metrics-*||
|Filter Query|relsvr.custom.page.install.failed : 1||
|Group| Count|number of custom page fail to install|
|Threshold| > 0| Count greter than 0, alert triggers|
|Time Window| 1 min| Verified data for last 1 minute|
|Frequency| 30 sec|Checks for every 30 seconds|

## Requires User Intervention
- Yes
  - Min time before the alert is active: 5 minutes
  - Min time before the alert is inactive: 1 - 3 minutes

## Visualization link
Kibana saved search link

## Related Alerts
Windows Service stopped alert

