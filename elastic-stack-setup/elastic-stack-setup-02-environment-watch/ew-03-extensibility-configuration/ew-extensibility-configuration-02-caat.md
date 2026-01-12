# Setting Up OpenTelemetry Java Agent for Relativity Analytics Engine (CAAT)


This guide provides step-by-step instructions for installing/updating Relativity Analytics Engine (CAAT) along with the OpenTelemetry Java Agent. This enables distributed tracing and observability with minimal code changes.

For other Other integrations, refer to the [Environment Watch Install other Integrations Guide](./ew-03-extensibility-configuration.md).

## Why use opentelemetry-javaagent?

- **Automatic Instrumentation:** The OpenTelemetry Java Agent automatically instruments supported libraries and frameworks, capturing telemetry data (traces, metrics, logs) without code changes.
- **Observability:** It provides deep visibility into application performance, dependencies, and bottlenecks.
- **Standardization:** OpenTelemetry is a vendor-neutral standard, ensuring compatibility with various observability backends (e.g., Azure Monitor, Jaeger, Zipkin).

## Why Auto-Instrumentation is safe

- **Non-Intrusive:** The agent works by attaching to the JVM at startup, using bytecode manipulation. It does not require code changes or recompilation.
- **Widely Adopted:** OpenTelemetry Java Agent is used in production by many organizations and is actively maintained.
- **Configurable:** Instrumentation can be enabled/disabled for specific libraries, and the agent can be removed by simply omitting the `-javaagent` flag.
- **No Persistent Changes:** The agent does not persistently modify your application or its dependencies.

## How to install/update CAAT

### Prerequisites

- Existing CAAT(4.7.11.A11) installer package
- CAAT EW bundle (contains `opentelemetry-javaagent.jar`, `startup.cmd`, and `replace_startup.bat`)
- Administrative access to the server

### Installation Steps

1.  Copy the CAAT EW bundle to your server and unzip it
2.  Copy the following files from the CAAT EW bundle to the CAAT installer directory:
    - `opentelemetry-javaagent.jar`
    - `startup.cmd`
    - `replace_startup.bat`
3.  Replace `response-file.properties` with your master copy
4.  Ensure no analytics jobs are currently running, then stop the **Relativity Analytics Engine** service
5.  Stop the **Relativity Environment Watch** service before starting installation
6.  Open PowerShell as an administrator
7.  Run `.\Install.cmd`
8.  Once the installation is complete, start the **Relativity Analytics Engine** and **Relativity Environment Watch** services and verify the engine is active
9.  Open Kibana and search for `service.name: "relsvr_caat"` in the `metrics-*` data view to confirm telemetry is being collected


## What's updated

- The Startup.cmd file is updated to include the OpenTelemetry Java Agent. This references the `opentelemetry-javaagent.jar` file, which is used to instrument the CAAT service for telemetry data collection.
- Check for these lines within `startup.cmd`:
    ```
    -javaagent:..bin\opentelemetry-javaagent.jar 
    -Dotel.service.name="relsvr.caat" 
    -Dotel.metrics.exporter=otlp 
    -Dotel.exporter.otlp.endpoint="http://localhost:4318" 
    -Dotel.instrumentation.runtime-metrics.enabled=true
    -Dotel.instrumentation.java-util-logging.enabled=false
    ```

## How to verify the changes

1. **Check Application Logs:** On startup, the agent logs its initialization. Look for lines mentioning `opentelemetry-javaagent` in the CAAT logs.
2. **Verify Telemetry Export:** Confirm that traces and metrics are being sent to your configured backend (e.g., OpenTelemetry Collector, Jaeger, Azure Monitor).
3. **Check Service Name:** Ensure the service appears as `Relativity Analytics Engine` (or your configured name) in your observability backend.