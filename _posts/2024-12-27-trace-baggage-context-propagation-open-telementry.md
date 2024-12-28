---
layout: single
title:  "Trace & Baggage Context Propagation in .NET with OpenTelemetry"
---

Context Propagation is an essential concept of distributed tracing. As the name suggests, it propagates the `trace` and `baggage` context over a network to other services.


[Trace Context](https://www.w3.org/TR/trace-context/) and [Baggage Context](https://www.w3.org/TR/baggage/) are defined by the W3C standard. The propagation is by HTTP headers.

## TraceContext

The `traceparent` header represents incomming information about previous trace context.

`traceparent: 00-0af7651916cd43dd8448eb211c80319c-b7ad6b7169203331-01`

Format
* version
* trace-id
* parent-id
* trace-flags

## BaggageContext

The `baggage` header represents incomming baggage context. Its `key=value` representation. 

`baggage: key1=value1`

The information is transferred in plaintext. Be aware of personally sensitive information.

# .NET Usage

Tracing configuration registers `TraceContextPropagator` and `BaggageContextPropagator` out of the box. 


 ```csharp 
builder.Services.AddOpenTelemetry()
    .UseOtlpExporter()
    .WithTracing(tracing =>
        {
            tracing.AddSource(builder.Environment.ApplicationName);
        });
 ```

You can use your own propagators (zipking,...)
 ```csharp 
Sdk.SetDefaultTextMapPropagator(new CompositeTextMapPropagator(
    new List<TextMapPropagator>() {
        new MyCustomPropagator()
    }));
 ```

To enable propagation (trace & baggege) over HTTP register 
`AddAspNetCoreInstrumentation` and 
`AddHttpClientInstrumentation` to do it by default.
 ```csharp 
builder.Services.AddOpenTelemetry()
    .UseOtlpExporter()
    .WithTracing(tracing =>
        {
            tracing
            .AddSource(builder.Environment.ApplicationName)
            .AddAspNetCoreInstrumentation()
            .AddHttpClientInstrumentation()
        });


 ```

 If you are interested how the HTTP Propagation works - [Incomming HTTP](https://github.com/open-telemetry/opentelemetry-dotnet-contrib/blob/main/src/OpenTelemetry.Instrumentation.AspNetCore/Implementation/HttpInListener.cs#L109), [Outgoing HTTP](https://github.com/open-telemetry/opentelemetry-dotnet-contrib/blob/main/src/OpenTelemetry.Instrumentation.Http/Implementation/HttpHandlerDiagnosticListener.cs#L94).


 
Propagate contexts over other protocols - messaging, database, etc. You need to do it manually with the `Inject` and then `Extract` methods through transfer attributes by `Propagators.DefaultTextMapPropagator` that provides registered propagators.
