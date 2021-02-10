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
#adding to existing type a choice of uid or code and 
type AttributeInput {
    attribute_code: String #deprecated
    entity_type: String #deprecated
    uid: ID # we will use either uid or a (attribute_code, entity_type) pair.
}

type CustomAttributeMetadata {
    items: [AttributeMetadataInterface]
}

#base metadata common to all attributes
interface AttributeMetadataInterface {
    uid: ID # base64Encode(entityID/codeID)
    attribute_code: String @deprecated(reason: "Use `uid` instead")
    label: String
    data_type: ObjectDataTypeEnum # string, int, float, boolean etc
    sort_order: Int
}

interface AttributeMetadataEntityTypeInterface {
    entity_type: EntityTypeEnum
}

interface AttributeMetadataUiTypeInterface {
    ui_input_type: UiInputTypeEnum
}

type CustomerAttributeMetadata implements AttributeMetadataInterface, AttributeMetadataEntityTypeInterface, AttributeMetadataUiTypeInterface {
    ui_input_type: UiInputTypeInterface!
    forms_to_use_in: [CustomAttributesListingsEnum]
}

type CustomerAddressAttributeMetadata implements AttributeMetadataInterface {
}

type ProductAttributeMetadata implements AttributeMetadataInterface {
}

# interfaces for different types used in inputs--------------

interface UiInputTypeInterface {
    ui_input_type: EntityTypeEnum
    value_required: Boolean!
}

interface InputFilterInterface {
    filter: InputValidationFilterEnum
}

interface InputValidationInterface {
    input_validation_type: InputValidationTypeEnum
}

interface AttributeOptionsInterface {
    attribute_options: [AttributeOptionInterface]
}

# --------------
type TextInputType implements UiInputTypeInterface, InputFilterInterface, InputFilterInterface {
    default_value: String
}

type InputValidationNone implements InputValidationInterface {
}

type InputValidationLength implements InputValidationInterface {
    minimum_text_length: Int
    maximum_text_length: Int
}

#--------------

type TextAreaInputType implements UiInputTypeInterface, InputFilterInterface {
    default_value: String
}

type MultipleLineInputType implements UiInputTypeInterface, InputValidationInterface, InputFilterInterface {
    default_value: String
    lines_count: Int
}

type FileInputType implements UiInputTypeInterface, InputValidationInterface {
    default_value: String
    maximum_file_size: Int # bytes
    allowed_file_extensions: [String]
}

type DropDownInputType implements UiInputTypeInterface, InputValidationInterface, AttributeOptionsInterface {
    default_value: String
}

#other types for other entities here

type AttributeOptionInterface {
    uid: ID! # base64Encode(entityID/codeID/OptionID)
    is_default: Boolean # marks if an option should be default
}

# base type of an option existing type
type AttributeOption implements AttributeOptionInterface {
    # UID and is_default is imported
    value: String @deprecated(reason: "use `uid` instead")
    label: String
}

# extended type of an option as it would look like for visual swatches
type AttributeOptionSwatch implements AttributeOptionInterface {
    swatch: SwatchOptionInterface
}

interface SwatchOptionInterface {
}

type SwatchOptionColor implements SwatchOptionInterface {
    color: String # html hex code format
}

type SwatchOptionImage implements SwatchOptionInterface {
    image_path: String # relative path
}
```

Additional fields should be added to the metadata response (`Attribute`  type), for example `is_dynamic`, `use_in_compare_products`, `display_in_product_listing`, `use_in_advanced_search`, `advanced_search_input_type`. The exact list of fields must be discussed and approved separately.

Introduction of the following query will allow fetching lists of attributes applicable to specific artifacts/listings:
```graphql
customAttributesListings(
    listing_type: CustomAttributesListingsEnum
): CustomAttributeMetadata

enum CustomAttributesListingsEnum {
    PRODUCTS_COMPARE
    PRODUCTS_LISTING
    ADVANCED_CATALOG_SEARCH
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
