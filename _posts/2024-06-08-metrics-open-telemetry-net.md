---
layout: single
title:  "Metrics in .NET with OpenTelemetry"
---

In the previous article, I wrote about [Dsitributed Tracing with OpenTelementry](https://tomasjurasek.github.io/2024/06/06/tracing-open-telemetry-net/). 
Now, let's write about [Metrics](https://opentelemetry.io/docs/specs/otel/metrics/).

## OpenTelementry .NET Metrics API

The .NET runtime contains most of [Metric API](https://opentelemetry.io/docs/specs/otel/metrics/api/), that allows applications to emit OpenTelemetry metrics using the standard .NET runtime.

## Metrics

Use the `Meter` class, which is responsible for creating metric instruments to create manual metrics from the `System.Diagnostics.DiagnosticSource` nuget package.



 ```csharp 
 static readonly Meter meter = new("serviceName", "1.0");
  ```

The default, there is no listener to meters; they need to be explicit configured into `MeterProvider.` The `MeterProvider` is the entry point to the configuration of all `Meter`'s and their instrumentation.

> It's not recommended to initialize the instrument for every metric execution. Create a (static) wrapper over metrics.

#### Initialize
Initialize some kinds of instruments - `Counter`, `Gauge`, `Histogram`, ...

 ```csharp 
public static class ApplicationMetrics
{
    private static readonly Meter meter = new Meter("applicationName", "1.0.0");
    private static readonly Counter<int> applicationCounter = meter.CreateCounter<int>("applicationCounter");
    public static void IncreateApplicationCounter() => applicationCounter.Add(1);
}
  ```
#### Usage
```csharp 
public void SomeMethod()
{
    // Code
    ApplicationMetrics.IncreateApplicationCounter();
    // Code
}
```

### Aggregation
It provides a way to aggregate large number of measurements. More detailed description [here](https://opentelemetry.io/docs/specs/otel/metrics/sdk/#aggregation).

### Views
It provides a way to customize the metrics output - which metric instruments are to be processed, which attributes report. More detailed description [here](https://opentelemetry.io/docs/specs/otel/metrics/sdk/#view-examples).
## Configure OpenTelemetry

As tracing, also metrics are part of the `OpenTelemetry.Extensions.Hosting` nuget package.

 ```csharp
builder.Services.AddOpenTelemetry()
    .WithMetrics(metrics =>
    {
        metrics.SetResourceBuilder(ResourceBuilder.CreateDefault()
                    .AddService("serviceName"))
                .AddMeter("serviceName"); // Configure meter to listen
    });
 ```

### Instrumentations

Metrics instruments must be registered by `AddMeter("meterName")` to `MeterProvider`. There exist some build-in `Meter`'s to instrument ASP.NET application
 `Microsoft.AspNetCore.Hosting`, `Microsoft.AspNetCore.Server.Kestrel`, `System.Net.Http`.

 To instrument the application's runtime (ram, cpu, gc,...) install `OpenTelemetry.Instrumentation.Runtime` and configure it by `AddRuntimeInstrumentation`.

  ```csharp
builder.Services.AddOpenTelemetry()
    .WithMetrics(metrics =>
    {
        metrics.SetResourceBuilder(ResourceBuilder.CreateDefault()
                    .AddService("serviceName"))
                .AddMeter("serviceName"); // Configure meter to listen
                .AddMeter("Microsoft.AspNetCore.Hosting", "Microsoft.AspNetCore.Server.Kestrel", "System.Net.Http");
                .AddRuntimeInstrumentation()
    });
 ```

### Exporters

There are several ways to export metrics to backends - OTLP, Console, and Prometheus in the namespace of the `OpenTelemetry.Exporter.NAME` nuget package.

To enable OTLP exporter for all signals, there is exists an extension `UseOtlpExporter`.

 ```csharp
builder.Services.AddOpenTelemetry()
    .UseOtlpExporter();
 ```