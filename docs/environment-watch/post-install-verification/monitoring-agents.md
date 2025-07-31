# [Relativity] Monitoring Agents Dashboard Post-Install Verification
---

## Prerequisites

**Important:** After installation, wait 10–15 minutes before starting the verification process. This allows time for:
- All services to fully initialize
- Data collection to begin
- Health indicators to show accurate statuses

---

## Navigation

* [Verify Monitoring Agent Dashboard Exists](#verify-monitoring-agent-dashboard-exists)
* [Verify Monitoring Agents Dashboard Tags](#verify-monitoring-agents-dashboard-tags)
* [Verify Dashboard Filters Populate with Data](#verify-dashboard-filters-populate-with-data)
* [Verify Visualization Order](#verify-visualization-order)
* [Verify Time Series Graph Is Line Chart](#verify-time-series-graph-is-line-chart)
* [Verify Data for Multiple Time Ranges](#verify-data-for-multiple-time-ranges)
* [Verify Data Population](#verify-data-population)
* [Verify Hosts and Agent Versions](#verify-hosts-and-agent-versions)
* [Verify Data in Discover](#verify-data-in-discover)
* [Verify Screen Resolution for the Dashboards](#verify-screen-resolution-for-the-dashboards)
* [Example API Health Check (Optional)](#example-api-health-check-optional)

---

## Verify Monitoring Agent Dashboard Exists

**Description:**
Ensure the Monitoring Agent dashboard is present.

**Steps:**
1. Login to Kibana.
2. Navigate to **Analytics → Dashboard**.

**Expected Result:**
* "Monitoring Agent" is listed.

**Screenshot:**
![Screenshot: Dashboard exists](./screenshots/dashboard-exists.png)

---

## Verify Monitoring Agents Dashboard Tags

**Description:**
Ensure the correct tags are assigned to the dashboard.

**Steps:**
1. Login to Kibana.
2. Navigate to **Observability → Dashboard**.
3. Open the Monitoring Agents dashboard.

**Expected Result:**
* Tags:
  * `Relativity Environment Watch`
  * `FeatureDomain: Monitoring`

**Screenshot:**
![Screenshot: Tags](./screenshots/dashboard-tags.png)

---

## Verify Dashboard Filters Populate with Data

**Description:**
Ensure filter dropdowns are populated with available data.

**Steps:**
1. Login to Kibana.
2. Navigate to **Observability → Dashboard**.
3. Click on the dashboard and open each filter dropdown.

**Expected Result:**
* Filter dropdowns show available values.

**Screenshot:**
![Screenshot: Filter dropdown populated](./screenshots/filter-dropdown-populated.png)

---

## Verify Visualization Order

**Description:**
Confirm the visualizations appear in the correct order.

**Steps:**
1. Login to Kibana.
2. Navigate to **Observability → Dashboard**.
3. Open the dashboard.

**Expected Result:**
* First: Health/metric indicators (if present)
* Second: Table visualizations
* Third: Time series graph

**Screenshot:**
![Screenshot: Visualization order](./screenshots/visualization-order.png)

---

## Verify Time Series Graph Is Line Chart

**Description:**
Ensure the graph uses Line chart type.

**Steps:**
1. Login to Kibana.
2. Navigate to **Observability → Dashboard**.
3. Open the dashboard.

**Expected Result:**
* Time series graph is a Line chart.

**Screenshot:**
![Screenshot: Line chart](./screenshots/line-chart.png)

---

## Verify Data for Multiple Time Ranges

**Description:**
Ensure the dashboard works correctly for various time ranges.

**Steps:**
1. Login to Kibana.
2. Navigate to **Observability → Dashboard**.
3. Change time ranges to 15 min, 1 hour, 12 hours, and 24 hours.

**Expected Result:**
* Data is shown for each selected time range.

**Screenshot:**
![Screenshot: Time range check](./screenshots/time-ranges.png)

---

## Verify Data Population

**Description:**
Ensure the dashboard data is loading correctly.

**Steps:**
1. Login to Kibana.
2. Open the "Monitoring Agent" dashboard.

**Expected Result:**
* All panels are populated with data.

**Screenshot:**
![Screenshot: Populated data](./screenshots/data-populated.png)

---

## Verify Hosts and Agent Versions

**Description:**
Ensure hosts and agent versions are correctly displayed.

**Steps:**
1. Login to Kibana.
2. Open the "Monitoring Agent" dashboard.

**Expected Result:**
* Host column lists multiple hosts.
* Agent Version is the same for all hosts.

**Screenshot:**
![Screenshot: Hosts and versions](./screenshots/hosts-agent-versions.png)

---

## Verify Data in Discover

**Description:**
Ensure dashboard data is reflected in Discover.

**Steps:**
1. Login to Kibana.
2. Open the dashboard → Monitoring Agents table → Three dots → Explore in Discover.

**Expected Result:**
* Data is visible in Discover.

**Screenshot:**
![Screenshot: Discover view](./screenshots/discover-view.png)

---

## Verify Screen Resolution for the Dashboards

**Description:**  
Ensure the dashboard layout and formatting are optimized for the recommended screen resolution of **1920x1080**, which ensures full visibility of all visual elements without scrollbars or layout distortion.

**Steps:**
1. Login to **Kibana**.
2. Navigate to **Observability → Dashboard**.
3. Click on the "Monitoring Agent" dashboard.
4. Press **F12** to open Developer Tools in **Chrome** or **Edge**.
5. Toggle the **Device Emulation** feature.
6. Set the device dimensions to **1920x1080**.

**Expected Result:**
- The dashboard fits within a 1920x1080 resolution.
- No horizontal or vertical scrollbars are required to view core components.
- All visualizations remain properly aligned.

**Screenshot:**  
*(Optional: Add a screenshot showing the dev tools and resolution setting if available)*

---

## Example API Health Check (Optional)

**Description:**  
Verify dashboard backend connectivity using a direct API call to Elasticsearch/Kibana for health indicators (for technical validation).

**Steps:**  
Run the following `curl` command from a secure terminal:

```bash
curl -u <username>:<password> -X GET "https://<hostname_or_ip>:5601/api/saved_objects/_find?type=dashboard&search_fields=title&search=Monitoring%20Agent" -H 'kbn-xsrf: true'
```

**Expected Result:**
- A JSON response returns the dashboard object.
- Status code `200 OK`.

---
