 # UrlResolver identifier 
 ## Problem statement   
 
In the client flow, for PWA, in order to load a entity/template, one must make a query to the `urlResolver` to obtain the *id* and *Entity Type* of the content that can be Product, Category or CMS.

####Example:
#####Request:
```graphql
urlResolver(url: "/breathe-easy-tank") {
  id
  type
}
```
#####Response:
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

Deprecate `id` field from `urlResolver` and introduce a new field called identifier, unique to each entity that's not the Id from the database.

```graphql
urlResolver(url: String) {
  id: Int @deprecated
  identifier: String!
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

 ## Alternatives
 
 We can directly return the entity instead of returning the identifier from UrlResolver.
 We can introduce an interface called ResolvedEntityInterface that returns the actual object.
 That means that we either wrap all supported entities into an object and use fragments or all existing objects will have to implement this empty interface.
 
  ```graphql
 interface ResolvedEntityInterface {
 
 }
 
 
  ```