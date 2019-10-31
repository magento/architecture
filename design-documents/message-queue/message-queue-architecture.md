# Messaging Architecture and Options

# Contents

[Overview and Introduction](#Overview-and-Introduction)

[Message Queue Processing Design](MagentoMessageQueueDesign.md#message-queue-processing-design)

[Consumers Runner Process](MagentoMessageQueueDesign.md#consumers-runner-process)

[Queue Interface](MagentoMessageQueueDesign.md#queue-interface)

[Evaluation of Technologies](EvaluationTechnologies.md#evaluation-of-technologies)

[Programming Language Support](EvaluationTechnologies.md#Programming-Language-Support)

[PHP Queue Abstraction - Libraries](EvaluationTechnologies.md#PHP-Queue-Abstraction---Libraries)

[Use Case Summary of Technologies](EvaluationTechnologies.md#Use-Case-Summary-of-Technologies)

[(1) AWS EventBridge](AWSEventBridge.md) 

[(2) AWS MQ](AWSMQ.md)

[(3) AWS SQS](AWSSQS.md)

[(4) AWS Kinesis](AWSKinesis.md)

[(5) Apache Kafka](ApacheKafka.md)

[(6) Azure Service Bus](AzureServiceBus.md)

[(7) PHP Enqueue Library](PHPEnqueueLibrary.md)

[(8) Adobe I/O](AdobeIO.md#8--Adobe-IO)



## Overview and Introduction

Magento uses message queue architecture for all asynchronous communication, where message sender and receiver are loosely coupled and doesn't talk to each other directly. For more information go through the following document 

[Magento Message Queue Overview](https://devdocs.magento.com/guides/v2.3/extension-dev-guide/message-queues/message-queues.html)

This document is looking into the current Queue Interface, its implementations, important interconnected modules like Queue Consumers & ConsumersRunner process; then finally looking into the different potential candidates for alternative Queueing technologies that can be used within Magento; which can serve as an alternative choice for the Magento customers in addition to currently supported technologies; the Queue technology should also support modern autonomous architecture for their communication needs for instance Event Sourcing and CQRS mechanism.
