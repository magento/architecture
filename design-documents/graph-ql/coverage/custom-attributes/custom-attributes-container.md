## Problem statement

Some Magento installations may have tens of thousands of custom (EAV) product attributes. This is possible when the merchant is selling thousands of different product types with unique attribute sets.

Custom attributes in GraphQL schema are presented in flat structure, which may be a problem for the client application when performing search in catalog. GraphQL requires all needed fields to be explicitly listed in the query, "get all fields" queries are not supported. At the same time Magento is expected to return all EAV attributes applicable to each product. 

One workaround for "getting all fields" is based on schema introspection, it allows to get names of all fields in `ProductInterface` including EAV attributes. Even with the described workaround client application will have to explicitly list tens of thousands of attributes in the search query because it does not know in advance products belonging to which attribute sets will be returned in search result.

# Proposed solution

To account for dynamic nature of EAV attributes and the need of "getting all fields" in product search queries, we can introduce `custom_attributes: [CustomAttribute]!` container (recommended approach). 
The container will return the list of attributes plus the actual values stored for each entity. 

```graphql
type CustomAttribute {
  selected_attribute_options: [SelectedAttributeOption] # used to store unique options values
  entered_attribute_value: EnteredAttributeValue # used to store the freetype entered values like texts 
  attribute_metadata: AttributeMetadataInterface # metadata of the actual attribute not related to the stored Entity-Attribute value
}

type SelectedAttributeOption {
  attribute_option: AttributeOptionInterface
  attribute_metadata: AttributeMetadataInterface
}

type EnteredAttributeValue {
  attribute_metadata: AttributeMetadataInterface
  value: String
}
```

We could also make value complex type to be able add more complex fields values in the future, but this doesn't seem necessary at this point.
This is also aligned with the Selected, Entered values from customizable options, or configurable product.

As for the attribute we would define a concrete type for each entity, because each entity's attributes are different and values of attributes are also complext
For this reason interfaces on multiple levels are the best approach for composability purposes and satisfying values needs like swatches.

```graphql
type EntityAttributeMetadata implements AttributeMetadataInterface, AttributeMetadataEntityTypeInterface, AttributeMetadataUiTypeInterface {
    ui_input_type: UiInputTypeInterface!
    forms_to_use_in: [CustomAttributesListingsEnum]
}
# --------
interface AttributeMetadataInterface {
    uid: ID # base64Encode(entityID/codeID)
    label: String
    data_type: ObjectDataTypeEnum # string, int, float, boolean etc
    sort_order: Int 
}

interface AttributeMetadataEntityTypeInterface {
    entity_type: EntityTypeEnum
}
interface AttributeMetadataUiTypeInterface {
    ui_input: UiInputTypeEnum
}
```
#### Sample queries for this alternative
```graphql
{
  customner {
    custom_attributes {
      selected_attribute_options {
        attribute_option {
          uid
          is_default
          ... on AttributeOption {
            label
            is_default
          }
        }
        attribute_metadata {
          uid
          code
          label
        }
      }
      entered_attribute_value {
        attribute_metadata {
          uid
          code
          label
        }
        value
      }
      attribute_metadata {
        uid
        code
        label # all attributes have a label per store
        data_type # enum : string, int, float etc
        sort_order # needed for ordering
        ... on CustomerAttributeMetadata {
          entity_type # enum
          forms_to_use_in # only in CustomerAttributeMetadata
          ui_input {
            ui_input_type
            is_value_required
            ... on DropDownInputType {
              default_option_id #id
              attribute_options {
                uid
                is_default
                ... on AttributeOption {
                  label
                }
              }
            }
            ... on TextInputType {
              default_value #sting
              filter
              input_validation {
                input_validation_type
                minimum_text_length
                maximum_text_length
              }
            }
          }
        }
        ... on ProductAttributeMetadata {
          entity_type # enum
          lists_to_use_in # only in ProductAttributeMetadata. As opposed to Customer which only had forms
          ui_input {
            ui_input_type
            is_value_required
            ... on VisualSwatchInputType {
              default_option_id #id
              update_product_preview_image
              use_product_image_for_swatch_if_possible
              attribute_options {
                uid
                is_default
                ... on ColorSwatchAttributeOption {
                  color
                  label
                }
                ... on ImageSwatchAttributeOption {
                  image_path
                  label
                }
              }
            }
            ... on TextSwatchInputType {
              default_option_id #id
              update_product_preview_image
              attribute_options {
                uid
                is_default
                ... on SwatchOptionText {
                  title
                  description
                }
              }
            }
          }
        }
      }
    }
  }
}
```

### Alternatives considered
```graphql
type CustomAttribute {
    code: String!
    values: [String]! # We want to account fo attributes that have single (text, dropdown) and multiple values (checkbox, multiselect)
}
```

```graphql
interface CustomAttributeInterface {
    code: String!
}

type SingleValueCustomAttribute implements CustomAttributeInterface {
    value: String!
}

type MultipleValuesCustomAttribute implements CustomAttributeInterface {
    values: [String]!
}
```

Here `value` is JSON-encoded value of the custom attribute.

Flat representation of custom attributes in `ProductInterface` (and other EAV entities like `Category`, `Customer` etc) will remain intact. The client will a have choice to query custom fields explicitly and have validation of the response against GraphQL schema or to "get all" custom attributes using `custom_attributes` container.

It is necessary to keep in mind that with `custom_attributes` it is not possible to query product EAV attributes selectively, which may lead to performance degradation.

#### Sample queries for this alternative

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

```json
{
  "data": {
    "products": {
      "items": [
        {
          "name": "Test Simple Product",
          "sku": "testSimpleProduct",
          "color": "Red",
          "manufacturer": "Company A",
          "size": null
        },
        {
          "name": "Test Configurable Product",
          "sku": "testConfigProduct",
          "color": null,
          "manufacturer": "Company B",
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

```json
{
  "data": {
    "products": {
      "items": [
        {
          "name": "Test Simple Product",
          "sku": "testSimpleProduct",
          "custom_attributes": [
            {
                "code": "color",
                "value": "Red"
            },
            {
                "code": "manufacturer",
                "value": "Company A"
            }
          ]
        },
        {
          "name": "Test Configurable Product",
          "sku": "testConfigProduct",
          "custom_attributes": [
            {
              "code": "manufacturer",
              "value": "Company B"
            },
            {
              "code": "size",
              "value": "XXL"
            }
          ]
        }
      ]
    }
  }
}
```

### Other Alternatives considered

 1. [Persisted queries](https://github.com/magento/graphql-ce/issues/781) can be leveraged to mitigate described issue with the increased size of the request.
 1. To improve flexibility and allow support of complex structures, `type` can be added to the definition of `CustomAttribute` in the future, if there are valid use cases.
 1. `value` of the `CustomAttribute` can have different format other than JSON, potentially more strict one.
 1. It is possible to eliminate rendering of custom attributes in flat structure from `ProductInterface` (and other EAV entities) and completely rely on containers. We believe this will cause developer experience degradation and does not bring any significant benefits for production deployments. 
