# Setting Up OpenTelemetry Java Agent for Relativity Analytics Engine (CAAT)

This guide provides step-by-step instructions for instrumenting the Relativity Analytics Engine (CAAT) with the OpenTelemetry Java Agent. This enables distributed tracing and observability with minimal code changes.

---

## 1. Why Use `opentelemetry-javaagent`?

- **Automatic Instrumentation:** The OpenTelemetry Java Agent automatically instruments supported libraries and frameworks, capturing telemetry data (traces, metrics, logs) without code changes.
- **Observability:** It provides deep visibility into application performance, dependencies, and bottlenecks.
- **Standardization:** OpenTelemetry is a vendor-neutral standard, ensuring compatibility with various observability backends (e.g., Azure Monitor, Jaeger, Zipkin).

---

## 2. Where to Download `opentelemetry-javaagent.jar`

- Download the latest stable release from the [OpenTelemetry Java Instrumentation GitHub Releases page](https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases).
- Look for the asset named `opentelemetry-javaagent.jar` under the desired release.

---

## 3. Where to Place the Downloaded JAR File

- Place the `opentelemetry-javaagent.jar` file in a directory accessible to the CAAT service.
    - **Recommended location:** In the same directory as your CAAT application binaries or a dedicated `lib` or `instrumentation` folder.
    - Example: `C:\Relativity\CAAT\instrumentation\opentelemetry-javaagent.jar`

---

## 4. Steps to Update the `startup.cmd` File

1. **Open** the `startup.cmd` file used to launch the CAAT service.
2. **Modify** the Java startup command to include the `-javaagent` flag, pointing to the downloaded JAR.

```    
-javaagent:C:\\CAAT\\otel\\opentelemetry-javaagent.jar 
-Dotel.service.name="relsvr.caat" ^
-Dotel.metrics.exporter=otlp ^
-Dotel.exporter.otlp.endpoint="http://localhost:4318" ^
-Dotel.instrumentation.runtime-metrics.enabled=true
```

3. **Save** the changes to `startup.cmd`.
4. **Restart** “Relativity Analytics Engine” from the services

Example:
```
"%JAVA_HOME%\bin\java.exe" -server ^
  -XX:+UseG1GC ^
  -XX:MaxGCPauseMillis=5000 ^
  -XX:StringTableSize=1000003 ^
  -XX:OnError="error.cmd %%p Error" %* ^
  -XX:OnOutOfMemoryError="error.cmd %%p OutOfMemoryError" ^
  -Xlog:gc*:file="%GC_FILE_NAME%":time,uptime:filecount=10,filesize=2M ^
  -cp "..\jetty\start.jar;%CLASSPATH%" ^
  --add-opens=java.base/java.io=ALL-UNNAMED ^
  --add-opens=java.base/java.util=ALL-UNNAMED ^
  --add-opens=java.base/java.lang=ALL-UNNAMED ^
  --add-opens=java.base/java.lang.reflect=ALL-UNNAMED ^
  --add-opens=java.desktop/java.awt.font=ALL-UNNAMED ^
  --add-modules=java.se ^
  --add-exports=java.base/sun.nio.ch=ALL-UNNAMED ^
  --add-exports=jdk.unsupported/sun.misc=ALL-UNNAMED ^
  --add-exports=java.base/sun.net.www=ALL-UNNAMED ^
  %JAVA_OPTS% %CAAT_OPTS% %HEAP_OPTS% ^
  -Dorg.slf4j.simpleLogger.defaultLogLevel=debug ^
  -Dorg.slf4j.simpleLogger.showDateTime=true ^
  -Dorg.slf4j.simpleLogger.dateTimeFormat="yyyy-MM-dd HH:mm:ss:SSS" ^
  -Dorg.slf4j.simpleLogger.showThreadName=true ^
  -Dhttp.port=%HTTP_PORT% ^
  -Djetty.host=localhost ^
  -Dhostname=%HOSTNAME% ^
  -Dsys.hostname=%HOSTNAME% ^
  -Djava.net.preferIPv4Stack=true ^
  -Dlog4j2.configurationFile=file:../etc/log4j2.xml ^
  -Dcom.contentanalyst.common.install.dir=.. ^
  -DSTOP.PORT=%STOP_PORT% ^
  -DSTOP.KEY=%STOP_KEY% ^
  -Djetty.home=..\jetty ^
  -Djetty.base=.. ^
  -Djava.io.tmpdir=..\temp ^
  -Djetty.logging.dir=..\logs ^
  -Dorg.eclipse.jetty.util.log.class=org.eclipse.jetty.util.log.StdErrLog ^
  -Djetty.ssl.sniHostCheck=false ^
  -Djetty.ssl.sniRequired=false ^
  -Dorg.eclipse.jetty.webapp.WebAppClassLoader.useServerClasses=false ^
  -Dorg.eclipse.jetty.server.webapp.parentLoaderPriority=true ^
  -Dorg.eclipse.jetty.webapp.WebAppClassLoader.allowDuplicateClasses=true ^
  -Djetty.home.lib.ext=../lib ^
  -javaagent:C:\\CAAT\\otel\\opentelemetry-javaagent.jar -Dotel.service.name="relsvr.caat" ^
  -Dotel.metrics.exporter=otlp ^
  -Dotel.exporter.otlp.endpoint="http://localhost:4318" ^
  -Dotel.instrumentation.runtime-metrics.enabled=true ^
  -Dorg.eclipse.jetty.security.authentication.crypt.algorithm=bcrypt ^
  -Dorg.eclipse.jetty.util.security.algorithm=bcrypt ^
  org.eclipse.jetty.start.Main ^
  ../etc/jetty-deploy.xml ../etc/caat-realm.xml "jetty.http.port=8081"
```

---

## 5. Why Auto-Instrumentation is Safe

- **Non-Intrusive:** The agent works by attaching to the JVM at startup, using bytecode manipulation. It does not require code changes or recompilation.
- **Widely Adopted:** OpenTelemetry Java Agent is used in production by many organizations and is actively maintained.
- **Configurable:** Instrumentation can be enabled/disabled for specific libraries, and the agent can be removed by simply omitting the `-javaagent` flag.
- **No Persistent Changes:** The agent does not persistently modify your application or its dependencies.

---

## 6. How to Verify the Changes

1. **Check Application Logs:** On startup, the agent logs its initialization. Look for lines mentioning `opentelemetry-javaagent` in the CAAT logs.
2. **Verify Telemetry Export:** Confirm that traces and metrics are being sent to your configured backend (e.g., OpenTelemetry Collector, Jaeger, Azure Monitor).
3. **Check Service Name:** Ensure the service appears as `caat-analytics-engine` (or your configured name) in your observability backend.
4. **Disable to Compare:** Temporarily remove the `-javaagent` flag and confirm that telemetry stops, verifying the agent is responsible for the data.

---

**Note:**  
This documentation uses the active document as context because you have the checkmark checked.  
You can include additional context using **#** references. Typing **#** opens a completion list of available context.
