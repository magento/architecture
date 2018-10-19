**Problem statement**

All resolvers in Magento must implement Resolver interface, which in its turn forces the instance of `\GraphQL\Deferred` to be returned by each resolver. To be precise, Magento wrapper for *deferred*, `\Magento\Framework\GraphQl\Query\Resolver\Value`, must be returned. 

```php
/**
 * Fetches the data from persistence models and format it according to the GraphQL schema.
 *
 * @param \Magento\Framework\GraphQl\Config\Element\Field $field
 * @param $context
 * @param ResolveInfo $info
 * @param array|null $value
 * @param array|null $args
 * @throws \Exception
 * @return Value
 */
public function resolve(
    Field $field,
    $context,
    ResolveInfo $info,
    array $value = null,
    array $args = null
) : Value;
```

The main reason for using *deferred* in GraphQL is to optimize performance by solving [N+1 problem](http://webonyx.github.io/graphql-php/data-fetching/#solving-n1-problem) where it exists.

An example of valid *deferred* usage in Magento can be found in [`\Magento\CatalogGraphQl\Model\Resolver\Product::resolve`](https://github.com/magento/graphql-ce/blob/5a570b8b674b04eb36930aa73f92d1e789b8843a/app/code/Magento/CatalogGraphQl/Model/Resolver/Product.php#L57).

In all other cases, when there is no need to solve N+1 problem, we end up with boilerplate code like:
```php
$result = function () use ($data) {
    return $data;
};

return $this->valueFactory->create($result);
```

Response type unification is one of the reasons why *deferred* was made the only possible return type for resolvers. However, after implementing more resolvers it became obvious that current design just adds unnecessary complexity to the resolver implementations. 

**Proposed solution**

In addition to *deferred*, allow scalars and arrays of scalars as return types for `\Magento\CatalogGraphQl\Model\Resolver\Product::resolve`.

It will also be easier to customize resolver with plugins, if it returns array/scalar instead of `deferred` object.

**Action items**

1. Modify existing resolvers, which have boilerplate code and do not require solving N+1 problem

   Change 
   ```php
    public function resolve(
        Field $field,
        $context,
        ResolveInfo $info,
        array $value = null,
        array $args = null
    ) : Value;
   ```
   to 
   ```php
    public function resolve(
        Field $field,
        $context,
        ResolveInfo $info,
        array $value = null,
        array $args = null
    );
   ```
   Also, replace boilerplate code in resolvers
   ```php
   $result = function () use ($data) {
       return $data;
   };
   
   return $this->valueFactory->create($result);
   ```
   with 
   ```php
   return $data;
   ```
1. Document the decision and make sure that all new resolvers return *deferred* only when necessary
