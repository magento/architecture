# Problem statement

Storefront applications built on top of Magento GraphQL web APIs need to support the following scenarios:
 - Render advanced catalog search form, which can be configured by the merchant
 - Display only a subset of all available product attributes in catalog listing based on the configuration
 - Render product compare page, which should only display attributes specified by the merchant
 
All these scenarios require specific EAV attribute metadata to be accessible from the storefront application.

Additionally, there should be a way to retrieve metadata for all storefront custom attributes in the system. This information should be cached on the server and on the client. 

# Proposed solution

Relaxing signature of existing `customAttributeMetadata` query by making its `attributes` argument optional will allow to fetch all storefront attributes metadata.

Existing schema:
```graphql
Query.customAttributesMetadata(
    attributes: [AttributeInput!]!
): CustomAttributeMetadata
```

Added schema:

```graphql
Query.customAttributesMetadataV2(
    attributes: [AttributeMetadataInput!]!
): CustomAttributeMetadata

#adding to existing type a choice of uid or code and 
type AttributeMetadataInput {
    attribute_uid: ID
}

type CustomAttributeMetadata { # this replaces existing Attribute type
    items: [AttributeMetadataInterface]
}

interface AttributeMetadataInterface { # base metadata common to all attributes
    uid: ID # base64Encode(entityID/codeID)
    label: String
    data_type: ObjectDataTypeEnum # string, int, float, boolean etc
    sort_order: Int
    entity_type: EntityTypeEnum
    ui_input: UiInputTypeInterface!
}

type CustomerAttributeMetadata implements AttributeMetadataInterface {
    forms_to_use_in: [CustomAttributesListsEnum]
}

type CustomerAddressAttributeMetadata implements AttributeMetadataInterface {
}

type ProductAttributeMetadata implements AttributeMetadataInterface {
    lists_to_use_in: [CustomAttributesListsEnum]
}

type TextUiInputType implements UiInputTypeInterface, TextInputTypeInterface, FilterableTextInputTypeInterface, ValidationTextInputTypeInterface {

}

type TextAreaUiInputType implements UiInputTypeInterface, TextInputTypeInterface, FilterableTextInputTypeInterface, ValidationTextInputTypeInterface {

}

type MultipleLineUiInputType implements UiInputTypeInterface, TextInputTypeInterface, FilterableTextInputTypeInterface, ValidationTextInputTypeInterface {
    lines_count: Int
}

type DateUiInputType implements UiInputTypeInterface, TextInputTypeInterface, FilterableTextInputTypeInterface {
    minimum_date_allowed: String
    maximum_date_allowed: String
}

type FileUiInputType implements UiInputTypeInterface, TextInputTypeInterface, FilterableTextInputTypeInterface {
    maximum_file_size: Int # bytes
    allowed_file_extensions: [String]
}

type ImageUiInputType implements UiInputTypeInterface, TextInputTypeInterface, FilterableTextInputTypeInterface {
    maximum_file_size: Int # bytes
    allowed_file_extensions: [String]
    maximum_image_width: Int # in pixels
    maximum_image_height: Int # in pixels
}

type DropDownUiInputType implements UiInputTypeInterface, SelectableInputTypeInterface, AttributeOptionsInterface {
}

type MultipleSelectUiInputType implements UiInputTypeInterface, SelectableInputTypeInterface, AttributeOptionsInterface {
}

interface SwatchInputTypeInterface {
    update_product_preview_image: Boolean
}

type VisualSwatchUiInputType implements UiInputTypeInterface, SelectableInputTypeInterface, AttributeOptionsInterface, SwatchInputTypeInterface {
    use_product_image_for_swatch_if_possible: Boolean
}

type TextSwatchUiInputType implements UiInputTypeInterface, SelectableInputTypeInterface, AttributeOptionsInterface, SwatchInputTypeInterface {

}

type AttributeOption implements AttributeOptionInterface {
    value: String @deprecated(reason: "use `uid` instead")
    label: String
}

type ColorSwatchAttributeOption implements AttributeOptionInterface {
    color: String # html hex code format
}

type ImageSwatchAttributeOption implements AttributeOptionInterface {
    image_path: String # relative path
}

type TextSwatchAttributeOption implements AttributeOptionInterface {
    description: String
}
```

Additional fields should be added to the metadata response (`Attribute`  type), for example `is_dynamic`, `use_in_compare_products`, `display_in_product_listing`, `use_in_advanced_search`, `advanced_search_input_type`. The exact list of fields must be discussed and approved separately.

See full schema [attributes-metadata.graphqls](attributes-metadata.graphqls)

Introduction of the following query will allow fetching lists of attributes applicable to specific artifacts/listings:
```graphql
customAttributesLists(
    listType: CustomAttributesListingsEnum
): CustomAttributeMetadata

enum CustomAttributesListingsEnum {
    PRODUCTS_COMPARE
    PRODUCTS_LISTING
    ADVANCED_CATALOG_SEARCH
    PRODUCT_SORT
    PRODUCT_FILTER
    PRODUCT_SEARCH_RESULTS
    PRODUCT_AGGREGATIONS
    RMA_FORM
    CUSTOMER_REGISTRATION_FORM
    CUSTOMER_ADDRESS_FORM
}
```

# Alternative solutions

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

### Blocker

It is problematic to support attribute options via schema metadata, while it is possible with `customAttributeMetadata` query.
