 # UrlResolver identifier 
 ## Problem statement   
 
In the client flow, for PWA, in order to load a entity/template, one must make a query to the `urlResolver` to obtain the *id* and *Entity Type* of the content that can be Product, Category or CMS.

#### Example:
##### Request:
```graphql
urlResolver(url: "/breathe-easy-tank") {
  id
  type
}
```
##### Response:
```json
{
  "data": {
    "urlResolver": {
      "id": 1820,
      "type": "PRODUCT"
    }
  }
}
```

Currently we can't use this Id in Product to filter by, because of how Search API works, also CMS has another unique identifier that can be used.
Also we want not to expose database autoincrement ID in graphql in the future because of replacements with UUID.

 ## Proposed solution


```graphql
urlResolverV2(url: String) {
  id: ID!
  ....
}
```

This ID has to be filterable in all entities, even in product

 ```graphql
 products(filter: { id: { eq: []"$valueOfIdentifierFromUrlResolver"}}) {
     items {
       id
       name
     }
   }
 ```
As of now this feature is not possible, but it has to be introduced to unlock this flow.

 ## Alternatives
 
 ##### #1
 We can directly return the entity instead of returning the identifier from UrlResolver.
 There's a different proposal fot this in [storefront-route.md](https://github.com/magento/architecture/blob/master/design-documents/graph-ql/coverage/storefront-route.md)
 
 We can introduce an interface called ResolvedEntityInterface that returns the actual object.
 That means that we either wrap all supported entities into an object and use fragments or all existing objects will have to implement this empty interface.
 
  ```graphql
 interface ResolvedEntityInterface {
 
 }
  ```
  
  We may need to keep urlResolverV2 and it's field `id` and introduce `urlResolverV2` as type for client-side caching. This might require a closer look to expose affected areas.
  
  
 ##### #2
  We could Deprecate `id` field from `urlResolver` and introduce a new field called identifier, unique to each entity that's not the Id from the database.
```graphql
urlResolverV2(url: String) {
  identifier: String!
  ....
}
```

Where `identifier` represents:
 
 - `sku` value - if Entity is a product
 - `id` value - if Entity is a category
 - `identifier` value - if Entity is a cms page (naming coincidence)
 
 We then can use this identifier to query the Product, Category or CmsPage
 ```graphql
 products(filter: { sku: "$valueOfIdentifierFromUrlResolver"}) {
     items {
       id
       name
     }
   }
 ```