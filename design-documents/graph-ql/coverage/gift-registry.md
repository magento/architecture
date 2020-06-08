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

Then create a new gift registry based on the user input. Registrants are added using a separate mutation, which can be sent in the same request if gift registry ID is client-side generated.
```graphql
# In real query only one should be provided: existing address ID OR address data
mutation CreateGiftRegistryWithRegistrants($giftRegistryData: CreateGiftRegistryInput!, $giftRegistryId: ID!, $registrantsData: [AddGiftRegistryRegistrantInput!]!) {
  createGiftRegistry(gift_registry: $giftRegistryData) {
    gift_registry {
      id
      event_name
      shipping_address {
        id
      	street
    	}
    }
  }
  addGiftRegistryRegistrants(gift_registry_id: $giftRegistryId, registrants: $registrantsData) {
    gift_registry {
      registrants {
        id
        first_name
        last_name
        email
        dynamic_attributes {
          code
          value
          label
        }
      }
    }
  }
}
```
The following JSON represents query variables for the `CreateGiftRegistryWithRegistrants` mutation above.
```json
{
  "giftRegistryId": "optional client-generated ID",
  "giftRegistryData": {
      "id": "optional client-generated ID",
      "event_name": "My Birthday",
      "type_id": "2",
      "message": "Pleas come to my birthday",
      "privacy_settings":"PUBLIC",
      "status": "ACTIVE",
      "shipping_address": {
        "address_id": 3,
        "address_data": {
          "firstname": "John",
          "lastname": "Doe",
          "street": ["123 Some Avenue"],
          "city": "Austin",
          "region": {
            "region_code": "TX"
          },
          "postcode": "78758",
          "company": "Magento",
          "country_code": "US"
        }
      },
      "dynamic_attributes": [
        {
          "code": "event_country",
          "value": "US"
        },
        {
          "code": "event_date",
          "value": "6/2/20"
        }
      ]
  },
  "registrantsData": [
    {
      "email": "John@example.com",
      "first_name": "John",
      "last_name": "Roller",
      "dynamic_attributes": [
        {
          "code": "diet",
          "value": "none"
        }
      ]
    }
  ]
}
```

### Gift registry owner views the list of the gift registries created earlier

```graphql
{
  customer {
    gift_registry_list {
      id
      event_name
      created_on
      message
    }
  }
}
```

### Gift registry owner views and modifies an existing gift registry on edit page

Render the edit gift registry form and populate it with current gift registry properties:

```graphql
{
  customer {
    gift_registry(id: "ID obtained from the list query") {
      event_name
      message
      privacy_settings
      status
      registrants {
        id
        first_name
        last_name
        email
        dynamic_attributes {
          code
          label
          value
        }
      }
      shipping_address {
        id
        firstname
        lastname
        company
        street
        city
        region {
          region
          region_code
        }
        postcode
        country_code
        telephone
        fax
      }
      dynamic_attributes {
        code
        group
        label
        value
      }
      type {
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
  }
}
```

Modify gift registry data:

```graphql
mutation UpdateGiftRegistryWithRegistrants($giftRegistryId: ID!, $giftRegistryData: UpdateGiftRegistryInput!, $registrantsData: [UpdateGiftRegistryRegistrantInput!]!) {
  updateGiftRegistry(id: $giftRegistryId, gift_registry: $giftRegistryData) {
    gift_registry {
      id
      event_name
      shipping_address {
        id
      	street
    	}
    }
  }
  updateGiftRegistryRegistrants(gift_registry_id: $giftRegistryId, registrants: $registrantsData) {
    gift_registry {
      registrants {
        id
        first_name
        last_name
        email
        dynamic_attributes {
          code
          value
          label
        }
      }
    }
  }
}
```
The JSON below should be used as query variable for the gift registry update mutation above:
```json
{
  "giftRegistryId": "existing-gift-registry-ID",
  "giftRegistryData": {
      "event_name": "My Birthday",
      "type_id": "2",
      "message": "Pleas come to my birthday",
      "privacy_settings":"PUBLIC",
      "status": "ACTIVE",
      "shipping_address": {
        "address_id": 3,
        "address_data": {
          "firstname": "John",
          "lastname": "Doe",
          "street": ["123 Some Avenue"],
          "city": "Austin",
          "region": {
            "region_code": "TX"
          },
          "postcode": "78758",
          "company": "Magento",
          "country_code": "US"
        }
      },
      "dynamic_attributes": [
        {
          "code": "event_country",
          "value": "US"
        },
        {
          "code": "event_date",
          "value": "6/2/20"
        }
      ]
  },
  "registrantsData": [
    {
      "id": "existing-registrant-id",
      "email": "John@example.com",
      "first_name": "John",
      "last_name": "Roller",
      "dynamic_attributes": [
        {
          "code": "diet",
          "value": "none"
        }
      ]
    }
  ]
}
``` 

