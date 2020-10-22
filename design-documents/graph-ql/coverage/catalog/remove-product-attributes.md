# Remove product dynamic attributes from ProductInterface

## Current state

Currently, Magento exposes EAV attributes as part of the product schema as top-level fields of ProductInterface. As consequences:
* The merchant can change the GraphQL from the admin, which makes it hard to cache GraphQL schema.
* Regular product interface overloaded with fields and almost unreadable.
* It is quite often practice to create product programmatically, so the number of attributes could be really excessive with tens thousands of attributes.
* Two Magento instances do not have the same GraphQL schema. As a result, it is near impossible to build a simple client/SDK, which could be easily transferred from one instance to another.
* It is impossible to aggregate the several Magento instances behind the common [BFF](https://docs.microsoft.com/en-us/azure/architecture/patterns/backends-for-frontends).

## Proposed solution

We should to remove attributes dynamic attributes,
except system attributes which makes sense for the storefront,
from the product top level and use a [dynamic container](https://github.com/magento/architecture/blob/master/design-documents/graph-ql/custom-attributes-container.md) instead.


```graphql
## ProductInteface with system EAV attributes 
type ProductInterface {
    uid: ID
    name: String
    sku: String
    attribute_set_id: Int
    description: ComplexTextValue
    short_description: ComplexTextValue
    meta_title: String
    meta_keyword: String
    meta_description: String
    created_at: String
    updated_at: String
    custom_attributes: [CustomAttribute]
    ...
}
```

## System EAV attributes that should be deprecated at the storefront

* `special_price: Float`  These fields should not be used a the source of the discount since the discount also could be caused by group price or rule price, which are not reflected in this field.
The same is true for `special_from_date: String` and `special_to_date`.

* `options_container: String` - this field just an exposure of `catalog_product_entity` table field, information about product options could be explicitly retrieved from the corresponding field. 
* `manufacturer: Int` - just a sample attribute from Magento 1.x, we can move it to the custom attributes container.
the same is true for `country_of_manufacture: String`
