# Overview

In the scope of adding features to the GraphQL schema, a client like PWA needs to know what queries need authentication.
This is needed to know to refresh a token each time it expires, so this logic or what queries need a token doesn't get hardcoded into the client side.
It is also useful to implement the "soft" auth mode for cart, when a token expires, and the client would still store some info from the last sesson client side.

## Annotation

We need add an anotatoin like `@auth` to support metadata that would be needed by the client:

Inspiration came out from this document from apollo: https://www.apollographql.com/docs/graphql-tools/schema-directives/

```java
directive @auth(
  requires: UserRole = ANONYMOUS_USER,
) on OBJECT | FIELD_DEFINITION

enum UserRole {
  REGISTERED_CUSTOMER
  ANONYMOUS_USER
  COMPANY_USER
  ADMIN_USER
}
```

```graphql
type Query {
    cart: Cart! @auth(requires:ANONYMOUS_USER)
    customerCart: Cart! @auth(requires:REGISTERED_CUSTOMER)
    customer: Customer @auth(requires:[REGISTERED_CUSTOMER, COMPANY_USER])
}
```
This can be hardcoded in the schema, or we could introduce an annotation to resolver level classes that could be read automatically

## Automation on reading annotations

We can do some automation on this annotation reading from the resolvers:

```php
/**
 * Get cart for the customer
 *
 * @auth requires:REGISTERED_CUSTOMER
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

In B2b or staging preview we do need some kind of authorization for different user types or levels. Next is about token kinds (customer vs integration or admin)
The same query might use different levels of authorization so we can use an array, this way we can even deprecate some of the redundant queries

```graphql
type Query {
    cart: Cart! @auth(requires:[ANONYMOUS_USER, REGISTERED_CUSTOMER])
    customerCart: Cart! @auth(requires:REGISTERED_CUSTOMER) @deprecated
    customer: Customer @auth(requires:[REGISTERED_CUSTOMER, COMPANY_USER])
```
