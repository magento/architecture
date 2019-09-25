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
 
## How to introduce this interface
I propose to deprecate existing single-field ResolverInterface and treat the new interface as the default for Query
resolvers. Right now Query and Mutation resolvers are not differentiated but that can be changed - since Mutation
resolvers will never have _$values_ we could introduce new interface for Mutations and have BatchResolverInterface
dedicated to Queries. Hopefully this move will steer developer into the performance considering approach to resolvers.
 
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
