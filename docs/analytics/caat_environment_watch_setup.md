# Setting Up OpenTelemetry Java Agent for Relativity Analytics Engine (CAAT)

![](../resources/caat_environment_watch_setup.png)

This guide provides step-by-step instructions for installing/updating Relativity Analytics Engine (CAAT) along with the OpenTelemetry Java Agent. This enables distributed tracing and observability with minimal code changes.

---

## 1. Why Use `opentelemetry-javaagent`?

- **Automatic Instrumentation:** The OpenTelemetry Java Agent automatically instruments supported libraries and frameworks, capturing telemetry data (traces, metrics, logs) without code changes.
- **Observability:** It provides deep visibility into application performance, dependencies, and bottlenecks.
- **Standardization:** OpenTelemetry is a vendor-neutral standard, ensuring compatibility with various observability backends (e.g., Azure Monitor, Jaeger, Zipkin).

---

## 2. How to install/update CAAT

    1.  Copy CAAT Bundle to your server
    2.  Unzip the folder
    3.  Replace response-file.properties with master copy
    4.  Open Powershell as an administrator
    5.  Run .\Install.cmd
    6.  Once the installation is done, start “Relativity Analytics Engine” and check if the page is loading.
    7.  Open ElasticSearch and search for service.name: “relsvr_caat” in metrics-* data view


---

## 3. What's updated

- Startup.cmd file is updated to include the OpenTelemetry Java Agent. This references the `opentelemetry-javaagent.jar` file, which is used to instrument the CAAT service for telemetry data collection.
- Check for these lines within `startup.cmd`:
```
-javaagent:C:\\CAAT\\bin\\opentelemetry-javaagent.jar 
-Dotel.service.name="relsvr.caat" 
-Dotel.metrics.exporter=otlp 
-Dotel.exporter.otlp.endpoint="http://localhost:4318" 
-Dotel.instrumentation.runtime-metrics.enabled=true
```
---

## 4. Why Auto-Instrumentation is Safe

- **Non-Intrusive:** The agent works by attaching to the JVM at startup, using bytecode manipulation. It does not require code changes or recompilation.
- **Widely Adopted:** OpenTelemetry Java Agent is used in production by many organizations and is actively maintained.
- **Configurable:** Instrumentation can be enabled/disabled for specific libraries, and the agent can be removed by simply omitting the `-javaagent` flag.
- **No Persistent Changes:** The agent does not persistently modify your application or its dependencies.

---

## 5. How to Verify the Changes

1. **Check Application Logs:** On startup, the agent logs its initialization. Look for lines mentioning `opentelemetry-javaagent` in the CAAT logs.
2. **Verify Telemetry Export:** Confirm that traces and metrics are being sent to your configured backend (e.g., OpenTelemetry Collector, Jaeger, Azure Monitor).
3. **Check Service Name:** Ensure the service appears as `caat-analytics-engine` (or your configured name) in your observability backend.
4. **Disable to Compare:** Temporarily remove the `-javaagent` flag and confirm that telemetry stops, verifying the agent is responsible for the data.

---

**Note:**  
This documentation uses the active document as context because you have the checkmark checked.  
You can include additional context using **#** references. Typing **#** opens a completion list of available context.
