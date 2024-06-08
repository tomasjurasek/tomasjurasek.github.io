---
layout: single
title:  "Metrics in .NET with OpenTelemetry"
---

[OpenTelemetry](https://opentelemetry.io/) establishes a de facto standard for application observability by providing standardized APIs, SDKs, and tools to collect signals - metrics, traces, and logs.

## OpenTelementry .NET Metrics API

The .NET runtime contains a most of [Metric API](...), that allows applications to emit OpenTelemetry metrics using the standard .NET Runtime.

## Metrics

Use the `Meter` class, which is responsibles to create metric instruments (reports to measurements) to create manually metrics from the `System.Diagnostics.DiagnosticSource` nuget package.

`static readonly Meter MyMeter = new("serviceName", "1.0");
`

The default there is no listener to meters, they need to be explicid configured into `MeterProvider`. The `MeterProvider` is entry point to configuration of all `Meter`'s and their instrumentatons.






## Configure OpenTelemetry

Install the nuget package `OpenTelemetry.Extensions.Hosting` to expose the OpenTelemetry SDK.

 ```csharp
builder.Services.AddOpenTelemetry()
    .WithMetrics(tracing =>
    {
        tracing.SetResourceBuilder(ResourceBuilder.CreateDefault()
                    .AddService("serviceName"))
                .AddMeter("serviceName"); // Configure sources to listen
    });
 ```
### Instrumentations
There are two ways how to instrument. 
* Manually instrumentation means adding custom instruments to .NET application using the `Activity` class. 
* [Automatic instrumentation](https://opentelemetry.io/docs/zero-code/net/instrumentations/) adds external instruments (HTTP, SQL, Elastic, Redis,...) to .NET application without modify a source code by adding `OpenTelemetry.AutoInstrumentation` nuget package.

Automatic instrumentation can also enabled by `OpenTelemetry.Instrumentation.X` nuget packages that are specific per resource.

For example, instruments for HTTP Requests (In) and Requests (Out) are in the `OpenTelemetry.Instrumentation.AspNetCore` and `OpenTelemetry.Instrumentation.Http` packages, which also configures and propagates trace context through [Propagator API](https://opentelemetry.io/docs/specs/otel/context/api-propagators/) via HTTP Headers.  

 ```csharp
builder.Services.AddOpenTelemetry()
    .WithTracing(tracing =>
    {
        tracing.SetResourceBuilder(ResourceBuilder.CreateDefault()
                    .AddService("serviceName"))
                .AddSource("serviceName"); // Configures sources to listen (instrument)
                .AddAspNetCoreInstrumentation() // HTTP Requests (In) - Configures tracing context from headers
                .AddHttpClientInstrumentation() // HTTP Requests (Out) - Propagates tracing context to headers
    });
 ```

[Sampling](https://opentelemetry.io/docs/concepts/sampling/) (especially  Head Sampling) can be configured by `tracing.SetSampler(...)`.

### Exporters

Exporters allow an application to send instrumentation data to various backends, such as Console, Zipkin, or OTEL.

For example, export traces to the OTEL collector by `OpenTelemetry.Exporter.OpenTelemetryProtocol` and configure it using [OTLP Exporter Configuration](https://opentelemetry.io/docs/languages/sdk-configuration/otlp-exporter/).

 ```csharp
builder.Services.AddOpenTelemetry()
    .WithTracing(tracing =>
    {
        tracing.SetResourceBuilder(ResourceBuilder.CreateDefault()
                    .AddService("serviceName"))
                .AddSource("serviceName"); // Configure sources to listen (instrument)
                .AddAspNetCoreInstrumentation() // HTTP Requests (In)
                .AddHttpClientInstrumentation() // HTTP Requests (Out)
                .AddOtlpExporter()
    });
 ```
