# Relativity license has expired. You have less than 7 days before access to your Relativity environment is disabled

## Description: 
The alert is true when Relativity license expiration date is equal to or greater than the current date. You have less than 7 days before access to your Relativity environment is disabled.

## Alert Details:
**Alert ID:** f564a7d2-fd39-4d9d-89ff-20ed32ee379a

**Tags:**
Each tag should follow "key:value" format.

- Type:Platform
- Group:Billing & Licensing
- PageType:RelativityTab
- PageID:
- CreatedBy:Relativity
- ResolutionText:
- ResolutionURL:/docs/alerts/00012-Relativity-license-has-expired-resolution-sop.md

## Metric Details:
**Metric Name:** relsvr.license.expires_days

**Metric Attributes:**

|Attribute Name|Description|Value|
|-------|---|--|
|labels.application_name|Application Name|System/Processing|
|labels.application_guid|Application GUID||
|labels.name||License Expires within N days|
|labels.relsvr_system||License Manager Application|
|labels.relsvr_subsystem||License management application|
|labels.message|Message describes the license expiration details||

## Rule details
**Alert Condition Description:** Alert triggers on when  Relativity license expiration date is equal to or greater than the current date.

|Name|Value|Description|
|-|-|-|
|Rule Type| Metric threshold||
|Filter Query|labels.application_guid: "BD10A60D-B8EC-4928-84EE-6FC4F30D9612"|Relativity Application|
|Group| min| Min of days to expire license|
|Threshold| < 1| Min of expire days less than 1, alert triggers|
|Time Window| 30 min| Verified data for last 30 minutes|
|Frequency| 1 min|Checks for each 1 minute|

## Requires User Intervention
- Yes: alert immediately
  - Is the metric based on expiration? Yes
    - Min time before the alert is active/inactive: 30 minutes

## Visualization link
Relativity link to License Tab

## Related Alerts
Host Heartbeat alert should not be in active state.

