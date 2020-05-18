# Overview

In the scope of adding features to the the GraphQL schema, a client like PWA needs to know what queries need authentication.
This is needed to know to refresh a token each time it expires, so this logic or what queries need a token doesn't get hardcoded into the client side.
It is also useful to implement the "soft" auth mode for cart, when a token expires, and the client would still store some info from the last sesson client side.

## Annotation

We need add an anotatoin like `@auth` to support metadata that would be needed by the client:

```graphql
type Query {
    customerCart: Cart! @auth(required:true)
    customer: Customer @auth(required:true)
}
```

This can be hardcoded in the schema, or we could introduce an annotation to resolver level classes that could be read automatically

## Automation on reading annotations

We can do some automation on this annotation reading from the resolvers:

```php
/**
 * Get cart for the customer
 *
 * @auth required: true
 */
class CustomerCart implements ResolverInterface
{
...
     /**
      * @inheritdoc
      */
     public function resolve(Field $field, $context, ResolveInfo $info, array $value = null, array $args = null)
     {
        ...
     }
...
}
```

## ACL possibilities

In B2b or staging preview we do need some kind of ACL for different user types or levels. Next is about token kinds (customer vs integration or admin)

Also the same query might be accessible by different roles/types

```graphql
type Query {
    customerCart: Cart! @auth(type:[Guest,Customer])
    customer: Customer @auth(type:[Customer])
    product: ProductInterface @auth(type:[Guest,Customer,Integration])
}
```

## Proposal
We suggest starting simple with @auth(required:true) and go from there, when we add roles, we'll stop using required.
