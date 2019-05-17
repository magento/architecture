#Batch query services

## Purpose

The main reason for such change is performance.
The system triggers multiple queries during an operation (page load, service call). 
Instead of making several requests for data retrieval,
it would be more efficient to assemble all requests and send them as the single call to query service.
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
## Operations output

Because we may receive multiple requests to one service it makes sense to return the results segregated per the request.
This decision allows us to reduce effort on the client to recognise part of the result that belongs to it.
As a consequence, the client should not keep state (criteria) between requests. 

```php
public class ResultContainer
{
    public function getCriteria() : CrtiteriaInterface
    public function getResults() : array
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
