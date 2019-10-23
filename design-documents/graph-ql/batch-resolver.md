# Introduce new BatchResolverInterface as default resolver interface
## Problem
Current resolver interface is used to resolve one field at a time,
enabling batching by returning a Deferred result and GraphQL engine delaying it's execution for as long as possible.
But internal and 3rd party developers will not always understand this optimization mechanism or remember it's existence
or want to bother with callbacks. By introducing a new resolver interface which clearly conveys that fields can
be grouped before resolving takes place will make batch field resolving easier and more clear for developers.
 
## Proposed SPI
* BatchResolverInterface
   
  The new resolver interface for developers to implement. _Resolve_ method receives context and field DTO (they would
  the same for every value,args pair), value and args pairs requested. Value and arguments pairs would be gathered from
  all _field_ occurrences that need resolving until the very last moment when actual results needed to fill
  leaves/branches. Developers will have the ability to group requested entity IDs/search parameters/entity properties
  requested from different branches and retrieve them all at once.
   
  The upside of this interface is that developers wouldn't have to bother with callbacks to perform batch resolving
  and have a list of values,args pairs clearly lined for them to process.
   
  As you can see this interface does not extend existing resolver interface nor will BatchResolverInterface
  implementations will be required to extend any classes. Gathering values,args pairs and providing deferred results
  to the GraphQL engine will be handled by the framework itself.
```php
namespace Magento\Framework\GraphQl\Query\Resolver;

use Magento\Framework\GraphQl\Config\Element\Field;

interface BatchResolverInterface
{
    /**
     * Resolve a field for multiple values and arguments requested.
     *
     * @param ContextInterface $context
     * @param Field $field
     * @param BatchRequestItem[] $requests
     * @return BatchResponse
     * @throws \Throwable
     */
    public function resolve(ContextInterface $context, Field $field, array $requests): BatchResponse;
}
``` 
* BatchRequestItem
   
  DTO. Info, value, args representing request data from trying to resolve a single branch/leaf
```php
namespace Magento\Framework\GraphQl\Query\Resolver;

use Magento\Framework\GraphQl\Config\Element\Field;
use Magento\Framework\GraphQl\Schema\Type\ResolveInfo;

class BatchRequestItem
{
    /**
     * @return ResolveInfo
     */
    public function getInfo(): ResolveInfo;

    /**
     * @return array|null
     */
    public function getValue(): ?array;

    /**
     * @return array|null
     */
    public function getArgs(): ?array;
}
```
* BatchResponse
   
  DTO. Result of batch resolving batches/leaves. Resolved data is grouped by values,args pairs.
```php
namespace Magento\Framework\GraphQl\Query\Resolver;

class BatchResponse
{
    /**
     * Set resolved data for a request.
     * 
     * @param BatchRequestItem $request
     * @param array|int|string|float|Value $response
     */
    public function addResponse(BatchRequestItem $request, $response): void;

    /**
     * Retreive resolved data based on the request.
     *
     * @param BatchRequestItem $item
     * @return mixed|Value
     */
    public function findResponseFor(BatchRequestItem $item);
}
```
 
 
## Proposed SPI for resolvers utilizing new batch service contracts
* BatchContractResolverInterface
   
  This interface is for graphql resolvers that delegate batch query/operation to a
  [batch service contract](../batch-query-services.md). Simple interface allows developers to gather criteria/argument items
  from GraphQL requests and convert service contract result to GraphQL acceptable format.
   
  Only service contracts following batch services specification can be used with these resolvers since the mechanism
  relies on results being returned in the same exact order as requests from a service contract.
   
```php
/**
 * Resolve multiple brunches/leaves by executing a batch service contract.
 */
interface BatchContractResolverInterface
{
    /**
     * Service contract to use, 1st element - class, 2nd - method.
     *
     * @return array
     */
    public function getServiceContract(): array;

    /**
     * Convert GraphQL arguments into a batch service contract argument item.
     *
     * @param ResolveRequestInterface $request
     * @return object
     * @throws GraphQlInputException
     */
    public function convertToArgument(ResolveRequestInterface $request);

    /**
     * Convert service contract result item into resolved brunch/leaf.
     *
     * @param object $result Result item returned from service contract.
     * @param ResolveRequestInterface $request Initial GraphQL request.
     * @return mixed|Value Resolved GraphQL response.
     * @throws GraphQlInputException
     */
    public function prepareResponse($result, ResolveRequestInterface $request);
}
```
 
* ResolveRequestInterface
   
  Interface containing all GraphQL request data (for a single branch/leaf)
   
```php
/**
 * Request for a resolver.
 */
interface ResolveRequestInterface
{
    /**
     * Field metadata.
     *
     * @return Field
     */
    public function getField(): Field;

    /**
     * GraphQL context.
     *
     * @return ContextInterface
     */
    public function getContext(): ContextInterface;

    /**
     * Information associated with the request.
     *
     * @return ResolveInfo
     */
    public function getInfo(): ResolveInfo;

    /**
     * Value passed from upper resolvers.
     *
     * @return array|null
     */
    public function getValue(): ?array;

    /**
     * Arguments from GraphQL request.
     *
     * @return array|null
     */
    public function getArgs(): ?array;
}
```
 
## How to introduce this interface
Existing `ResolverInterface` is still good for mutations so it cannot be deprecated in favour of these new interfaces.
But we have to nudge developers into writing more performance considerate resolvers by utilizing batch resolver interfaces
so we will create a static test that will propose to use either `BatchResolverInterface` or `BatchContractResolverInterface`
instead of regular `ResolverInterface`.
 
## POC
I have created POC by replacing Related/CrossSell/UpSell resolvers by batch resolvers and testing them against
multilevel complex query to _products_.
 
POC can be found [here](https://github.com/AlexMaxHorkun/magento2/tree/batch-graphql-proto)
 
Points of interest:
* [Performance testing information](https://github.com/AlexMaxHorkun/magento2/blob/batch-graphql-proto/batch-graphql-proto.md)
* [BatchRequestInterface](https://github.com/AlexMaxHorkun/magento2/blob/batch-graphql-proto/lib/internal/Magento/Framework/GraphQl/Query/Resolver/BatchResolverInterface.php)
* [UpSell/Related/CrossSell resolver](https://github.com/AlexMaxHorkun/magento2/blob/batch-graphql-proto/app/code/Magento/RelatedProductGraphQl/Model/Resolver/Batch/AbstractLikedProducts.php)
 
Performance test results:
 
With a complex recursive query described in the testing information batch resolver is invoked 2 times vs 20 times
for existing resolvers. The query is being resolved 200ms faster with batch resolver than with existing resolver
with Magento being in _Developer_ mode. 
