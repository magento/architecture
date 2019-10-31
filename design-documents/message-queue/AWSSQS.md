#### 3- AWS SQS 

AWS SQS is a distributed & fault tolerant Queuing Technology; it provides point to point connectivity. It can be used with SNS to add publish / subscribe mechanism as well. Single message gets replicated across different SQS Servers.

##### High Level Architecture 



![image-20190927145647952](/Users/jawed/sources/magento/architecture/design-documents/message-queue/AWS_SQSArchitecture1.png)



SQS uses Visibility Timeout to prevent other consumers to receive the same message, during which a consumer has to Delete the message explicitly afrer processing it or the message will be available for others for reuse.

![image-20190927150306008](/Users/jawed/sources/magento/architecture/design-documents/message-queue/AWS_SQSArchitecture2.png)



##### Evaluation Table - Details

| Method        | Evaluation | Implementation Readiness                                     |
| ------------- | ---------- | ------------------------------------------------------------ |
| dequeue()     | Available  | ReceiveMessage() - possiblity with many available options for instance long & short polling. |
| acknowledge() | Possiblity | DeleteMessage() for positive acknowledge, by default message locked for **Visibility Timeout** period for other consumers. |
| subscribe()   | Workaround | ReceiveMessage() - Using polling based mechanism, it can be implemented; but not exactly as true callback mechanism. |
| reject()      | Possiblity | The messages are auto visible again for consumption, if explicit DeleteMessage() is not called before timeout, as explained above. |
| push()        | Available  | SendMessage()                                                |



<img src="/Users/jawed/sources/magento/architecture/design-documents/message-queue/legend_img.png" alt="Legend" width="70%" height="70%" />

