# Alert: Kepler API - at least one endpoint is not responding

|||
|-|-|
|Content owner||
|Team|server-architects|
|Status|Pending|
|Last reviewed date||
|Rollout date||
## When to Use
Use this procedure when responding to a "Kepler API - at least one endpoint is not responding" alert notification.

This alert indicates that atleast one http request of the application/service is not responding.

## Audience
Site services, Server support.

## Prerequisites
Backend access to the Relativity instance
SQL access for the Relativity instance

## Procedures
- Go to the Backend of the Relativity instance.
- Locate 'kCura Service Host Manager' windows service is up and running
- If any one application/service not working re-run the 'kCura Service Host Manager' window service.
- Wait for 5 min so that all applications/services are deleted and re-created.
- Refresh Kibana browser and the alert will be recovered.