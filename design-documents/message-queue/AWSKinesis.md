#### 4- AWS Kinesis 

AWS Kinesis is a streaming based distributed messaging technology; it uses publish/subscribe mechanism for loose coupling between senders and receivers. It is designed for extremely high throughput for realtime applications. 

##### High Level Architecture 

Streaming and the concept of **Stream** itself is the central idea behind Kinesis. It is pretty similar to Apache Kafka with some differences. It is also suitable for implementing Event Sourcing and CQRS pattern, which is commonly used in Microservices Architecture because of the out-of-the-box support for high throughput messaging and publish/subscribe mechanism.

![product-page-diagram_Amazon-Kinesis-Data-Streams](/Users/jawed/sources/magento/architecture/design-documents/message-queue/AWSKinesisArchitecture.png)

##### Evaluation Table - Details

| Method        | Evaluation | Implementation Readiness                                     |
| ------------- | ---------- | ------------------------------------------------------------ |
| dequeue()     | Possiblity | getRecords() - ShardIterator needs to be managed behind the scenes. Stream can be Queue name. It can use getShardIterator(), before the call. |
| acknowledge() | Possiblity | Need to maintain the ShardIterator & SequenceNumber associated with it, using getShardIterator(). We can save NextShardIterator from getRecords() to acknowledge the message. |
| subscribe()   | Workaround | Workaround - batch reads with long polling can be implemented, example [Long Polling Subscribe Mechanism in AWS Kinesis](https://github.com/kaliop-uk/kueueingbundle-kinesis/blob/master/Adapter/Kinesis/Consumer.php) |
| reject()      | Possiblity | If we don't move the ShardIterator to NextShardIterator, we are pretty much staying on the same message. |
| push()        | Possiblity | Since you will have to provide stream, data & partition; we need to have some strategy to for partition selection; and need to maintain these values for Consumers. |



<img src="/Users/jawed/sources/magento/architecture/design-documents/message-queue/legend_img.png" alt="Legend" width="70%" height="70%" />

