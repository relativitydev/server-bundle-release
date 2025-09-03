# RabbitMQ Integration

This guide provides information on integrating RabbitMQ with Environment Watch.


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

<details>
<summary>Expected output</summary>

```
Listing plugins with pattern ".*" ...
 Configured: E = explicitly enabled; e = implicitly enabled
 | Status: [failed to contact rabbit@localhost - status not shown]
 |/
[  ] rabbitmq_amqp1_0                  3.13.7
[  ] rabbitmq_auth_backend_cache       3.13.7
[  ] rabbitmq_auth_backend_http        3.13.7
[  ] rabbitmq_auth_backend_ldap        3.13.7
[  ] rabbitmq_auth_backend_oauth2      3.13.7
[  ] rabbitmq_auth_mechanism_ssl       3.13.7
[  ] rabbitmq_consistent_hash_exchange 3.13.7
[  ] rabbitmq_event_exchange           3.13.7
[  ] rabbitmq_federation               3.13.7
[  ] rabbitmq_federation_management    3.13.7
[  ] rabbitmq_jms_topic_exchange       3.13.7
[E ] rabbitmq_management               3.13.7
[e ] rabbitmq_management_agent         3.13.7
[  ] rabbitmq_mqtt                     3.13.7
[  ] rabbitmq_peer_discovery_aws       3.13.7
[  ] rabbitmq_peer_discovery_common    3.13.7
[  ] rabbitmq_peer_discovery_consul    3.13.7
[  ] rabbitmq_peer_discovery_etcd      3.13.7
[  ] rabbitmq_peer_discovery_k8s       3.13.7
[  ] rabbitmq_prometheus               3.13.7
[  ] rabbitmq_random_exchange          3.13.7
[  ] rabbitmq_recent_history_exchange  3.13.7
[  ] rabbitmq_sharding                 3.13.7
[  ] rabbitmq_shovel                   3.13.7
[  ] rabbitmq_shovel_management        3.13.7
[  ] rabbitmq_stomp                    3.13.7
[  ] rabbitmq_stream                   3.13.7
[  ] rabbitmq_stream_management        3.13.7
[  ] rabbitmq_top                      3.13.7
[  ] rabbitmq_tracing                  3.13.7
[  ] rabbitmq_trust_store              3.13.7
[e ] rabbitmq_web_dispatch             3.13.7
[  ] rabbitmq_web_mqtt                 3.13.7
[  ] rabbitmq_web_mqtt_examples        3.13.7
[  ] rabbitmq_web_stomp                3.13.7
[  ] rabbitmq_web_stomp_examples       3.13.7
```
**Note:** You should see `[E ] rabbitmq_management` indicating the plugin is enabled.
</details>.

## Access the Management UI

The management UI can be accessed at: `http://<hostname/ipaddress>:15672/`

## User Management with curl Commands
> [!NOTE] 
> For best compatibility on Windows, execute the following curl commands in Command Prompt (`cmd.exe`), not PowerShell. This ensures proper handling of double quotes and JSON payloads.

### Create Administrative User

```
# Create admin user with full access
curl -i -u <username>:<password> -H "content-type:application/json" -X PUT http://<hostname/ipaddress>:15672/api/users/admin_user -d "{\"password\":\"<password>\",\"tags\":\"administrator\"}"
```

<details>
<summary>Expected output</summary>

```
HTTP/1.1 201 Created
content-length: 0
content-security-policy: script-src 'self' 'unsafe-eval' 'unsafe-inline'; object-src 'self'
date: Fri, 29 Aug 2025 07:56:44 GMT
server: Cowboy
vary: accept, accept-encoding, origin
```
</details>

```
# Grant full permissions on default virtual host
curl -i -u <username>:<password> -H "content-type:application/json" -X PUT http://<hostname/ipaddress>:15672/api/permissions/%2F/admin_user -d "{\"configure\":\".*\",\"write\":\".*\",\"read\":\".*\"}"
```

<details>
<summary>Expected output</summary>

```
HTTP/1.1 201 Created
content-length: 0
content-security-policy: script-src 'self' 'unsafe-eval' 'unsafe-inline'; object-src 'self'
date: Fri, 29 Aug 2025 08:08:00 GMT
server: Cowboy
vary: accept, accept-encoding, origin
```
</details>

### Create Monitoring User (Read-Only)

```
# Create monitoring user
curl -i -u <username>:<password> -H "content-type:application/json" -X PUT http://<hostname/ipaddress>:15672/api/users/monitoring_user -d "{\"password\":\"<password>\",\"tags\":\"monitoring\"}"
```

<details>
<summary>Expected output</summary>

```
HTTP/1.1 201 Created
content-length: 0
content-security-policy: script-src 'self' 'unsafe-eval' 'unsafe-inline'; object-src 'self'
date: Fri, 29 Aug 2025 08:08:32 GMT
server: Cowboy
vary: accept, accept-encoding, origin
```
</details>

