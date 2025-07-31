# Elastic Cluster Health
---

## Prerequisites

**Important:** After installation, wait 10–15 minutes before starting the verification process. This allows time for:
- All services to fully initialize
- Data collection to begin
- Health indicators to show accurate statuses

---

## Navigation

* [Verify Dashboard Exists](#verify-dashboard-exists)
* [Verify Dashboard Tag](#verify-dashboard-tag)
* [Verify Cluster Health Summary](#verify-cluster-health-summary)
* [Verify Node Metrics](#verify-node-metrics)
* [Verify Index Statistics](#verify-index-statistics)
* [Verify JVM and GC Metrics](#verify-jvm-and-gc-metrics)
* [Verify Disk and Storage Utilization](#verify-disk-and-storage-utilization)
* [Verify Visualization Format and Compliance](#verify-visualization-format-and-compliance)
* [Verify Screen Resolution for the Dashboard](#verify-screen-resolution-for-the-dashboard)
* [API-Based Cluster Health Check](#api-based-cluster-health-check)

---

## Verify Dashboard Exists

**Description:**
Confirm that the Elastic Cluster Dashboard has been installed and is visible in Kibana.

**Steps:**
1. Open **Microsoft Edge**.
2. Navigate to Kibana.
3. Go to **Observability → Dashboards**.
4. Search for `Elastic Cluster Dashboard`.

**Expected Result:**
* The dashboard titled `Elastic Cluster Dashboard` is visible and accessible.

**Screenshot:**
![Screenshot: Dashboard found in list](./screenshots/elastic-dashboard-list.png)

---

## Verify Dashboard Tag

**Description:**
Ensure that the dashboard is appropriately tagged for identification and filtering.

**Steps:**
1. In the Kibana dashboard list, locate `Elastic Cluster Dashboard`.
2. Verify the tag labeled `Cluster Monitoring`.

**Expected Result:**
* Dashboard has the `Cluster Monitoring` tag.

**Screenshot:**
![Screenshot: Dashboard tag](./screenshots/elastic-dashboard-tag.png)

---

## Verify Cluster Health Summary

**Description:**
Ensure the cluster health summary is displayed with accurate health status and color-coding.

**Steps:**
1. Open `Elastic Cluster Dashboard`.
2. Locate the **Cluster Health Summary** section at the top.
3. Validate that the health status (Green, Yellow, or Red) is displayed.

**Expected Result:**
* Cluster health is displayed using a **Health Indicator**.
* Color matches cluster status:
  * Green: Healthy
  * Yellow: Warnings (e.g., unassigned replicas)
  * Red: Critical issues

**Screenshot:**
![Screenshot: Cluster Health Summary](./screenshots/cluster-health-summary.png)

---

## Verify Node Metrics

**Description:**
Validate that node-level metrics such as CPU, memory, and load average are displayed correctly.

**Steps:**
1. Scroll to the **Node Metrics** section.
2. Ensure each node is listed with its:
   * CPU %
   * Memory %
   * Load Average
   * JVM usage

**Expected Result:**
* All running nodes are listed.
* No metrics display N/A or missing values.

**Screenshot:**
![Screenshot: Node Metrics](./screenshots/node-metrics.png)

---

## Verify Index Statistics

**Description:**
Ensure the section on indices includes total document counts, index sizes, and indexing/search rates.

**Steps:**
1. Scroll to the **Index Summary** or **Indexing Performance** panel.
2. Review:
   * Total indices
   * Document count
   * Index size
   * Indexing rate
   * Search query latency

**Expected Result:**
* Metrics update in near real time.
* All major indices show valid stats.

**Screenshot:**
![Screenshot: Index Statistics](./screenshots/index-stats.png)

---

## Verify JVM and GC Metrics

**Description:**
Ensure that JVM heap usage and garbage collection activity are correctly visualized.

**Steps:**
1. Scroll to **JVM Memory & GC** section.
2. Verify:
   * JVM heap usage graph
   * GC count and time per node

**Expected Result:**
* JVM heap usage graph shows consistent data.
* GC metrics are plotted without gaps.

**Screenshot:**
![Screenshot: JVM and GC Metrics](./screenshots/jvm-gc-metrics.png)

---

## Verify Disk and Storage Utilization

**Description:**
Ensure disk space usage is being accurately tracked and visualized.

**Steps:**
1. Locate the **Disk Usage by Node** panel.
2. Confirm each node's:
   * Total disk space
   * Used vs. available
   * Usage % in graphical or tabular format

**Expected Result:**
* Each node has visible and accurate disk usage data.
* No fields marked as N/A.

**Screenshot:**
![Screenshot: Disk Utilization](./screenshots/disk-usage.png)

---

## Verify Visualization Format and Compliance

**Description:**
Confirm that all visual elements follow Elastic guidelines for color, font, and label settings.

**Steps:**
1. For each tile or graph:
   * Verify font color = black (default)
   * Font size = default
   * Decimal places = 0
   * Health colors use:
     * Green (#58D8B5)
     * Red (#FAA0A0)
     * Gray (#D3D3D3)
2. Confirm:
   * Health tiles use “Titles and Text” for subtitles.
   * Color mapping is dynamic: **Color by value**.

**Expected Result:**
* Visuals comply with UI/UX standards.
* Health indicators are visually consistent.

**Screenshot:**
![Screenshot: Visualization Format](./screenshots/visualization-format.png)

---

## Verify Screen Resolution for the Dashboard

**Description:**
Ensure the dashboard layout and formatting are optimized for the recommended screen resolution of **1920x1080**, ensuring complete and uncluttered visibility.

**Steps:**
1. Login to **Kibana**.
2. Navigate to **Observability → Dashboard**.
3. Open the `Elastic Cluster Dashboard`.
4. Press **F12** to open Developer Tools in **Chrome** or **Edge**.
5. Enable **Device Emulation**.
6. Set the device resolution to **1920x1080**.

**Expected Result:**
- The dashboard fits within a 1920x1080 resolution.
- No scrollbars are required.
- All components remain properly aligned and legible.

**Screenshot:**
*(Optional: Include a screenshot of developer tools with the resolution applied)*

---

## API-Based Cluster Health Check

**Description:**  
Verify Elasticsearch backend connectivity and cluster health using a direct API call (for technical validation).

**Steps:**  
Run the following `curl` command from a secure terminal:

```bash
curl -u <username>:<password> -X GET "https://<hostname_or_ip>:9200/_cluster/health" -H "Content-Type: application/json"
```

**Expected Result:**
- Status code: `200 OK`
- A JSON response with cluster health details, like the example below:

<!-- ```json
{
  "cluster_name": "elasticsearch",
  "status": "green",
  "timed_out": false,
  "number_of_nodes": 3,
  "number_of_data_nodes": 3,
  "active_primary_shards": 10,
  "active_shards": 20,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 0,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 100.0
}
``` -->

- Key fields to look for:
  - `"status"` should be `green` (ideal), `yellow` (acceptable), or `red` (problem).
  - `"number_of_nodes"` and `"active_shards"` confirm data nodes are active and functioning.


---

