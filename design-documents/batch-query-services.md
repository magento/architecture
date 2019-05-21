#Batch query services

## Purpose

The main reason for such change is performance.
The system triggers multiple queries during an operation (page load, service call). 
Instead of making several requests for data retrieval,
it would be more efficient to assemble all requests to the same service and send them as the single call to query service.
Even without internal code optimization such approach saves time that required for the service components bootstrap.
* Such an approach saves time that required for the service components bootstrap.
* The service implementation can be changed to guarantee better performance for batch operations.

## Arguments of operation

A service that manipulates with a single request could have a straightforward interface:
```php
    /**
     * @param int $productId
     * @param int $customerGroupId
     * @param int $websiteId
     * @return float
     */
    public function getProductPrice(int $productId, int $customerGroupId, int $websiteId) : float
```

If service has to work with multiple input arguments our interface will lose part of transparency.
```php
    /**
     * @param array $args
     * @return array
     */
    public function getProductPrice(array $args) : array
```

We can use object representation for input arguments to achieve better transparency and enforce strict types check.
```php
class ProductPriceSearchCriteria implements CriteriaInterface
{
    public function getProductId() : int;
    public function getCustomerGroupId() : int;
    public function getWebsiteId() : int;
}

```
Unfortunately, we cannot enforce consistency of arguments this check on a language level.
But at least input arguments types are transparent for the developers. 
```php
    /**
     * @param ProductPriceSearchCriteria[] $args
     * @return array
     */
    public function getProductPrice(array $args) : array
```





## Results output

Because we may receive multiple requests to one service 
it makes sense to return the results segregated per the request.


There are a few options to consider:

### 1. Returns execution results with original request

This decision allows us to reduce effort on the client to recognise part of the result that belongs to it.
As a consequence, the client should not keep state (criteria) between requests. 

```php
public class ResultContainer implements ContainerInterface
{
    public function getCriteria() : CrtiteriaInterface
    public function getResult() : array
}
```

Currently, the interface looks more clear.
```php
    /**
     * @param ProductPriceSearchCriteria[] $args
     * @return CrtiteriaInterface[]
     */
    public function getProductPrice(array $args) : array
```
For some cases, requested information that returned with the response may help to build a stateless scenario.
For instance, a widget which shows weather for the detected area and refreshes forecast every 5 minutes.

Cons service always has to return request arguments increasing the size of the message that we are transfering through the network

#### Payload example
**Request**
```json
[
   {
      "websiteId":"1",
      "customerGroupId":"1",
      "productId":"1"
   },
   {
      "websiteId":"1",
      "customerGroupId":"1",
      "productId":"2"
   }
]
```

**Response**
```json
[
   {
      "result":{
         "minimalPrice":"10.22"
      },
      "criteria":{
         "websiteId":"1",
         "customerGroupId":"1",
         "productId":"1"
      }
   },
   {
      "result":{
         "minimalPrice":"8.56"
      },
      "criteria":{
         "websiteId":"1",
         "customerGroupId":"1",
         "productId":"2"
      }
   }
]
```

### 2. Return execution result in the order of received requests

Results can be mapped to requests according to the order of incoming arguments.
With this approach, we do not have to return request criteria with results.
Client assumes that the order of received result containers is equal to the order of sent arguments.
Service implementation has to guarantee this sort order.
This approach does not work with services that returns 

#### Payload example
**Request**
```json
[
   {
      "websiteId":"1",
      "customerGroupId":"1",
      "productId":"1"
   },
   {
      "websiteId":"1",
      "customerGroupId":"1",
      "productId":"2"
   }
]
```
**Response**
```json
[
   {
      "minimalPrice":"10.22"
   },
   {
      "minimalPrice":"8.56"
   }
]
```


### 3. Service signs each result container with an identifier that was received from client

A client may identify each set of arguments by assigning a key,
for instance the hash sum of arguments or just arguments set number.
Service assigns these keys to corresponding containers with results.
This approach is simple and similar to the previous one with one significant difference 
it does support services that may return result containers without sort order.
It could be the case if our client will utilise async service that uses several threads and returns data as soon as it ready.

Cons, we have to explicitly send this identifier as a part of the request. 

#### Payload example
**Request**
```json
[
   {
      "id":"0",
      "args":{
         "websiteId":"1",
         "customerGroupId":"1",
         "productId":"1"
      }
   },
   {
      "id":"1",
      "args":{
         "websiteId":"1",
         "customerGroupId":"1",
         "productId":"2"
      }
   }
]
```

**Response**
```json
[
   {
      "id":"0",
      "result":{
         "minimalPrice":"10.22"
      }
   },
   {
      "id":"1",
      "result":{
         "minimalPrice":"8.56"
      }
   }
]
```

## Error reporting

An error may occur on overall service execution or during processing one of the received criteria.
This is up to a developer how the service tolerates and handles runtime exceptions during.
Where it meaningful and applicable a service may return exception wrapped with result container.

```php
public class ErrorContainer implements ContainerInterface
{
    public function getCriteria() : CrtiteriaInterface
    public function getError() : ErrorInterface
}
```
If not applicable, the service may throw an exception according to the case.