```
# Grant empty permissions (monitoring access without resource permissions)
curl -i -u <username>:<password> -H "content-type:application/json" -X PUT http://<hostname/ipaddress>:15672/api/permissions/%2F/monitoring_user -d "{\"configure\":\"^$\",\"write\":\"^$\",\"read\":\"^$\"}"
```

<details>
<summary>Expected output</summary>

```
HTTP/1.1 201 Created
content-length: 0
content-security-policy: script-src 'self' 'unsafe-eval' 'unsafe-inline'; object-src 'self'
date: Fri, 29 Aug 2025 08:09:01 GMT
server: Cowboy
vary: accept, accept-encoding, origin
```
</details>

### Create Management User (Basic Access)

```
# Create management user
curl -i -u <username>:<password> -H "content-type:application/json" -X PUT http://<hostname/ipaddress>:15672/api/users/mgmt_user -d "{\"password\":\"<password>\",\"tags\":\"management\"}"
```

<details>
<summary>Expected output</summary>

```
HTTP/1.1 201 Created
content-length: 0
content-security-policy: script-src 'self' 'unsafe-eval' 'unsafe-inline'; object-src 'self'
date: Fri, 29 Aug 2025 08:09:33 GMT
server: Cowboy
vary: accept, accept-encoding, origin
```
</details>

```
# Grant specific permissions
curl -i -u <username>:<password> -H "content-type:application/json" -X PUT http://<hostname/ipaddress>:15672/api/permissions/%2F/mgmt_user -d "{\"configure\":\"^specific_queue.*\",\"write\":\"^specific_queue.*\",\"read\":\"^specific_queue.*\"}"
```

<details>
<summary>Expected output</summary>

```
HTTP/1.1 201 Created
content-length: 0
content-security-policy: script-src 'self' 'unsafe-eval' 'unsafe-inline'; object-src 'self'
date: Fri, 29 Aug 2025 08:10:08 GMT
server: Cowboy
vary: accept, accept-encoding, origin
```
</details>

## Verify User Setup

```
# List all users
curl -i -u <username>:<password> http://<hostname/ipaddress>:15672/api/users
```

<details>
<summary>Expected output</summary>

```
HTTP/1.1 200 OK
cache-control: no-cache
content-length: 884
content-security-policy: script-src 'self' 'unsafe-eval' 'unsafe-inline'; object-src 'self'
content-type: application/json
date: Fri, 29 Aug 2025 08:14:11 GMT
server: Cowboy
vary: accept, accept-encoding, origin

[{"hashing_algorithm":"rabbit_password_hashing_sha256","limits":{},"name":"admin","password_hash":"R1rPwt+kCnp/dBAl38a97mFvzxBPHVKSsrMbhcwCOjgBmHsC","tags":["administrator"]},{"hashing_algorithm":"rabbit_password_hashing_sha256","limits":{},"name":"admin_user","password_hash":"xxDctOHpGvwpwX4dHNtSfoo47crYYLL7b6LKnPlKbBQOmbOa","tags":["administrator"]},{"hashing_algorithm":"rabbit_password_hashing_sha256","limits":{},"name":"guest","password_hash":"LPKa7nr7nwIeUa4lGWtjG5Og6fi1JPFWbnjXVL3d2ReTCYsj","tags":["administrator"]},{"hashing_algorithm":"rabbit_password_hashing_sha256","limits":{},"name":"mgmt_user","password_hash":"K1tIRGiroPcXAw80MzKksRf4yBwcvAXBsnW/GgMsNrQB1GL1","tags":["management"]},{"hashing_algorithm":"rabbit_password_hashing_sha256","limits":{},"name":"monitoring_user","password_hash":"hZLpi5Ql5y+k28FVL0UdcMUPbMM0y9o/ZXtbZQCjFtjhhNM+","tags":["monitoring"]}]
```
</details>

```
# Check specific user permissions
curl -i -u <username>:<password> http://<hostname/ipaddress>:15672/api/permissions/%2F/admin_user
```

<details>
<summary>Expected output</summary>

```
HTTP/1.1 200 OK
cache-control: no-cache
content-length: 75
content-security-policy: script-src 'self' 'unsafe-eval' 'unsafe-inline'; object-src 'self'
content-type: application/json
date: Fri, 29 Aug 2025 08:12:35 GMT
server: Cowboy
vary: accept, accept-encoding, origin

{"user":"admin_user","vhost":"/","configure":".*","write":".*","read":".*"}
```
</details>

## User Permission Tags

| Tag | Access Level | Description |
|-----|-------------|-------------|
| **administrator** | Full access | Complete management access |
| **monitoring** | Read-only | View all data, no modifications |
| **management** | Basic access | Manage own virtual hosts only |

**Note:** Virtual host `/` is encoded as `%2F` in URLs.
