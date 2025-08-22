# Install Other Integrations

![](../resources/caat_environment_watch_setup.png)

This guide provides information on integrating various Relativity components with Environment Watch. These integrations extend observability capabilities across your environment, enabling comprehensive monitoring and performance analysis.


### RabbitMQ Management Plugin

## Prerequisites

- RabbitMQ server installed and running
- Administrative access to the RabbitMQ server
- Access to command line tools (`rabbitmq-plugins`, `curl`)
- Default admin user credentials (default: `guest`/`guest` for localhost only)
- For production: Create a dedicated admin user for API access

## Enable the Management Plugin

```powershell
# Enable the management plugin
rabbitmq-plugins enable rabbitmq_management
```

**Note:** Node restart is not required after plugin activation.

## Verify Plugin Status

```powershell
# Check if the plugin is enabled
rabbitmq-plugins list

# Check listening ports
rabbitmq-diagnostics -s listeners
```

Expected output should show the management plugin listening on port 15672:
```
# => Interface: [::], port: 15672, protocol: http, purpose: HTTP API
```

## Access the Management UI

The management UI can be accessed at: `http://<hostname/ipaddress>:15672/`

## User Management with curl Commands

### Create Administrative User

```
# Create admin user with full access
curl -i -u <username>:<password> -H "content-type:application/json" \
  -X PUT http://<hostname/ipaddress>:15672/api/users/admin_user \
  -d '{"password":"strong_password_123","tags":"administrator"}'

# Grant full permissions on default virtual host
curl -i -u <username>:<password> -H "content-type:application/json" \
  -X PUT http://<hostname/ipaddress>:15672/api/permissions/%2F/admin_user \
  -d '{"configure":".*","write":".*","read":".*"}'
```

### Create Monitoring User (Read-Only)

```
# Create monitoring user
curl -i -u <username>:<password> -H "content-type:application/json" \
  -X PUT http://<hostname/ipaddress>:15672/api/users/monitoring_user \
  -d '{"password":"monitor_password_456","tags":"monitoring"}'

# Grant empty permissions (monitoring access without resource permissions)
curl -i -u <username>:<password> -H "content-type:application/json" \
  -X PUT http://<hostname/ipaddress>:15672/api/permissions/%2F/monitoring_user \
  -d '{"configure":"^$","write":"^$","read":"^$"}'
```

### Create Management User (Basic Access)

```
# Create management user
curl -i -u <username>:<password> -H "content-type:application/json" \
  -X PUT http://<hostname/ipaddress>:15672/api/users/mgmt_user \
  -d '{"password":"management_password_789","tags":"management"}'

# Grant specific permissions
curl -i -u <username>:<password> -H "content-type:application/json" \
  -X PUT http://<hostname/ipaddress>:15672/api/permissions/%2F/mgmt_user \
  -d '{"configure":"^specific_queue.*","write":"^specific_queue.*","read":"^specific_queue.*"}'
```

## Verify User Setup

```
# List all users
curl -i -u <username>:<password> http://<hostname/ipaddress>:15672/api/users

# Check specific user permissions
curl -i -u <username>:<password> http://<hostname/ipaddress>:15672/api/permissions/%2F
```

## User Permission Tags

| Tag | Access Level | Description |
|-----|-------------|-------------|
| **administrator** | Full access | Complete management access |
| **monitoring** | Read-only | View all data, no modifications |
| **management** | Basic access | Manage own virtual hosts only |

**Note:** Virtual host `/` is encoded as `%2F` in URLs.



## Next Step

[Click here for the next step](environment-watch/post-install-verification/elasticsearch-cluster-health.md)