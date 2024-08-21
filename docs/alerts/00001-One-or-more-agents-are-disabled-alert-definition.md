# One or more agents are disabled

## Description: 
The alert is active when at least 1 disabled Agent exists within an active Resource Pool.

## Alert Details:
**Alert ID:** dae7d8e9-c33d-470c-b36a-5b6e380e0a25

**Tags:**
Each tag should follow "key:value" format.

- Type:Platform
- Group:Agents
- PageType:RelativityTab
- PageID:
- CreatedBy:Relativity
- ResolutionText:Go to the Agents tab and identify any agents for which the Enabled value is No. Enable them, if necessary
- ResolutionURL:/docs/alerts/00001-One-or-more-agents-are-disabled-alert-resolution-sop.md

## Metric/Log/Trace Details:
**Metric Name:** relsvr.agent.disabled

**Metric Attributes:**

|Attribute Name|Description|Value|
|-------|---|--|
|labels.agent_name|Relativity Agent Name||
|labels.agent_type_name|Relativity Agent Type|
|labels.application_name|Application Name|Environment Administration & Operations|
|labels.exception_message|Any exception message on Agent||
|labels.message|Message describes the issue|Agent [Agent Name placeholder] is disabled.|
|labels.name|Name of metric|Agent Disabled|
|labels.relsvr_artifact_id|Relativity agent artifact Id||
|labels.relsvr_subsystem|Agent Name||
|labels.relsvr_system|System name|Agents|

## Rule details
**Alert Condition Description:** Alert triggers on disabled agents count greater than 0 for last 30 seconds.

|Name|Value|Description|
|-|-|-|
|Rule Type| Elastic Query||
|Data View| metrics-*||
|Filter Query|relsvr.agent.disabled : 1|Agent should be disabled|
|Group| Count|number of agent disabled|
|Threshold| > 0| Count greter than 0, alert triggers|
|Time Window| 1 min| Verified data for last 1 minute|
|Frequency| 30 sec|Checks for each 30 seconds|

## Requires User Intervention
- Yes: alert immediately
  - Min time before the alert is active/inactive: 90 seconds

## Visualization link
Relativity link to Agents Tab

## Related Alerts
Host Heartbeat alert should not be in active state.

