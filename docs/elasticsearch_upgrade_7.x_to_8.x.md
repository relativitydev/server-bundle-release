# Elasticsearch Upgrade

This document provides comprehensive steps to upgrade Elasticsearch from 7.x to 8.x across multiple DataGrid servers, including master and data node configurations.

![Elasticsearch Setup](../resources/elasticsearch_setup_001.png)

> [!NOTE]
> This upgrade process should be performed when transitioning Elasticsearch from version 7.x to 8.x in a multi-node cluster environment with dedicated master and data nodes.

## Prerequisites

Before starting the upgrade process, ensure the following requirements are met:

1. **Environment Configuration**: Multiple DataGrid servers with master node on one server and data nodes on other servers
2. **Agent Management**: Disable or delete Audit Agents in Relativity before proceeding
   - Navigate to Relativity UI → Agents tab
   - Disable/Delete all Audit Agents before setup

## Pre-Installation Procedures

Perform the following steps on **all node servers** one by one:

### Step 1: Extract and Place Elasticsearch

1. Extract the Elasticsearch download archive
2. Place the extracted Elasticsearch folder on a suitable drive

   **Example**: `C:\Elastic\elasticsearch-8.x.x`

### Step 2: Environment Setup

**2.1 Configure Java Environment Variables**

Configure the Java environment variables for Elasticsearch. While Elasticsearch includes a bundled Java runtime, proper environment configuration ensures optimal performance.

