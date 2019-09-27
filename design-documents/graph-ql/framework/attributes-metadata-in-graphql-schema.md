## Problem statement

Storefront applications built on top of Magento GraphQL web APIs need to support the following scenarios:
 - Render advanced catalog search form, which can be configured by the merchant
 - Display only a subset of all available product attributes in catalog listing based on the configuration
 - Render product compare page, which should only display attributes specified by the merchant
 
All these scenarios require specific EAV attribute metadata to be accessible from the storefront application.


## Proposed solution

GraphQL schema supports [directives](https://graphql.github.io/graphql-spec/June2018/#sec-Language.Directives) which can be used to describe additional information for types, fields, fragments and operations. Since all EAV attributes are automatically added to GraphQL schema in flat structure as fields of the respective types, it is possible to use directives to expose necessary attribute metadata for the client application.
Client is able to fetch all necessary attribute metadata using an introspection query and cache it for future use.

Even though directives are accessible via schema introspection queries, worth mentioning that it is up to the client application how to use and display them. Currently developer tools like GraphiQL do not display custom directives in the Docs section, however it is possible to write an introspection query to retrieve all of them.
Another potential caveat is the need of flushing schema cache on the client as soon as attributes metadata is modified by the merchant. This should be pretty rare on the stores running in production after deployment phase. 

### Sample schema

`@metadata` directive should be generated and added to the final GraphQL schema during schema merging. All fields defined for the types which represent EAV entities will have `@metadata` attribute in the final schema.

The fields which are defined in the schema explicitly can use `@metadata` directive to override default metadata added by the generator.

Specific metadata to be exposed will be discussed and approved in scope of separate proposal.

In the following example of the schema enriched with auto-generated `@metadata` directives like `@doc` and `@resolver` are omitted for readability:
```graphql
interface ProductInterface {
    name: String @metdata(is_dynamic: false, use_in_compare_products: true, display_in_product_listing: true, use_in_advanced_search: true, advanced_search_input_type: "text")
    sku: String @metdata(is_dynamic: false, use_in_compare_products: false, display_in_product_listing: true, use_in_advanced_search: true, advanced_search_input_type: "text")
    color: String @metdata(is_dynamic: true, use_in_compare_products: false, display_in_product_listing: true, use_in_advanced_search: true, advanced_search_input_type: "dropdown")
    manufacturer: String @metdata(is_dynamic: true, use_in_compare_products: true, display_in_product_listing: false, use_in_advanced_search: false)
}
```

## Alternative solutions

 - Extend existing `customAttributeMetadata` query which allows to retrieve metadata for the specified attributes, and introduce new queries which will allow to get a list of attributes to be used in product compare, product listing etc. Most likely different GraphQL queries will be required for each of the scenarios.  
