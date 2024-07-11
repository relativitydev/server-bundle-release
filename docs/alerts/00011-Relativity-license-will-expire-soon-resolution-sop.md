# Alert: Relativity license will expire soon

|||
|-|-|
|Content owner||
|Team|server-architects|
|Status|Pending|
|Last reviewed date||
|Rollout date||
## When to Use
- For JIRA Service Desk incidents, "License Expires within 30 days"
- Use this procedure when responding to a "Relativity license will expire soon" alert notification.
- For general Relativity license expirations

## Audience
Site services.

## Prerequisites
Access to License Manager - https://licensemanager.kcura.corp/

Front end access to Relativity instance

## Procedures
- Updating an Existing License
  - Take a Backup of License Table
  - Delete the Existing License that is to be replaced (from SQL primary database)
  - Generate and Apply New Licenses
  - Backout Procedure (if necessary)