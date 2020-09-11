# First Class Queue Configuration

This proposal intends to expose a first-class "queue" object in `queue_topology.xml` of a module and provides backward compatibility considerations for prior versions of Magento before this change.

## Overview

The Message Broker, RabbitMQ, supports `arguments` that are defined at queue creation time that dictate certain behaviors of the resulting queue. Currently (as of v2.4.0), Magento infers what queues will be created from properties of the `exchange.binding` defined in any given module's `queue_topology.xml`.

Unfortunately, the current implementation of `queue_topology.xml` does not expose a queue object to specify arguments at queue creation time, preventing utilization of the configurable behavior of RabbitMQ queues.

## Use Cases

- As a developer, I would like to configure my queues with additional arguments.
- As a developer, I would like to achieve High Availability with my queueing system.

### The Scenario

In order to support High Availability message processing, we would like to utilize RabbitMQ's [`x-dead-letter-exchange`](https://www.rabbitmq.com/dlx.html) and [`x-message-ttl`](https://www.rabbitmq.com/ttl.html) arguments for queues. Achieving High Availability with these two arguments would require two queues:

1. A primary queue that processes messages.
2. A secondary "retry" queue with a defined `x-message-ttl` and `x-dead-letter-exchange` arguments.

The first queue's consumers attempt to process its messages, and then upon failure conditions, publish "retryable" messages (via an exchange) into the "retry" queue.

After the retry queue's TTL passes, the now "expired" message is then moved (**without being consumed**) as a dead-letter back to the primary queue (via the x-dead-letter-exchange).

In the end, this results in messages being retryable in whatever mechanism a developer wishes, based upon whatever failure conditions a developer desires.

## Design

An additional object would need to be added to the `queue_topology.xml` API, called a `queue`. A `queue` supports the following properties:

```txt
name
connection
durable
autoDelete
arguments
```

These arguments are consistent with the existing arguments for exchanges and bindings and should feel very familiar to developers accustomed to the current syntax.

### Matching Behavior

Since there is pre-existing behavior defined for queue creation from `exchange.binding` there must be a matching mechanism that determines which mechanism of queue-creation wins.

A queue is considered a "match" (and therefore overrules the prior `exchange.binding` behavior) when the first-class queues `name` and `connection` exactly match (string comparison, case-sensitive) a queue which would be created by `exchange.binding`. That is to say, the bindings -> queue generator mechanism should not attempt to create queues where the `queue` with the same name and connection exists.

### Sample Configurations

- [A Simple Configuration](#a-simple-configuration)
- [First-Class Queue Priority](#first-class-queue-priority)
- [Queue-Connection Mismatch](#queue-connection-mismatch)

#### A Simple Configuration

This configuration would result in a single queue: `some-queue` which is configured by the queue object.

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework-message-queue:etc/topology.xsd">
    <queue name="some-queue" connection="amqp">
        <arguments>
           <argument name="x-message-ttl" xsi:type="string">arg-value</argument>
           <argument name="x-dead-letter-exchange" xsi:type="string">my-dlx</argument>
       </arguments>
    </queue>
</config>
```

#### First-Class Queue Priority

When a `queue` and an existing `exchange.binding` collide via the described matching mechanism, a single queue results: `some-queue` as described in the below case.

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

#### Queue-Connection Mismatch

If a `queue` is defined with a connection that is different from the `exchange` that would bind to it via the old queue-creation mechanism, two queues are created, one in each connection. We should likely warn in this scenario, as likely this is an unintentional behavior. A suggested message (appearing during `setup:upgrade`) is:

> `some-queue` is defined in two connections: `amqp` and `db`. If this intentional, this message can be ignored.

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework-message-queue:etc/topology.xsd">
    <exchange name="inbox.mdc" type="topic" connection="db">
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

## Backwards Compatability

Backward compatibility is well-defined and should be as follows:

### No Queue Defined

If no `queue` is defined, the behavior is as before, a single queue is created in the defined connection.

```xml
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework-message-queue:etc/topology.xsd">
    <exchange name="inbox.mdc" type="topic" connection="amqp">
        <binding id="someBinding"
                 topic="some.topic.name"
                 destinationType="queue"
                 destination="some-queue"/>
    </exchange>
</config>
```

## Open Questions

1. What about `autoDelete` differences?
       I suspect that connection/name is a sufficient matching pair, so this case may be irrelevant.
2. What about `durable` differences?
       I suspect that connection/name is a sufficient matching pair, so this case may be irrelevant.
