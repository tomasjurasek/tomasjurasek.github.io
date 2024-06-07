---
layout: single
title:  "Distributed Tracing in .NET with OpenTelemetry"
---

[OpenTelemetry](https://opentelemetry.io/) brings a de-facto standard for application observability with standardized APIs, SDKs, and tools to collect observability signals - metrics, traces, and logs.

## OpenTelementry .NET Tracing API

.NET runtime contains an `Activity` class, which is used for tracing purposes and represents a [Span](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/trace/api.md#span) in the OpenTelemetry terminology. The `Activity` class is part of `System.Diagnostics.DiagnosticSource` package. 

It means, you can instrument applications to emit OpenTelemetry traces using standard .NET Runtime.

If you want to use OpenTelemetry terminology (like tracer, span instead of activitySource, activity) also in your application, you can use [OpenTelementry.API](https://www.nuget.org/packages/opentelemetry.api) that is a wrapper around the .NET `Activity` classes. 


## Instrumenting .NET Application

Use the `ActivitySource` class, which provides the name and version of the application doing the instrumentation. The instance of `ActivitySource` should be created once and reused throughout the application.


`static ActivitySource activitySource = new ActivitySource(
    "MyApplicationName",
    "1.0.0");
`

Then, use the `ActivitySource` instance to create an `Activity` instance, which represents a single-span operation.

`using var activity = activitySource.StartActivity("ActivityName");`

If there are no listeners/samplers interested in this activity, the `StartActivity` returns null.

`StartActivity` start an activity and automatically becomes the current activity in the `ActivityContext.Current` and propagate the `ActivityContext` from the parent activity.


 ```csharp
using var activity = activitySource.StartActivity("ActivityName");
activity?.SetTag("http.url", "url");
activity?.AddEvent(new ActivityEvent("activity event"));
activity?.SetStatus(ActivityStatusCode.Error, "error");
 ```

Populate activity with tags follow the [ OpenTelemetry semantic conventions](https://github.com/open-telemetry/semantic-conventions/blob/main/docs/general/trace.md).



## Configure OpenTelemetry

Firstly install the nuget package `OpenTelemetry.Extensions.Hosting` to expose the OpenTelemetry SDK.

Use OpenTelemetry SDK to setup a basic collection of traces.

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
* Manually instrumentation means adding custom instruments to .NET application using the `Activity` class. 
* [Automatic instrumentation](https://opentelemetry.io/docs/zero-code/net/instrumentations/) adds external instruments (HTTP, SQL, Elastic, Redis,...) to .NET application without modify a source code by adding `OpenTelemetry.AutoInstrumentation` nuget package.

Automatic instrumentation can also enabled by `OpenTelemetry.Instrumentation.X` nuget packages that are specific per resource.

For example, instruments for HTTP Request (In) and HttpClient Request (Out) are in the `OpenTelemetry.Instrumentation.AspNetCore` and `OpenTelemetry.Instrumentation.Http` packages, which also configures and propagates trace context through [Propagator API](https://opentelemetry.io/docs/specs/otel/context/api-propagators/) via HTTP Headers.  

 ```csharp
builder.Services.AddOpenTelemetry()
    .WithTracing(tracing =>
    {
        tracing.SetResourceBuilder(ResourceBuilder.CreateDefault()
                    .AddService("serviceName"))
                .AddSource("serviceName"); // Configures sources to listen (instrument)
                .AddAspNetCoreInstrumentation() // HTTP Request (In) - Configures tracing context from headers
                .AddHttpClientInstrumentation() // HTTP Request (Out) - Propagates tracing context to headers
    });
 ```

[Sampling](https://opentelemetry.io/docs/concepts/sampling/) (especially  Head Sampling) can be configured by `tracing.SetSampler(...)`.

### Exporters

An application can export instrumentations by exporters. Many exporters implementation exist such as Console, Zipkin, OTEL.

For example, export traces to the OTEL collector install `OpenTelemetry.Exporter.OpenTelemetryProtocol` and configure it using [OTLP Exporter Configuration](https://opentelemetry.io/docs/languages/sdk-configuration/otlp-exporter/).

 ```csharp
builder.Services.AddOpenTelemetry()
    .WithTracing(tracing =>
    {
        tracing.SetResourceBuilder(ResourceBuilder.CreateDefault()
                    .AddService("serviceName"))
                .AddSource("serviceName"); // Configure sources to listen (instrument)
                .AddAspNetCoreInstrumentation() // HTTP Request (In)
                .AddHttpClientInstrumentation() // HTTP Request (Out)
                .AddOtlpExporter()
    });
 ```
