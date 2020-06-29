---
layout: post
title: Service Bus - wrong order received messages
---


We had a problem with one of our services that received and processed messages in the wrong order from the service bus topic. The sending part of the system sends the message one by one in good order but on the receiving part, the message was in the wrong order.

You can do a lot of mistakes in a Service Bus - Queue/Topics set up.

* [Partitioning](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-partitioning#how-it-works) - when you enable this option you need to put a *PartitionKey* into your message that you send. If you don't do this, the internal algorithm creates some *PartitionKey* and store the message to the partition by the *PartitionKey*. Then, when you want to receive the messages Service Bus queries all partitions for messages and then returns the first message that is obtained from any of the messaging stores to the receiver. **That was the problem we had.**

* [Duplicate detection](https://docs.microsoft.com/en-us/azure/service-bus-messaging/duplicate-detection) - when you enable this option you have a 'sliding window' where the messages are detected from duplication by the *MessageId* property. Also when you have enabled Partitioning and you don't have a *PartitionKey* property, the *MessageId* property serves as *PartitionKey*.

Property *MaxConcurrencyCalls* is also tricky. This property is set up on the [MessageHandlerOptions](https://docs.microsoft.com/en-us/dotnet/api/microsoft.azure.servicebus.messagehandleroptions?view=azure-dotnet) when you register your *handlers/receivers* in code. The *MaxConcurrencyCalls* will start more threads to process more parallel messages.


