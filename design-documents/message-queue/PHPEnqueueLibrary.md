#### 7- PHP Enqueue Library 

PHP Enqueue provides JMS style abstraction layer over many brokers as discussed here

[PHP Queue Abstraction - Libraries](#PHP-Queue-Abstraction---Libraries)

[Quick Tour](https://github.com/php-enqueue/enqueue-dev/blob/master/docs/quick_tour.md)

[Extentions for additional functionality](https://github.com/php-enqueue/enqueue-dev/blob/master/docs/consumption/extensions.md)

##### PHP Enqueue Library Evaluation Table - Details

| Method        | Evaluation | Implementation Readiness                                     |
| ------------- | ---------- | ------------------------------------------------------------ |
| dequeue()     | Available  | Consumer > receive()                                         |
| acknowledge() | Available  | Consumer > acknowledge(message)<br />Callback function / processor can return ACK |
| subscribe()   | Available  | bindCallback(topic, callback function)                       |
| reject()      | Available  | Consumer > reject(message)<br />callback function / processor can return REJECT or REQUEUE |
| push()        | Available  | sendEvent(topic, message)                                    |

<img src="legend_img.png" alt="Legend" width="70%" height="70%" />