For Java configuration troubleshooting and advanced setup, refer to the [Elasticsearch Troubleshooting Guide](troubleshooting/elasticsearch.md#1-windows-service-issues).

   ![Environment Variables Setup](../resources/elasticsearch_setup_002.png)

### Step 3: Install Elasticsearch Service

**3.1 Prepare Installation Environment**

1. Open **PowerShell** or **Command Prompt** in administrator mode
2. Navigate to Elasticsearch bin directory
   
   **Example**: `C:\Elastic\elasticsearch-8.x.x\bin`

**3.2 Remove Existing Elasticsearch Service (if present)**

1. **Stop the service**:
   - Right-click on **Elasticsearch 7.x.x** in Services
   - Select **Stop**

2. **Remove the service**:
   ```powershell
   .\elasticsearch-service.bat remove Elasticsearch
   ```

**3.3 Install New Elasticsearch Service**

Execute the following commands in sequence from the Elasticsearch bin directory:

1. **Setup Elasticsearch**:
   ```powershell
   .\elasticsearch.bat
   ```
   
   > [!NOTE]
   > Elasticsearch token gets generated during this step, indicating completion. Terminate execution by typing `Ctrl+C` to gracefully stop, then confirm batch job termination.

2. **Install Elasticsearch service**:
   ```powershell
   .\elasticsearch-service.bat install Elasticsearch
   ```

3. **Start Elasticsearch service**:
   ```powershell
   .\elasticsearch-service.bat start Elasticsearch
   ```
   
   > [!NOTE]
   > If the service fails to start through command, try starting manually from the Services window. If command fails with cluster unhealthy exit code 69, restart Elasticsearch Windows Service, restart PowerShell, and try again. If issues persist, restart the server and execute the command.

**3.4 Verify Service Installation**

1. Open **Local Services**
2. Verify the **Elasticsearch service** (with the recently installed version) is running
3. If not running, right-click on the Elasticsearch service and select **Start**

### Step 4: Configuration and Verification

**4.1 Configure elasticsearch.yml**

1. Navigate to the Elasticsearch config folder
   
   **Example**: `C:\Elastic\elasticsearch-8.x.x\config`

2. Open the `elasticsearch.yml` file
3. Add the following security configuration at the end of the file:

   ```yaml
   # Enable security features
   xpack.security.enabled: true
   
   xpack.security.enrollment.enabled: true
   
   # Enable encryption for HTTP API client connections, such as Kibana, Logstash, and Agents
   xpack.security.http.ssl:
     enabled: true
     keystore.path: certs/http.p12
   
   # Enable encryption and mutual authentication between cluster nodes
   xpack.security.transport.ssl:
     enabled: true
     verification_mode: certificate
     keystore.path: certs/transport.p12
     truststore.path: certs/transport.p12
   
   # Create a new cluster with the current node only
   # Additional nodes can still join the cluster later
   cluster.initial_master_nodes: ["domain_server_name"]
   
   # Allow HTTP API connections from anywhere
   # Connections are encrypted and require user authentication
   http.host: 0.0.0.0
   
   # Allow other nodes to join the cluster from anywhere
   # Connections are encrypted and mutually authenticated
   #transport.host: 0.0.0.0
   ```

**4.2 Enable Stack Monitoring**

Add the following line to `elasticsearch.yml` to enable built-in Stack Monitoring dashboard:

```yaml
xpack.monitoring.collection.enabled: true
```

Save changes and restart the Elasticsearch service.

**4.3 Verify Elasticsearch Accessibility**

1. Access Elasticsearch at: `https://domain_server_name:9200/`

2. **Chrome Certificate Issue Resolution** (if needed):
   - Navigate to `chrome://net-internals/#hsts`
   - Type `domain_server_name` in the input field next to Delete button
   - Press **Delete** button
   - Query the domain to verify HSTS entries are removed
   - Try accessing the URL again

**4.4 Reset Elasticsearch Password**

Reset the elastic user password using:

```powershell
.\elasticsearch-reset-password.bat -i -u elastic
```

### Step 5: JVM Heap Configuration

Configure JVM heap memory (8-10 GB recommended):

1. **Stop** the Elasticsearch Windows service
2. Navigate to `.\config\jvm.options.d` directory in Elasticsearch installation
3. Create a new file with `.options` extension (e.g., `elasticsearch.options`)
4. Add the following content:

   ```
   -Xms8g
   -Xmx10g
   ```

5. **Save** the file
6. **Restart** the Elasticsearch Windows service
7. **Verify** the `java.exe` process memory usage is within the specified range

### Step 6: Troubleshooting and Additional Operations

**6.1 Password Management**

To change or reset passwords:

```powershell
.\elasticsearch-reset-password.bat -i -u elastic
```

- Confirm with **Y** to change or **N** to keep current password
- Enter new password and re-enter for confirmation

**6.2 Service Management Commands**

- **Start service**:
  ```powershell
  .\elasticsearch-service.bat start Elasticsearch
  ```

- **Stop service**:
  ```powershell
  .\elasticsearch-service.bat stop Elasticsearch
  ```

- **Remove service**:
  ```powershell
  .\elasticsearch-service.bat remove Elasticsearch
  ```

**6.3 SSL Certificate Configuration**

For each node server where Elasticsearch URL is not secure, follow the certificate import procedures detailed in the [Certificate Troubleshooting Guide](troubleshooting/pre-requisite-troubleshooting.md#certificate-troubleshooting).

## Multi-Node Cluster Configuration

### Master Node Configuration

Update the `elasticsearch.yml` file on the master node with the following parameters:

```yaml
# Cluster configuration - must be same across all nodes
cluster.name: your-cluster-name

# Master node role
node.roles: [ master ]

# Discovery configuration
discovery.seed_hosts: ["master_node_domain", "data_node1_domain", "data_node2_domain"]
cluster.initial_master_nodes: ["master_node_domain"]

# Network configuration
http.host: 0.0.0.0
transport.host: 0.0.0.0
network.host: 0.0.0.0

# Additional settings
ingest.geoip.downloader.enabled: false
transport.compress: true
transport.port: 9300
```

### Data Node Configuration

Update the `elasticsearch.yml` file on each data node with the following parameters:

```yaml
# Cluster configuration - must be same across all nodes
cluster.name: your-cluster-name

# Data node role
node.roles: [ data ]

# Discovery configuration
discovery.seed_hosts: ["master_node_domain", "data_node1_domain", "data_node2_domain"]
cluster.initial_master_nodes: ["master_node_domain"]

# Network configuration
http.host: 0.0.0.0
transport.host: 0.0.0.0
network.host: 0.0.0.0

# Data and log paths (avoid C: drive or temporary storage)
path.data: X:/ElasticData
path.logs: X:/ElasticLogs

# Additional settings
ingest.geoip.downloader.enabled: false
transport.compress: true
transport.port: 9300
```

> [!NOTE]
> Replace `X:` with an appropriate drive letter that is not the C: drive or temporary storage drive.

## Certificate and Security Setup

Before proceeding with cluster formation, ensure Elasticsearch services are **stopped** on all node servers.

### Step 1: Create Certificate Authority (CA)

Perform on the **master node server**:

1. Open **PowerShell** in admin mode
2. Navigate to Elasticsearch bin folder
3. Run the following command:

   ```powershell
   .\elasticsearch-certutil ca
   ```

This generates `elastic-stack-ca.p12` file (root CA certificate).

### Step 2: Generate Node Certificates

1. Open **PowerShell** in admin mode
2. Navigate to Elasticsearch bin folder
3. Run the following command:

   ```powershell
   .\elasticsearch-certutil cert --ca elastic-stack-ca.p12
   ```

During execution:
- **Certificate Name**: Provide unique name for each node (e.g., node1, node2)
- **Password**: Set password (use same password for all nodes)

4. **Repeat** for each node in the cluster
5. **Copy** each certificate to its respective node server in the same directory

### Step 3: Configure Keystore for Password Management

Perform on **all DataGrid servers** using the same password:

**Remove existing passwords**:

```powershell
.\elasticsearch-keystore remove xpack.security.transport.ssl.keystore.secure_password
.\elasticsearch-keystore remove xpack.security.transport.ssl.truststore.secure_password
```

**Add new passwords**:

```powershell
.\elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
.\elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
```

Enter the password created during certificate generation for both prompts.

### Step 4: Clean Node Data Folders

1. Navigate to Elasticsearch data folder
2. **Delete** all subfolders and files within the data directory
3. If deletion fails, restart the server and try again

### Step 5: Configure User Authentication

**Check existing users**:

```powershell
.\elasticsearch-users list
```

**Generate user credentials** (if no users found):

```powershell
.\elasticsearch-setup-passwords auto
```

Type **y** when prompted and save the generated credentials securely.

### Step 6: Install Mapper Plugin

**With internet connection**:

```powershell
.\elasticsearch-plugin install mapper-size
```

**For offline installation**:
1. Download: `https://artifacts.elastic.co/downloads/elasticsearch-plugins/mapper-size/mapper-size-8.x.x.zip`
2. Extract to: `C:\elasticsearch-8.x.x-windows-x86_64\elasticsearch-8.x.x\plugins` folder
3. Adjust version number accordingly

## Cluster Startup and Verification

### Step 1: Start Services

**Master Node**:
1. Start the Elasticsearch service via Services window
2. Access master node URL: `https://<master-node-domain>:9200`

**Data Nodes**:
1. Start each data node service one by one via Services window
2. Check status by accessing: `https://<data-node-domain>:9200`

### Step 2: Verify Cluster Formation

Run the following API on the master node to verify connected nodes:

```
https://<master-node-domain>:9200/_cat/nodes
```

This displays a list of all connected nodes in the cluster.

## Advanced Configuration

### Specific Data Type Allocation

To allocate specific index types to particular nodes:

**Step 1**: Add the following parameter to `elasticsearch.yml` for the target node:

```yaml
node.attr.audit: true
```

**Step 2**: Restart the Elasticsearch service

**Step 3**: In Relativity UI, navigate to **Instance Settings** → **ESIndexCreationSettings**

**Step 4**: Add the following key-value under settings:

```json
"routing.allocation.require.audit": "true"
```

This configuration ensures indexes starting with "audit" are allocated to the specified node.

## Next Steps

After completing the Elasticsearch upgrade:

1. **Verify cluster health** using the `_cat/health` API
2. **Test connectivity** from all application servers
3. **Update Relativity configurations** to point to the new Elasticsearch cluster
4. **Restart Relativity services** on all servers
5. **Monitor cluster performance** and adjust configurations as needed

For troubleshooting issues during or after the upgrade, refer to the [Troubleshooting Guide](troubleshooting/pre-requisite-troubleshooting.md).