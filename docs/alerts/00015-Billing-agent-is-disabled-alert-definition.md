# Billing agent is disabled

## Description: 
The alert is active when Billing agent is disabled.

## Alert Details:
**Alert ID:** d33236f4-4431-4d53-aa4e-299bc70caead

**Tags:**
Each tag should follow "key:value" format.

- Type:Platform
- Group:Agents
- PageType:RelativityTab
- PageID:
- CreatedBy:Relativity
- ResolutionText:Go to the Agents tab and filter for Billing agent for which the Enabled value is No. Enable them, if necessary
- ResolutionURL:/docs/alerts/00015-Billing-agent-is-disabled-alert-resolution-sop.md
  
## Metric/Log/Trace Details:
**Metric Name:** relsvr.agent.disabled

**Metric Attributes:**

|Attribute Name|Description|Value|
|-------|---|--|
|labels.agent_name|Relativity Agent Name|Billing Agent|
|labels.agent_type_name|Relativity Agent Type|
|labels.application_name|Application Name|Environment Administration & Operations|
|labels.exception_message|Any exception message on Agent||
|labels.message|Message describes the issue|Billing Agent is disabled.|
|labels.name|Name of metric|Agent Disabled|
|labels.relsvr_artifact_id|Relativity agent artifact Id||
|labels.relsvr_subsystem|Agent Name|Billing Agent|
|labels.relsvr_system|System name|Agents|

## Rule details
**Alert Condition Description:** Alert triggers on disabled agent count greater than 0 for last 30 seconds.

|Name|Value|Description|
|-|-|-|
|Rule Type| Elastic Query||
|Data View| metrics-*||
|Filter Query|relsvr.agent.disabled : 1 and labels.agent_type_name:"Billing Agent" |Agent should be disabled|
|Group| Count|number of agent disabled|
|Threshold| > 0| Count greater than 0, alert triggers|
|Time Window| 1 min| Verified data for last 1 minute|
|Frequency| 30 sec|Checks for each 30 seconds|

## Requires User Intervention
- Yes: alert immediately
  - Min time before the alert is active/inactive: 90 seconds

## Visualization link
Relativity link to Agents Tab

## Related Alerts
Host Heartbeat alert should not be in active state.

