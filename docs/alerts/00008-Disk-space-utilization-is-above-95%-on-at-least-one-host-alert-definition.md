# Disk space utilization is above 95% on at least one host

## Description: 
The alert is active when the disk space utilization exceeds 95% on at least one host for at least 5 minutes.

## Alert Details:
**Alert ID:** 3adc059d-d2a0-4706-8ceb-eeeac29f6337

**Tags:**
Each tag should follow "key:value" format.

- Type:Infrastructure
- Group:System Health
- PageType:Dashboard
- PageID:ee8a7dab-7422-499e-94e0-7b5449468a94
- CreatedBy:Relativity
- ResolutionText:
- ResolutionURL:/docs/alerts/00008-Disk-space-utilization-is-above-95%-on-at-least-one-host-alert-resolution-sop.md

## Metric Details:
**Metric Name:** system.filesystem.utilization

**Metric Attributes:**

|Attribute Name|Description|Value|
|-------|---|--|
|labels.device|The filesystem device name. For Windows based OS's, this is normally the primary OS drive letter. ||
|labels.mode|Mountpoint mode such "ro", "rw", etc.||
|labels.mountpoint|Mountpoint path.||
|labels.type|Filesystem type, such as, "NTFS", "CDFS", etc.||

## Rule details
**Alert Condition Description:** Alert triggers on Disk space utilization exceeds 95% on at least one host for at least 5 minutes.

|Name|Value|Description|
|-|-|-|
|Rule Type| Metric threshold||
|Group| Average| Average of Disk utilization per host|
|Threshold| > 0.95| Average > 0.95, alert triggers|
|Time Window| 5 min| Verified data for last 5 minutes|
|Frequency| 1 min|Checks for each 1 minute|
|Group alerts by| host.name| Group by host |

## Requires User Intervention
- No
  - Min time before the alert is active: 5 minutes
  - Min time before the alert is inactive: 1 - 3 minutes

## Visualization link
Kibana dashboard link

## Related Alerts
Host Heartbeat alert should not be in active state.

