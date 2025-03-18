# Installing the telemetry backend software (Elasticsearch, Kibana, and APM Server)

## Summary

This first step for setting up Environment Watch involves installing the required telemetry backend software components: Elasticsearch, Kibana, and APM Server.

## Prerequisites

- Download [Elasticsearch, Kibana, and APM Server](https://www.elastic.co/downloads/past-releases) from Elastic's website.
- The minimum Relativity major version and patch is installed on all servers in the environment. See the [release bundle](https://github.com/relativityone/server-environment-watch-releases/releases) requirements for the minimum version required.
> [!NOTE]
> In this document, have considered a single cluster Elasticsearch to simplify the setup.

## Elasticsearch System Requirements

The number of nodes required and the specifications for the nodes change depends on both the infrastructure tier and the amount of data client plan to store in Elasticsearch.
Below table provides brief information related to storage requirements.

| Node type                | # of nodes needed | CPU | RAM | DISK (GB) |
|--------------------------|-------------------|-----|-----|-----------|
| Master - Audit           | 1                 | 4   | 32  | 1000      |
| Data - Audit             | 2                 | 4   | 32  | 1000      |
| Data - Environment Watch | 1                 | 4   | 32  | 2000      |

![](/resources/RelativityCentricElasticsearchCluster.png)
> [!NOTE]
> These recommendations are for Audit and Environment Watch only.

## Elasticsearch Key Concepts

### Elasticsearch Cluster

- Refer [Elasticsearch Cluster](https://www.elastic.co/guide/en/elasticsearch/reference/current/high-availability.html) for more details on cluster.
- An Elasticsearch cluster is a group of one or more node instances that are connected together. The power of an Elasticsearch cluster lies in the distribution of tasks, searching and indexing, across all the nodes in the cluster.
- Clustering enables Elasticsearch nodes to work together to ensure high availability when one or more nodes are down.
- An Elasticsearch cluster can continue operating normally if some of its nodes are unavailable or disconnected, as long as there are enough well-connected nodes to ensure high resilience and improved search performance.
- When Elasticsearch is designed with multiple clusters, located in different geographies. If there is a risk of single cluster going offline due to a disaster, second cluster will take over without interruption in the operation and compromising on search performance.

### Elasticsearch Nodes

- Refer [Elasticsearch Nodes](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-node.html) for more details on nodes.
- An Elasticsearch node is a single server that is a part of a cluster. A node stores data and participates in the cluster's indexing and search capabilities. An Elasticsearch node can be configured in different ways.
- Master Node: controls the Elasticsearch cluster and is responsible for all cluster-wide operations like creating/deleting an index and adding/removing nodes.
- Data Node  : stores data and executes data-related operations such as search and aggregation.
- Ingest Node: perform common transformations on data before indexing.

### Authentication

- Refer, [Elasticsearch install certificate](/docs/elasticsearch_certificates_setup.md) for more details.
- For communication between Relativity(e.g. Audit->Agent/Web Servers) and Elasticsearch, authentication part is critically important. 
- The [Custom Realms](https://www.elastic.co/guide/en/elasticsearch/reference/current/custom-realms.html) feature was used to authenticate between Relativity and Elasticsearch in prior releases.
    - This was the feature that required a Platinum license.
- Starting with Server 2024 Patch 1, Relativity can now use an API key to authenticate with Elasticsearch v7.17+ or v8.17+.
- When Elasticsearch cluster is configured in production environment, it is a best practice to install Elasticsearch certificates. 
- The admin must create Elasticsearch certificates and must be installed within each server wherever Elasticsearch is used.(e.g. Audit->Agent/Web Servers, Environment Watch->All monitored servers).

> [!WARNING] 
> Failing to install Elasticsearch certificates in production environment, components related to Environment Watch and DataGrid/Audit will not work.

## Installing Elasticsearch

- Please refer [Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html) for setup guidance.
- Before proceeding with the setup, it is imperative to verify some of the hardware requirements for Elasticsearch:
    - Ensure to increase the heap size if Elasticsearch is running out of memory.
    - Ensure that there is sufficient disk space available. Elasticsearch requires adequate disk space for indexing and storing data.
- From Relativity perspective, there are 3 use-cases when Elasticsearch is setup:
    - DataGrid and Environment Watch
    - Environment Watch Only
    - DataGrid Only

    ### Setting up an Elasticsearch cluster for Environment Watch only
    - As part of production setup, multiple nodes are configured to install Elasticsearch, Kibana and APM Server instances.
    - Complete system configuration which is required before installing Elasticsearch.
    - Verify in the PowerShell window, Elastic search configuration token gets generated, which indicates completion of the step.
    ![](/resources/ElasticSearchToken.png)

    ### Setting up an Elasticsearch cluster for Data Grid only
    - From production point of view, the setup employs multiple nodes configuration for Elasticsearch cluster. Please refer [Elasticsearch Nodes](https://www.elastic.co/guide/en/elasticsearch/reference/current/add-elasticsearch-nodes.html) for more details.
    - From Elasticsearch configuration point of view, a single cluster, with multiple nodes are configured, in which one acts as master node and others as data nodes.
    - In this setup, master and data nodes will be assigned to DataGrid servers.
    - From DataGrid side, verify Relativity front end, if Audit page is able to fetch data from Elasticsearch.
    ![](/resources/AuditPageWorkingOrNot.png)

    ### Setting up an Elasticsearch cluster for Environment Watch and Data Grid
    - From production point of view, the setup employs multiple nodes configuration for Elasticsearch cluster. Please refer [Elasticsearch Nodes](https://www.elastic.co/guide/en/elasticsearch/reference/current/add-elasticsearch-nodes.html) for more details.
    ![](/resources/ElasticsearchMultiNode.png)
    - From Elasticsearch point of view, a single cluster, with 2 or more nodes are configured, in which one acts as master node and others as data nodes.
    - In this setup, master node and a data node can be assigned for DataGrid servers(Elasticsearch and DataGrid) and other data node can be assigned for Environment Watch Server(which consists of Elasticsearch, Kibana and APM-Server).

    ### Verifying Elasticsearch
    - One must create and install Elasticsearch certificates in all servers where Elasticsearch is used. 
    - After installation of Elasticsearch is complete, start the Elasticsearch Windows service and verify it's running. If not properly installed, the Window service will fail to start.
    ![](/resources/ElasticsearchWindowService.png)
    - Log on to SQL Primary Server, run the Elasticsearch endpoint through Edge browser. 
    - Navigate to the installed Elasticsearch URL (example: https://elasticsearch.relativity.com:9200). It runs on port 9200 by default. Verify the response for the installed Elasticsearch version and cluster name.
    ![](/resources/ElasticSearchUpAndRunning.png)
    - Similarly, verify Elasticsearch endpoint can be accessed and above response can be verified in each of the following servers.
        - SQL Distributed
        - Web Servers
        - Agent servers
        - Other (File share, Analytics, Message Broker, Worker, etc.)
    - If ran into any issue, Please refer [TroubleShooting Guide](/docs/troubleshooting.md) doc for resolution.

> [!NOTE]
> Deciding and assigning number of nodes for the DataGrid and Environment Watch, will be done by Relativity Admin.

## Installing Kibana

### Setting up of Kibana
- Please refer [Kibana](https://www.elastic.co/guide/en/kibana/current/install.html) for setup guidance.
- Extract the Kibana to a desired folder, complete the initial configuration from the setup link.
- Run Kibana batch file after downloading from official Elasticsearch page.
![](/resources/KibanaLink.png)
- Copy paste the Kibana link in a new browser tab and paste the enrollment token obtained from Elasticsearch installation.
![](/resources/KibanaEnrollmentToken.png)
- Once the configure button is clicked, Kibana will be successfully installed as below.
![](/resources/KibanaInstallSuccess.png)
- When Kibana is setup, it is not installed as Windows service. To proceed with that, download NSSM service installer and install it as Windows service.

### Verifying Kibana
- Start the Elasticsearch Windows service. 
- Start Kibana Windows service and verify if service is up and running.
- Open Edge browser, navigate to the installed Kibana URL (example: https://kibana.relativity.com:5601). It runs on port 5601 by default. Verify, Kibana home page opens up.
![](/resources/KibanaWindowService.png)
- If ran into any issue, Please refer [TroubleShooting Guide](/docs/troubleshooting.md) doc for resolution.

## Installing APM Server

### Setting up of APM
- Please refer [APM Server](https://www.elastic.co/guide/en/observability/current/get-started-with-apm-server-binary.html) for setup guidance.
- Make necessary configuration changes in APM Server yml file related to Elasticsearch details/credentials. This step is important because through APM Server, data is sent/stored in Elasticsearch and can be seen through user interface Kibana.
![](/resources/APM-Elasticsearch-Kibana.png)
- Complete all step by step setup of APM server related to configuration and then install as Windows service with admin rights.

### Verifying APM
- Please make sure, Elasticsearch, Kibana Windows service are up and running. 
- Run the APM Server Windows service.
![](/resources/ApmWindowService.png)
- Open Edge browser, navigate to the installed APM Server URL (example: https://elasticsearch-apm.relativity.com:8200/). It runs on port 8200 by default.
- Verify for the response and observe for most critical part 'publish_ready' is true. If the 'publish_ready' is true in the response, that should indicate the APM Server is successfully installed. 
![](/resources/PublishReady.png)
- If ran into any issue, Please refer [TroubleShooting Guide](/docs/troubleshooting.md) doc for resolution.

## Next Steps
- After successful setup/verification of Elasticsearch, Kibana and APM Server, please proceed with the [Relativity Server CLI Setup](/docs/cli_environmentwatch_setup.md).





