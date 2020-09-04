# First Class Queue Configuration

This proposal exposes a first class "queue" object in `queue_topology.xml` and provides backwards compatability for prior versions of Magento before this change.

## Overview and Introduction
The Message Broker, RabbitMQ supports arguments that are defined at Queue creation time which dictate certain behaviors of the queue in question. Currently (as of v2.4.0), Magento infers what queues will be created from properties of the `exchange.bindings` defined in any given module's `queue_toplogy.xml`. 

Unfortunately, the current implementation of `queue_topology.xml` doesn't expose a queue object to specify arguments at queue creation time, preventing the configurable behavior of RabbitMQ.

## Use Cases
* As a developer, I would like to configure my queues with additional arguments.
* As a developer, I would like to achieve High Availability with my message Queueing Behavior.

### The Scenario
In order to support High Availability message processing, we would like to utilize RabbitMQ's [`x-dead-letter-exchange`](https://www.rabbitmq.com/dlx.html) and [`x-message-ttl`](https://www.rabbitmq.com/ttl.html) argument for queues. This would require two queues:

1. A main queue which processes messages.
2. A "retry" queue with a defined `x-message-ttl` and a `x-dead-letter-exchange`. 

The first queue's consumers attempt to process its messages, and then upon failure conditions, publish these messsages (via an exchange) into the "retry" queue. 

After the retry queues TTL passes, these messages are then expire (**without being consumed**) and move the messages as a dead-letter to the origin queue.

In the end, this results in messages being retryable in whatever mechanism a developer wishes, based upon whatever failure conditions a developer desires.

## How it would work 
An additional object would need to be added to the `queue_topology.xml` API, called a `queue`. This `queue` supports the following properties:

```txt
name
connection
durable
autoDelete
arguments
```

These arguments are consistent with the existing arguments for exchanges and bindings, and should feel very familiar to developers accustomed to the current syntax.

### A Sample Configuration
```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework-message-queue:etc/topology.xsd">
    <exchange name="inbox.mdc" type="topic" connection="amqp">
        <binding id="someBinding"
                 topic="some.topic.name"
                 destinationType="queue"
                 destination="some-queue"/>
    </exchange>
    <queue name="some-queue" connection="amqp">
        <arguments>
           <argument name="x-message-ttl" xsi:type="string">arg-value</argument>
           <argument name="x-dead-letter-exchange" xsi:type="string">my-dlx</argument>
       </arguments>
    </queue>
</config>
```

This configuration would result in a single queue: `some-queue` which is configured by the queue object.

## Backwards Compatability
Backwards compatability is achieved via the following mechanism:

1. If no `queue`s are defined, behavior is as before.
2. If a `queue` is defined with a connection that is different from the `exchange` that would bind to it, two queues are created, one in each connection.
    * We should warn people about this.
3. The older bindings -> queue generator mechansim should not attempt to create queues where the `queue` with the same name and connection exists.

## Open Questions

1. What about `autoDelete` differences?
    I suspect that connection/name is a sufficient matching pair, so this case may be irrelevant.  
2. What about `durable` differences?
    I suspect that connection/name is a sufficient matching pair, so this case may be irrelevant.