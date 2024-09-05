# Processing License will expire soon

## Description: 
The alert is true when Processing license expiration date is 1-30 days greater than current date.

## Alert Details:
**Alert ID:** d4274f6a-7ae3-4664-b0ba-25578af76afd
              

**Tags:**
Each tag should follow "key:value" format.

•	Type:Platform

•	Group:Billing & Licensing

•	PageType:RelativityTab

•	PageID:

•	CreatedBy:Relativity

•	ResolutionText:

•	ResolutionURL:/docs/alerts/00016-Processing-license-will-expire-soon-alert-resolution-sop.md


## Metric/Log/Trace Details:
**Metric Name:** relsvr.license.expires_days

**Metric Attributes:**

| Attribute Name          | Description                                      | Value                          |
|-------------------------|--------------------------------------------------|--------------------------------|
| labels.application_name | Application Name                                 | System/Processing              |
| labels.application_guid | Application GUID                                 |                                |
| labels.name             |                                                  | License Expires within N days  |
| labels.relsvr_system    |                                                  | License Manager Application    |
| labels.relsvr_subsystem |                                                  | License management application |
| labels.message          | Message describes the license expiration details |

## Rule details
**Alert Condition Description:** Alert triggers on when Processing license expiration date is 1-30 days greater than current date. 

| Name         | Value                                                           | Description                                       |
|--------------|-----------------------------------------------------------------|---------------------------------------------------|
| Rule Type    | Metric threshold                                                |                                                   |
| Filter Query | labels.application_guid: "ED0E23F9-DA60-4298-AF9A-AE6A9B6A9319" | Relativity Application                            |
| Group        | min                                                             | Min of days to expire license                     |
| Threshold    | Between 1-30                                                    | Min of expire days between 1 - 30, alert triggers |
| Time Window  | 30 min                                                          | Verified data for last 30 minutes                 |
| Frequency    | 1 min                                                           | Checks for each 1 minute                          |

## Requires User Intervention
- Yes: alert immediately
  - Is the metric based on expiration? Yes
  - Min time before the alert is active/inactive: 30 minutes

## Visualization link
Relativity link to License Tab

## Related Alerts
Host Heartbeat alert should not be in active state.

