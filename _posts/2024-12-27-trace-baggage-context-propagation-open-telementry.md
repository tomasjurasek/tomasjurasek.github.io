---
layout: single
title:  "Trace & Baggage Context Propagation in .NET with OpenTelemetry"
---

Context propagation is a key part of distributed tracing. It enables transmitting both `trace` and `baggage` context across network boundaries so that different services in your system can participate in the same `trace`.


The W3C provides standards for propagation:

* [Trace Context](https://www.w3.org/TR/trace-context/) 
* [Baggage Context](https://www.w3.org/TR/baggage/)

These contexts are transferred in HTTP headers (or other protocols), typically as `traceparent` for `trace context` and `baggage` for `baggage context`. Below, we will explore how this works in .NET with OpenTelemetry.

## TraceContext

The `traceparent` header contains information about the current trace.

```csharp 
traceparent: 00-0af7651916cd43dd8448eb211c80319c-b7ad6b7169203331-01
```

**Format**
* Version (00)
* Trace-Id (0af7651916cd43dd8448eb211c80319c)
* Parent-Id (b7ad6b7169203331)
* Trace-Flags (01)

In .NET, a trace (or span) is represented by an Activity object. Each Activity instance corresponds to one span in the trace.

## BaggageContext

The `baggage` header holds additional context in the form of `key=value` pairs.
```csharp 
baggage: key1=value1
```


In .NET with OpenTelemetry, do not use `Activity.Baggage` to read or modify baggage. Instead, use `Baggage.Current` from the `OpenTelemetry.Api` package. 

This ensures that `baggage context` is managed consistently with the OpenTelemetry specifications.

> Tmportant: This header values are transferred in plaintext. Avoid placing sensitive or personally identifiable information in baggage context.

# .NET Usage

Tracing configuration registers `TraceContextPropagator` and `BaggageContextPropagator` out of the box.


 ```csharp 
builder.Services.AddOpenTelemetry()
    .UseOtlpExporter()
    .WithTracing(tracing =>
        {
            tracing
                .SetResourceBuilder(ResourceBuilder.CreateDefault()
                    .AddService(builder.Environment.ApplicationName));
        });
 ```

 These propagators enable extracting `traceparent` and `baggage` headers format from incoming requests and injecting them into outgoing requests.

## Custom Propagators
If you need support for alternative or custom propagators (e.g., Zipkin headers), you can set them explicitly using a CompositeTextMapPropagator

 ```csharp 
Sdk.SetDefaultTextMapPropagator(new CompositeTextMapPropagator(
    new List<TextMapPropagator>() {
        new MyCustomPropagator()
    }));
 ```

 
## HTTP Propagation

 By adding the following instrumentation, OpenTelemetry will automatically handle context propagation for incoming and outgoing HTTP requests.

 * **AddAspNetCoreInstrumentation** extracts traceparent and baggage from incoming HTTP requests.
* **AddHttpClientInstrumentation** injects traceparent and baggage into outgoing HTTP requests.


 ```csharp 
 builder.Services.AddOpenTelemetry()
    .UseOtlpExporter()
    .WithTracing(tracing =>
    {
        tracing
            .SetResourceBuilder(ResourceBuilder.CreateDefault()
                .AddService(builder.Environment.ApplicationName));
            .AddAspNetCoreInstrumentation()
            .AddHttpClientInstrumentation();
    });
 ```



If you want to see how this works under the hood, check out the OpenTelemetry .NET code for [incoming HTTP instrumentation](https://github.com/open-telemetry/opentelemetry-dotnet-contrib/blob/main/src/OpenTelemetry.Instrumentation.AspNetCore/Implementation/HttpInListener.cs#L109) and [outgoing HTTP instrumentation]((https://github.com/open-telemetry/opentelemetry-dotnet-contrib/blob/main/src/OpenTelemetry.Instrumentation.Http/Implementation/HttpHandlerDiagnosticListener.cs#L94)).

 
## Manual Propagation

For protocols not natively instrumented (messaging systems, outbox, etc), you'll need to manually propagate the context by calling the `Inject` and `Extract` methods on the `Propagators.DefaultTextMapPropagator`.


 ```csharp 
public void SendMessage(IMessage message)
{
    var propagationContext = new PropagationContext(Activity.Current?.Context ?? default, Baggage.Current);

    Propagators.DefaultTextMapPropagator.Inject(
        propagationContext,
        message.Headers,
        (headers, key, value) => headers[key] = value
    );

    // Publishing ...
}

public void ProcessMessage(IMessage message)
{
    var propagationContext = Propagators.DefaultTextMapPropagator.Extract(
        default,
        message.Headers,
        (headers, key) => headers.TryGetValue(key, out var value) ? new[] { value } : Array.Empty<string>()
    );
    
    using var activity = ActivityProvider.ActivitySource.StartActivity("receive", ActivityKind.Consumer, propagationContext.ActivityContext);

    Baggage.Current = propagationContext.Baggage;

    // Processing ...
  
}

 ```

 Using this approach, you maintain consistent `trace` and `baggage` context across any protocol or communication mechanism in your distributed system.