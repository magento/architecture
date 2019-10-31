#### 8- Adobe I/O 

Adobe I/O is a serverless event driven platform that allows you to quickly deploy custom functions/code in the cloud without any server setup. These functions executes via HTTP requests or Adobe I/O Events. These Events can be orchestrated with Sequences & Compositions. It is built on top of Apache OpenWhisk framework. 

Events are triggered by Event Providers within Adobe Services, for instance Creative Cloud Assets, Adobe Experience Manager & Adobe Analytics. To start listening to events for your application, you need to register a Webhook (URL endpoint) specifying which Event Types from which Event Providers it wants to receive; Adobe pushes events to your webhook via HTTP POST messages.

*"Magento SaaS based next generation platform can push it's Events on Adobe I/O Events like other Adobe Services, to be consumed by Developers through Adobe I/O Runtime for custom functionality and integrations. But it is not a right candidate for Magento Event Bus"*

[Adobe I/O Runtime Docs](https://www.adobe.io/apis/experienceplatform/runtime/docs.html)

[Apache OpenWhisk](https://openwhisk.apache.org/)

[Adobe I/O Events](https://www.adobe.io/apis/experienceplatform/events.html)

##### Example Architecture

Here is a nice example of Slack integration with Adobe Experience Manager (AEM) for asset change notification,

<img src="https://miro.medium.com/max/1920/1*ajkz4p7Q8Dc0BbrcOo6TpQ.jpeg" alt="Adobe I/O Events" width="80%" height="80%" />

[For more details follow the link](https://medium.com/adobetech/monitoring-aem-asset-updates-with-adobe-i-o-events-9c2a8395880d)



##### Evaluation Table - Details

| Method        | Evaluation          | Implementation Readiness                                     |
| ------------- | ------------------- | ------------------------------------------------------------ |
| dequeue()     | Not Possible or N/A | There is not a concept of explicit fetching of event, rather you define a trigger/event and the actions associated with it. |
| acknowledge() | Not Possible or N/A | This concept is not used, the architecture is funadementally different |
| subscribe()   | Not Possible or N/A | A PHP callback function is not possible, although a custom webhook (http endpoint) can be configured to be triggered for a particular Event. |
| reject()      | Not Possible or N/A | This concept is not used, the architecture is funadementally different |
| push()        | Not Possible or N/A | Events are triggered by Adobe SaaS Services in the Adobe Cloud as discussed above. |

<img src="/Users/jawed/sources/magento/architecture/design-documents/message-queue/legend_img.png" alt="Legend" width="70%" height="70%" />

