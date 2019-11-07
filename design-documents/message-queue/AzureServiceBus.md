#### 6- Azure Service Bus

Microsoft Azure Service Bus is a fully managed enterprise integration message broker. It support familiar concepts like Queues, Topics, Rules/Filters and much more.

##### High Level Architecture



<img src="AzureServiceBusQueue.png" alt="Legend" width="70%" height="70%" />



<img src="AzureServiceBusTopic.png" alt="Legend" width="70%" height="70%" />



##### Evaluation Table - Details

Azure Service Bus supports AMQP 1.0,  and couple of languages, PHP support is again limited for the protocol

[AMQP Azure Service Bus Overview](https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-amqp-overview)

[azure-uaqmp-c PHP Bindings for AMQP 1.0](https://github.com/norzechowicz/php-uamqp)

[Exported PHP Modules from C Native Library](https://github.com/norzechowicz/php-uamqp/tree/master/ext/src/php)

| Method        | Evaluation  | Implementation Readiness                                     |
| ------------- | ----------- | ------------------------------------------------------------ |
| dequeue()     | Available   | receive()                                                    |
| acknowledge() | Available   | accept() / release()                                         |
| subscribe()   | *Workaround | Long Polling might need to be implemented, unless we find a good library that supports AMQP 1.0 for PHP; Java has full support for required features. |
| reject()      | Available   | reject(errorCondition, errorDescription)                     |
| push()        | Available   | sendMessage(message, destination)                            |

<img src="legend_img.png" alt="Legend" width="70%" height="70%" />