### Gift registry owner removes an existing gift registry

```graphql
mutation RemoveGiftRegistry($giftRegistryId: ID!) {
  removeGiftRegistry(id: $giftRegistryId) {
    is_removed
  }
}
```

### Gift registry owner adds items to the gift registry from cart

First get the cart item data which can be used to add a new item to the gift registry:

```graphql
{
  customerCart {
    items {
      id
      quantity
      product {
        sku
      }
      ... on SimpleCartItem {
        customizable_options {
          id
          values {
            id
            value
          }
        }
      }
      ... on VirtualCartItem {
        customizable_options {
          id
          values {
            id
            value
          }
        }
      }
      ... on DownloadableCartItem {
        customizable_options {
          id
          values {
            id
          }
        }
      }
      ... on BundleCartItem {
        bundle_options {
          id
          values {
            id
            quantity
          }
        }
        customizable_options {
          id
          values {
            id
            value
          }
        }
      }
      ... on ConfigurableCartItem {
        configurable_options {
          id
          value_id
        }
        customizable_options {
          id
          values {
            id
            value
          }
        }
      }
    }
  }
}

```

Based on that information we can send a mutation to add the selected item to gift registry.

```graphql
mutation AddGiftRegistryItems($giftRegistryId: ID!, $giftRegistryItems: [AddGiftRegistryItemInput!]!) {
  addGiftRegistryItems(gift_registry_id: $giftRegistryId, items: $giftRegistryItems) {
    gift_registry {
      event_name
      items {
        id
        product {
          name
          thumbnail {
            url
          }
        }
        selected_customizable_options {
          id
          is_required
          label
          sort_order
          values {
            id
            price {
              type
              units
              value
            }
            value
            label
          }
        }
        added_on
        note
        quantity
        quantity_fulfilled
        ... on BundleGiftRegistryItem {
            selected_bundle_options {
            	id
              label
              type
              values {
                id
                label
                price
                quantity
              }
            }
          }
        ... on ConfigurableGiftRegistryItem {
          selected_configurable_options {
            id
            option_label
            value_id
            value_label
          }
        }
        ... on DownloadableGiftRegistryItem {
          links {
            price
            sample_url
            sort_order
            title
          }
          samples {
            sample_url
            sort_order
            title
          }
        }
        ... on GiftCardGiftRegistryItem {
          sender_name
          recepient_name
          amount {
            currency
            value
          }
          message
        }
      }
    }
  }
}
```

The following JSON should be provided as query variables for the mutation above:

```json
{
  "giftRegistryId": "existing-gift-registry-id",
  "giftRegistryItems": [
    {
      "sku": "custom-hat-red",
      "quantity": 2.0,
      "note": "Really like this color",
      "parent_sku": "custom-hat",
      "selected_options": [
			 	"hash from the color option ID and its value ID",
        "hash from the size option ID and its value ID"
      ],
      "entered_options": [
      	{
          "id": "hash from custom phrase option ID",
          "value": "Custom Hat"
        }
      ]
    }
  ]
}
```

### Gift registry owner adds items to the gift registry from wish list

Wishlist query does not support fetching the information about selected options and need to be extended. After that the same mutation can be used to add items to gift registry as describe above in the use case of adding items to gift registry from cart.

```graphql
{
  customer {
    wishlist {
      items {
        qty
        product {
          sku
        }
      }
    }
  }
}
```

### Gift registry visitor adds items from the gift registry to the cart

?

### Gift registry owner removes items from an existing gift registry


### Storefront application retrieves gift registry search form metadata

?

### Gift registry visitor searches a gift registry by the recipient name

+
Search by registrant name and dynamic attributes.
 
Explicitly excluded scenarios: search by ID and by email. 


### Gift registry owner shares a gift registry with friends

When gift registry is shared with the invitees, the email they receive will contain a link. 
This link will contain gift registry hash as query parameter and should lead to the page processed by the storefront application. 
The application should parse the URL, extract gift registry ID hash and query gift registry details by ID. 


### Gift registry visitor opens a gift registry using the link from email

