## Problem statement

Some Magento installations may have tens of thousands of custom (EAV) product attributes. This is possible when the merchant is selling thousands of different product types with unique attribute sets.

Custom attributes in GraphQL schema are presented in flat structure, which may be a problem for the client application when performing search in catalog. GraphQL requires all needed fields to be explicitly listed in the query, "get all fields" queries are not supported. At the same time Magento is expected to return all EAV attributes applicable to each product. 

One workaround for "getting all fields" is based on schema introspection, it allows to get names of all fields in `ProductInterface` including EAV attributes. Even with the described workaround client application will have to explicitly list tens of thousands of attributes in the search query because it does not know in advance products belonging to which attribute sets will be returned in search result.

# Proposed solution

To account for dynamic nature of EAV attributes and the need of "getting all fields" in product search queries,we can introduce `custom_attributes: [CustomAttribute]!` container. 

```graphql
type CustomAttribute {
    code: String!
    value: [String]! # We want to account fo attributes that have single (text, dropdown) and multiple values (checkbox, multiselect)
}

# We could also make value complex type to be able add more fields in the future
type CustomAttribute {
    code: String!
    value: [CustomAttributeValue]!
}

type CustomAttributeValue {
    value: String!
}
```

Alternative approach would be is to introduce an interface `custom_attributes: [CustomAttributeInterface]!`.

```graphql
type CustomAttributeInterface {
    code: String!
}

type CustomAttribute extends CustomAttributeInterface {
    value: CustomAttributeValue!
}

type CustomAttributeMulti extends CustomAttributeInterface {
    values: [CustomAttributeValue]!
}

type CustomAttributeValue {
    value: String!
}
```

Here `value` is JSON-encoded value of the custom attribute.

Flat representation of custom attributes in `ProductInterface` (and other EAV entities like `Category`, `Customer` etc) will remain intact. The client will a have choice to query custom fields explicitly and have validation of the response against GraphQL schema or to "get all" custom attributes using `custom_attributes` container.

It is necessary to keep in mind that with `custom_attributes` it is not possible to query product EAV attributes selectively, which may lead to performance degradation.

### Sample queries

Current implementation allows the following query
```graphql
{
  products(search: "test") {
    items {
      name
      sku
      color
      manufacturer
      size
    }
  }
}
```

Let's assume the response will be

```graphql
{
  "data": {
    "products": {
      "items": [
        {
          "name": "Test Simple Product",
          "sku": "testSimpleProduct",
          "color": "Red",
          "manufacturer": "Company A"
          "size": null
        },
        {
          "name": "Test Configurable Product",
          "sku": "testConfigProduct",
          "color": null,
          "manufacturer": "Company B"
          "size": "XXL"
        }
      ]
    }
  }
}
```

With the proposed changes the above mentioned queries will still be supported. In addition, the following query will become possible
 
 ```graphql
 {
   products(search: "test") {
     items {
       name
       sku
       custom_attributes {
         code
         value
       }
     }
   }
 }
 ```
Note that color and size are not applicable to some products in the search result. In the previous example they were returned as `null`. In the following example they are not returned at all

```graphql
{
  "data": {
    "products": {
      "items": [
        {
          "name": "Test Simple Product",
          "sku": "testSimpleProduct",
          "custom_attributes": [
            {
                "code": "color"
                "value": "Red"
            },
            {
                "code": "manufacturer"
                "value": "Company A"
            }
          ]
        },
        {
          "name": "Test Configurable Product",
          "sku": "testConfigProduct",
          "custom_attributes": [
            {
              "code": "manufacturer"
              "value": "Company B"
            },
            {
              "code": "size"
              "value": "XXL"
            }
          ]
        },
      ]
    }
  }
}
```

# Alternatives considered

 1. [Persisted queries](https://github.com/magento/graphql-ce/issues/781) can be leveraged to mitigate described issue with the increased size of the request.
 1. To improve flexibility and allow support of complex structures, `type` can be added to the definition of `CustomAttribute` in the future, if there are valid use cases.
 1. `value` of the `CustomAttribute` can have different format other than JSON, potentially more strict one.
 1. It is possible to eliminate rendering of custom attributes in flat structure from `ProductInterface` (and other EAV entities) and completely rely on containers. We believe this will cause developer experience degradation and does not bring any significant benefits for production deployments. 
