# Alert: Processing License will expire soon

|||
|-|-|
| Content owner      |                   |
| Team               | server-architects |
| Status             | Pending           |
| Last reviewed date |                   |
| Rollout date       |

## When to Use
•	For JIRA Service Desk incidents, "License Expires within 30 days"

•	Use this procedure when responding to a "Processing license will expire soon" alert notification.

•	For general Processing license expirations

## Audience
Site services.

## Prerequisites
Access to License Manager - https://licensemanager.kcura.corp/

Front end access to Relativity instance

## Procedures
•	Updating an Existing License

o	Take a Backup of License Table

o	Delete the Existing License that is to be replaced (from SQL primary database)

o	Generate and Apply New Licenses

o	Backout Procedure (if necessary)
