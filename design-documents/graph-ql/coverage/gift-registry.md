## Use Cases

### Registered customer creates a new gift registry

First, get a list of available gift registry types and dynamic attributes metadata to render the gift registry creation form:
```graphql
{
  giftRegistryTypes {
    id
    label
    dynamic_attributes_metadata {
      code
      label
      attribute_group
      input_type
      is_required
      sort_order
      ... on GiftRegistryCountryAttributeMetadata {
        show_region
      }
      ... on GiftRegistryEventCountryAttributeMetadata {
        show_region
      }
      ... on GiftRegistrySearcheableAttributeMetadataInterface {
        is_searcheable
      }
      ... on GiftRegistryEventDateAttributeMetadata {
        format
      }
      ... on GiftRegistrySelectAttributeMetadataInterface {
        options {
          code
          is_default
          label
        }
      }
    }
  }
}
``` 

Then create a new gift registry based on the user input:
```graphql
mutation {
  createGiftRegistry(
    gift_registry: {
      id: "optional client-generated ID"
      event_name: "My Birthday"
      type_id: "2"
      message: "Pleas come to my birthday"
      privacy_settings: PUBLIC
      status: ACTIVE
      shipping_address: {
        address_id: 3
      }
      dynamic_attributes: [
        {
          code: "event_country"
          value: "US"
        },
        {
          code: "event_date"
          value: "6/2/20"
        }
      ]
    }
  )
}
```

### Gift registry owner modifies an existing gift registry

### Gift registry owner removes items from an existing gift registry

### Gift registry owner removes an existing gift registry

### Gift registry owner adds items to the gift registry from cart

### Gift registry owner adds items to the gift registry from wish list

### Gift registry owner shares a gift registry with friends

### Gift registry visitor adds items from the gift registry to the cart

### Storefront application retrieves gift registry search form metadata

### Gift registry visitor searches a gift registry by the recipient name

Search by registrant name and dynamic attributes.
 
Explicitly excluded scenarios: search by ID and by email. 

### Gift registry visitor opens a gift registry using the link from email

### Storefront application retrieves gift registry edit form metadata
