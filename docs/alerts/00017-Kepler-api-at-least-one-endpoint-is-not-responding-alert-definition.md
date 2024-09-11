# Kepler API - at least one endpoint is not responding

## Description: 
The alert is active when HTTP request to application/service returns status code other than 200.

## Alert Details:
**Alert ID:** e6759278-b70f-4d11-a4ab-13b6f231a1a5
              

**Tags:**
Each tag should follow "key:value" format.

- Type:Application
- Group:Kepler Services
- PageType:Dashboard
- PageID:
- CreatedBy:Relativity
- ResolutionText:
- ResolutionURL:/docs/alerts/00017-Kepler-api-at-least-one-endpoint-is-not-responding-alert-resolution-sop.md

## Metric/Log/Trace Details:
**Metric Name:** NOT numeric_labels.http_status_code : 200 and httpcheck.status: *

**Metric Attributes:**

| Attribute Name          | Description                                      | Value                          |
|-------------------------|--------------------------------------------------|--------------------------------|
| labels.http_url         | HTTP request endpoint                            | Example:http://emttest/relativity.rest/api/massoperation/v1/mass%20operation%20manager/getkeplerstatusasync              |
| httpcheck.status        | HTTP request status                              | 0/1                                |
| numeric_labels.http_status_code             |   Status code of HTTP request                                               | 200/404/500  |
| labels.http_status_class    | HTTP request stages                                             | 1XX/2XX/3XX/4XX/5XX    |


## Rule details
**Alert Condition Description:** Alert triggers on when HTTP request to application/service returns status code other than 200. 

|Name|Value|Description|
|-|-|-|
|Rule Type| Elastic Query||
|Data View| metrics-*||
|Filter Query|NOT numeric_labels.http_status_code : 200 and httpcheck.status: *|HTTP request of application/service is inaccessible|
|Threshold| > 0| Count greater than 0, alert triggers|
|Time Window| 5 min| Verified data for last 5 min|
|Frequency| 1 min |Checks for every 1 min|

## Requires User Intervention
- Yes: alert immediately
  - Min time before the alert is active/inactive: 5 minutes

## Visualization link
Kibana dashboard link

## Related Alerts
One or more Resource Servers are inactive.

