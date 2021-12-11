#### 5- Apache Kafka

Apache Kafka is a popular open-source stream-processing / messaging platform; its by design distributed, replicated & resilient (or fault tolerent) which can acheive very high throughput.   

##### High Level Architecture

Topic and Publish / Subscribe mechanism is at the core of Kafka. Effective for implementing Event Sourcing and CQRS pattern, which is commonly used in Microservices Architecture. It is also used for variety of streaming use cases, which requires near real-time processing of records.

<img src="Apache_Kafka_HLD.png" alt="image-20190927212814114" width="60%" />



##### Evaluation Table - Details

Consumer is usually part of Consumer Group, it ensures that each group receives a copy of the message from the topic. Consumer needs to know its offset and partition, although parition can be automatically assigned when you begin consuming data from the topic, but you can also choose to manually assign parition, but these two cannot be mixed up. Consumer first needs to subscribe itself to the list of topics.

After you read the message(s), you can either configure auto-commit or allow manual commits for the offsets, in real-life use-cases you probably would want to control manually based on your strategy.

| Method        | Evaluation | Implementation Readiness                                     |
| ------------- | ---------- | ------------------------------------------------------------ |
| dequeue()     | Possiblity | Initiate **poll () or consume()**. If there are records available, the call will immediately returns, otherwise it will wait for specified timeout which can be passed as parameter. |
| acknowledge() | Possiblity | There are several ways to commit the offset, which indicates that a particular consumer has consumed those messages. The way you call commit API controls the delivery semantics. |
| subscribe()   | Possiblity | There are multiple ways in which the subscribtion mechanism can be implemented, the default Kafka subscribtion is telling Kafka which topics a consumer is interested in. But we can also subscribe a callback function; and we can use Kafka Stream API to receive messages in near realtime. |
| reject()      | Possiblity | If we don't auto-commit or manually commit the offset, then we are not moving the needle. |
| push()        | Possiblity | Since you will have to provide topic, data & partition; we need to have some strategy to for partition selection; and need to maintain these values for Consumers. |



<img src="legend_img.png" alt="Legend" width="70%" height="70%" />

