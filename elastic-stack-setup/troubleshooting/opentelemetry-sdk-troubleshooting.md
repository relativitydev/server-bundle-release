# Open Telemetry SDK Bootstrapping Troubleshooting

This document explains how to find and work around Open Telemetry SDK bootstrapping failures in custom pages, event handlers, and web applications.

> [!NOTE]
> This guide applies to Relativity Server 2026 and later.

Custom pages, event handlers, and web applications bootstrap the Open Telemetry SDK when they start, so they can send logs, metrics, and traces to Environment Watch. If the bootstrapping fails, the component keeps working but sends no telemetry through the Open Telemetry SDK. In some cases, such as a dependency conflict in an ASP.NET MVC custom page, the page itself can fail to load.

## Find the Bootstrapping Error

1. Open Kibana and go to **Discover**.
2. Select the `logs-*` data view and set the time range to when the problem started.
3. Run the following query:
    ```
    message : "Failed to build the Open Telemetry providers"
    ```

    Expected result:
    No results. Each result is a bootstrapping failure.
4. Expand a result and review the application name and the exception to identify the component. For example:
    ```
    Could not load file or assembly 'System.Diagnostics.DiagnosticSource, Version=10.0.0.0, Culture=neutral, PublicKeyToken=cc7b13ffcd2ddd51' or one of its dependencies.
    ```

## Skip the Open Telemetry SDK Bootstrapping for the Failing Component

Use the `DefaultBuildExclusions` instance setting to stop the failing component from bootstrapping the Open Telemetry SDK. Other components are not affected.

1. In Relativity, go to **Instance Settings** and create a new instance setting with these values. If the setting already exists, edit its value instead.

    | Field | Value |
    |---|---|
    | Name | `DefaultBuildExclusions` |
    | Section | `Relativity.OpenTelemetry` |
    | Machine | Leave blank |
    | Value Type | `Text` |
    | Value | The component to exclude (see the table below) |

    | Failing component | Value |
    |---|---|
    | Custom pages | `CustomPage` |
    | Event handlers | `EventHandler` |
    | One application only | The application GUID |
    | More than one | Separate entries with a semicolon, for example `CustomPage;EventHandler` |

    Alternatively, create the setting with SQL on the EDDS database:
    ```sql
    EXEC [eddsdbo].CreateInstanceSetting
        @section = 'Relativity.OpenTelemetry',
        @name = 'DefaultBuildExclusions',
        @machineName = '',
        @value = 'CustomPage',
        @description = 'A semicolon-separated list of components that skip the default Open Telemetry SDK bootstrapping.',
        @valueType = 'Text',
        @initialValue = '',
        @createAsSystemArtifact = 0
    ```

2. Restart the component so it reads the new value:
    - Custom pages, web applications, and event handlers that run in IIS: run `iisreset` from an elevated PowerShell window on each web server.
    - Event handlers that run in agents or Kepler services: restart the `kCura EDDS Agent Manager` or `kCura Service Host Manager` Windows service on the affected servers.

3. Open the component again, then run the query from [Find the Bootstrapping Error](#find-the-bootstrapping-error) with the time range starting after the restart.

    Expected result:
    No new results for the excluded component. Other components continue to log "Successfully created the Open Telemetry providers".

> [!IMPORTANT]
> An excluded component sends no telemetry to Environment Watch, so its dashboards and alerts stop receiving data. Once the underlying issue is fixed, clear the `DefaultBuildExclusions` value and restart the component again.
