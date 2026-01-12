# Post-Install Verification for Alerts

## Verify that the dashboard exists

Ensure that the **[Relativity] Alerts Overview** dashboard is successfully installed and visible in Kibana.

**Steps:**

1. Open Microsoft Edge.
2. Navigate to Kibana.
3. In Kibana, go to **Dashboards**.
4. Use the search bar to search for `Alerts Overview`.
5. Select **[Relativity] Alerts Overview** from the results.

**Expected Result:**

- **[Relativity] Alerts Overview** appears in the dashboard list.
- The dashboard is accessible without errors.

**Screenshot:**  
![Screenshot: Dashboard listed in Kibana](../../../resources/post-install-verification-images/alerts-overview/dashboard-listed.png)


## Verify the dashboard tag

Confirm that the correct tag is applied to the dashboard for proper categorization.

**Steps:**

1. In Kibana, go to **Dashboards**.
2. Search for `Alerts Overview` and select **[Relativity] Alerts Overview**.
3. Verify that the tag **Relativity Environment Watch** is displayed.

**Expected Result:**

- The dashboard includes the **Relativity Environment Watch** tag.

**Screenshot:**  
![Screenshot: Dashboard tag](../../../resources/post-install-verification-images/alerts-overview/dashboard-tag.png)


## Verify that health indicators are displayed

Ensure that the health indicators section is visible at the top of the dashboard and follows the correct layout.

**Steps:**
1. Click into **[Relativity] Alerts Overview** dashboard.
2. Confirm that the health indicators appear at the top of the main content area.
3. Confirm all tiles include a **title**, **subtitle ("Healthy")**, and **color status**.

**Expected Result:**
- Health indicators are present and aligned at the top.
- All indicators include:
  - Title (e.g., "Agents", "Monitoring")
  - Subtitle: *Healthy*
  - Color: Green or Red based on alert state.

**Screenshot:**  
![Screenshot: Health indicators at top](../../../resources/post-install-verification-images/alerts-overview/health-indicators-overview.png)


## Verify individual health indicator status

Verify the status and formatting of each health indicator tile, based on alert-driven logic and the color coding conventions.

**Steps:**
1. Open the **[Relativity] Alerts Overview** dashboard.
2. Locate each of the following indicators and confirm the title, subtitle, and color.

**Expected Result:**
- **Green** = No active alerts (Healthy)
- **Red** = Active alerts present (Unhealthy)

> All health indicators should display either Green or Red status after the initial wait period. If any health indicators aren't displaying correctly after waiting the full 10-15 minutes, verify that all related services are running properly and data collection is functioning correctly.

**Example Screenshot:**  
![All Health Indicators](../../../resources/post-install-verification-images/alerts-overview/all-health-indicators.png)


## Verify dashboard in a time range

Ensure that the dashboard is using a custom 15-minute time range as required for health indicators.

**Steps:**
1. On the dashboard, navigate to one of the alerts.
2. Locate the time filter at the top right.
3. Click on the time range selector.
4. Select **Apply custom time range**.
5. Set the range to the **last 15 minutes**.
6. Apply changes.

**Expected Result:**
- The time range reflects the last 15 minutes.
- Health indicators update dynamically based on this range.

**Screenshot:**  
![Screenshot: Time range 15 minutes](../../../resources/post-install-verification-images/alerts-overview/time-range-15-minutes.png)
