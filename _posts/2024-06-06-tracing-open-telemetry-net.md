---
layout: single
title:  "Distributed Tracing in .NET with OpenTelemetry"
---

This article is part of a series about [OpenTelemetry](https://opentelemetry.io/) in .NET. OpenTelemetry establishes a de facto standard for application observability by providing standardized APIs, SDKs, and tools to collect signals - metrics, traces, and logs.

## OpenTelementry .NET Tracing API

The .NET runtime contains an `Activity` class, which is used for tracing purposes and represents a [Span](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/trace/api.md#span) in OpenTelemetry terminology. The `Activity` class is part of the `System.Diagnostics.DiagnosticSource` package. 

This allows applications to emit OpenTelemetry traces using the standard .NET Runtime.

To align more closely with OpenTelemetry terminology (like tracer, span instead of activitySource, activity) in the application, use the [OpenTelementry.API](https://www.nuget.org/packages/opentelemetry.api), which wraps the .NET `Activity` classes. 


## Manual Instrumentation

Use the `ActivitySource` class, which provides the name and version of the application doing the instrumentation. The instance of `ActivitySource` should be created once and reused throughout the application.


 ```csharp 
 static ActivitySource activitySource = new ActivitySource(
    "serviceName",
    "1.0.0");
 ```

Use the `ActivitySource` instance to create an `Activity` instance, which represents a single-span operation.

 ```csharp 
 using var activity = activitySource.StartActivity("ActivityName");
  ```

If there are no listeners/samplers interested in this activity, the `StartActivity` returns null.

The `StartActivity` method starts an activity  and sets it as the current activity in the `ActivityContext.Current` and propagates the `ActivityContext` from the parent activity.


 ```csharp
using var activity = activitySource.StartActivity("ActivityName");
activity?.SetTag("http.url", "url");
activity?.AddEvent(new ActivityEvent("activity event"));
activity?.SetStatus(ActivityStatusCode.Error, "error");
 ```

Populate activity with tags follow the [ OpenTelemetry semantic conventions](https://github.com/open-telemetry/semantic-conventions/blob/main/docs/general/trace.md).



## Configure OpenTelemetry

Install the nuget package `OpenTelemetry.Extensions.Hosting` to expose the OpenTelemetry SDK.

 ```csharp
builder.Services.AddOpenTelemetry()
    .WithTracing(tracing =>
    {
        tracing.SetResourceBuilder(ResourceBuilder.CreateDefault()
                    .AddService("serviceName"))
                .AddSource("serviceName"); // Configure sources to listen
    });
 ```
### Instrumentations
There are two ways how to instrument. 
* Manual instrumentation means adding custom instruments to .NET application using the `Activity` class. 
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
