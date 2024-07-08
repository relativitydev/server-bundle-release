# Alert: Custom pages failed to install for at least one application

|||
|-|-|
|Content owner||
|Team|server-architects|
|Status|Pending|
|Last reviewed date||
|Rollout date||
## When to Use
The following procedure should be used during the Custom Page deployment stage of a Relativity Upgrade pipeline when deploying new version of a Relativity application.

This alert indicates that one or more custom pages fail to install.

## Audience
Site Services, Internal Developers

## Prerequisites
Below are the pre-requisites
- Access to Relativity EDDS database
- Backend access to the Relativity Web servers

## Procedures
- Make sure that agents are enabled. Custom Page deployment is performed by Custom Page Deployment Manager agent. 
- Make sure that Web Processing service is running. Custom Page Deployment Manager agents are a special case of background workers. They are not running on the agent servers like most of the Relativity agents. They're running directly on the Web servers under the kCura EDDS Web Processing Manager service.
  Make sure that the kCura EDDS Web Processing Manager service is up and running on all web servers.
- If, it is not running, restart the kCura EDDS Web Processing Manager service.