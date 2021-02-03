---
layout: post
title: Azure Service Bus - Clients
---
The [Azure Service Bus](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-messaging-overview) is a cloud message-based platform that offers you asynchronous communication between several applications by messages. The Azure Service Bus provides you several clients, where the functionality depends on what you need to:

# SessionClient
The [client](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.servicebus.sessionclient?view=azure-dotnet) consumes messages from the sessions*. The sessions guarantee to consume messages in good sequence, but with concurrent calls, this is not guaranteed.


## Sample
```cs
var options = new MessageHandlerOptions((exceptionReceivedEvent) => throw exceptionReceivedEvent.Exception)
{
    MaxConcurrentCalls = 100,
    AutoComplete = true
};

var sessionClient = new SessionClient("connectionString", $"topicName/subscriptions/subscriptionName");
var messageSession = await sessionClient.AcceptMessageSessionAsync("sessionName"); //You can accept a message from any session or set the session name

messageSession.RegisterMessageHandler(Handle, options);

async Task Handle(Message message, CancellationToken cancellationToken)
{
    //Processing message
}
```

*Your topic subscription has enabled sessions, and a message needs to contain SessionId property.


# QueueClient
The [client](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.servicebus.queueclient?view=azure-dotnet) also consumes messages from the sessions, but you can set up concurrency sessions - it means you can consume messages from more sessions concurrently.

**Potential issue:** When the number of active sessions and concurrent sessions limit are the equals, and someone creates a new session, the handler cannot process messages from the new session because you hit the concurrent sessions limit.

## Sample
```cs
var options = new SessionHandlerOptions((exceptionReceivedEvent) => throw exceptionReceivedEvent.Exception)
{
    MaxConcurrentSessions = 100,
    AutoComplete = true
};

var queueClient = new QueueClient("connectionString", $"topicName/subscriptions/subscriptionName");
queueClient.RegisterSessionHandler(Handle, options);

async Task Handle(IMessageSession messageSession, Message message, CancellationToken cancellationToken)
{
    //Processing message
}
```

# SubscriptionClient
The [client](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.servicebus.subscriptionclient?view=azure-dotnet) consumes messages from the topics. There is an option to set up consuming more messages concurrently.

## Sample
```cs
var options = new MessageHandlerOptions((exceptionReceivedEvent) => throw exceptionReceivedEvent.Exception)
{
    MaxConcurrentCalls = 100,
    AutoComplete = true
};

var subscriptionClient = new SubscriptionClient("connectionString", "topicName", "subscriptionName");
subscriptionClient.RegisterMessageHandler(Handle, options);

async Task Handle(Message message, CancellationToken cancellationToken)
{
    //Processing message
}
```
