## Configuration

The following configurations must be exposed via existing `storeConfig` query:
- Enable gift registry
- Maximum registrants

## Use Cases

### Registered customer creates a new gift registry

First, get a list of available gift registry types and dynamic attributes metadata to render the gift registry creation form:
```graphql
{
  giftRegistryTypes {
    uid
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
mutation CreateGiftRegistryWithRegistrants($giftRegistryData: CreateGiftRegistryInput!, $giftRegistryUid: ID!, $registrantsData: [AddGiftRegistryRegistrantInput!]!) {
  createGiftRegistry(giftRegistry: $giftRegistryData) {
    gift_registry {
      uid
      event_name
      shipping_address {
        id
      	street
    	}
    }
  }
  addGiftRegistryRegistrants(giftRegistryUid: $giftRegistryUid, registrants: $registrantsData) {
    gift_registry {
      registrants {
        iud
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
  "giftRegistryUid": "optional client-generated UID",
  "giftRegistryData": {
      "uid": "optional client-generated UID",
      "event_name": "My Birthday",
      "giftRegistryTypeUid": "2",
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
    gift_registries {
      uid
      event_name
      created_at
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
    gift_registry(giftRegistryUid: "ID obtained from the list query") {
      event_name
      message
      privacy_settings
      status
      registrants {
        uid
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
mutation UpdateGiftRegistryWithRegistrants($giftRegistryUid: ID!, $giftRegistryData: UpdateGiftRegistryInput!, $registrantsData: [UpdateGiftRegistryRegistrantInput!]!) {
  updateGiftRegistry(giftRegistryUid: $giftRegistryUid, giftRegistry: $giftRegistryData) {
    gift_registry {
      uid
      event_name
      shipping_address {
        id
      	street
    	}
    }
  }
  updateGiftRegistryRegistrants(giftRegistryUid: $giftRegistryUid, registrants: $registrantsData) {
    gift_registry {
      registrants {
        uid
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
  "giftRegistryUid": "existing-gift-registry-ID",
  "giftRegistryData": {
      "event_name": "My Birthday",
      "giftRegistryTypeUid": "2",
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
      "uid": "existing-registrant-uid",
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
mutation RemoveGiftRegistry($giftRegistryUid: ID!) {
  removeGiftRegistry(giftRegistryUid: $giftRegistryUid) {
    is_removed
  }
}
```

### Adding items to the gift registry

It should be possible to additems to the gift registry from the product/category page, cart or wishlist. 
Selected and entered options are specified in a form of hash, which is based on option type, selected values and potentially quantity. 
It is critical to have ability to avoid the hash generation on the client, that is why it must be accessible through products, cart, wishlist and gift registry queries.

#### Getting details about cart item which needs to be added to gift registry

```graphql
{
  customerCart {
    items {
      uid
      quantity
      product {
        sku
      }
      customizable_options {
        uid
      }
      ... on BundleCartItem {
        bundle_options {
          values {
            child_sku
            quantity
          }
        }
      }
      ... on ConfigurableCartItem {
        child_sku
        configurable_options {
          uid
        }
      }
      ... on DownloadableCartItem {
        links_v2 {
          uid
        }
      }
      ... on GiftCardCartItem {
        amount {
          uid
        }
      }
    }
  }
}
```

#### Getting details about wishlist item which needs to be added to gift registry

Note that if the item is not fully configured, the user must be redirected to the product page to complete selections before the item can be added to the gift registry.

Query structure is almost identical to the query for getting items  

```graphql
{
  customer {
    wishlist {
      items {
        uid
        quantity
        product {
          sku
        }
        customizable_options {
          uid
        }
        ... on BundleWishlistItem {
          bundle_options {
            values {
              child_sku
              quantity
            }
          }
        }
        ... on ConfigurableWishlistItem {
          child_sku
          configurable_options {
            uid
          }
        }
        ... on DownloadableWishlistItem {
          links_v2 {
            uid
          }
        }
        ... on GiftCardWishlistItem {
          uid
        }
      }
    }
  }
}
```

#### Executing mutation to add items to gift registry

Based on that information we can send a mutation to add the selected item to gift registry.

```graphql
mutation AddGiftRegistryItems($giftRegistryUid: ID!, $giftRegistryItems: [AddGiftRegistryItemInput!]!) {
  addGiftRegistryItems(giftRegistryUid: $giftRegistryUid, items: $giftRegistryItems) {
    gift_registry {
      event_name
      items {
        uid
        product {
          name
          thumbnail {
            url
          }
        }
        selected_customizable_options {
          uid
          is_required
          label
          sort_order
          values {
            uid
            price {
              type
              units
              value
            }
            value
            label
          }
        }
        added_at
        note
        quantity
        quantity_fulfilled
        ... on BundleGiftRegistryItem {
            selected_bundle_options {
            	uid
              label
              type
              values {
                uid
                label
                price
                quantity
              }
            }
          }
        ... on ConfigurableGiftRegistryItem {
          selected_configurable_options {
            uid
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
          recipient_name
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

Simple product with custom options:

```json
{
  "giftRegistryUid": "existing-gift-registry-uid",
  "giftRegistryItems": [
    {
      "sku": "simple-hat",
      "quantity": 2,
      "note": "Really like this color",
      "selected_options": [
        "hash based on custom option for the select type goes here"
      ],
      "entered_options": [
      	{
          "uid": "hash from custom phrase option UID",
          "value": "Custom Hat"
        }
      ]
    }
  ]
}
```

Configurable product with custom options:
```json
{
  "giftRegistryUid": "existing-gift-registry-uid",
  "giftRegistryItems": [
    {
      "sku": "custom-hat",
      "quantity": 2,
      "note": "Really like this color",
      "selected_options": [
        "hash from the color configurable option ID and its value ID",
        "hash from the size configurable option ID and its value ID",
        "hash based on custom option for the select type goes here"
      ],
      "entered_options": [
      	{
          "uid": "hash from custom phrase option UID",
          "value": "Custom Hat"
        }
      ]
    }
  ]
}
```

Bundle product with custom options:

```json
{
  "giftRegistryUid": "existing-gift-registry-uid",
  "giftRegistryItems": [
    {
      "sku": "fan-kit-hat",
      "parent_sku": "fan-kit",
      "quantity": 2,
      "parent_quantity": 3,
      "note": "Really like this color. Must be identical for all bundle children.",
      "selected_options": [
        "hash based on custom option for the select type goes here. Must be identical for all bundle children."
      ],
      "entered_options": [
      	{
          "uid": "hash based on custom phrase option goes here. Must be identical for all bundle children.",
          "value": "Custom Text"
        }
      ]
    },
    {
      "sku": "fan-kit-scarf",
      "parent_sku": "fan-kit",
      "quantity": 1,
      "parent_quantity": 3,
      "note": "Really like this color. Must be identical for all bundle children.",
      "selected_options": [
        "hash based on custom option for the select type goes here. Must be identical for all bundle children."
      ],
      "entered_options": [
      	{
          "uid": "hash based on custom phrase option goes here. Must be identical for all bundle children.",
          "value": "Custom Text"
        }
      ]
    }
  ]
}
```

Downloadable product with custom options:
```json
{
  "giftRegistryUid": "existing-gift-registry-uid",
  "giftRegistryItems": [
    {
      "sku": "downloadable-product",
      "quantity": 2,
      "selected_options": [
        "hash from the selected link A",
        "hash from the selected link B",
        "hash from the selected custom option"
      ],
      "entered_options": [
      	{
          "uid": "hash from custom text option UID",
          "value": "Custom Entry"
        }
      ]
    }
  ]
}
```

Gift card product with custom options:
```json
{
  "giftRegistryUid": "existing-gift-registry-uid",
  "giftRegistryItems": [
    {
      "sku": "giftcard-product",
      "quantity": 2,
      "selected_options": [
        "hash from the selected gift card amount",
        "hash from the selected custom option"
      ],
      "entered_options": [
      	{
          "uid": "hash from custom text option UID",
          "value": "Custom Entry"
        }
      ]
    }
  ]
}
```

Virtual card product with custom options:
```json
{
  "giftRegistryUid": "existing-gift-registry-uid",
  "giftRegistryItems": [
    {
      "sku": "virtual-product",
      "quantity": 2,
      "selected_options": [
        "hash from the selected custom option"
      ],
      "entered_options": [
      	{
          "uid": "hash from custom text option ID",
          "value": "Custom Entry"
        }
      ]
    }
  ]
}
```

### Gift registry visitor adds items from the gift registry to the cart

The following query returns enough data to add product to cart or wishlist.
```graphql
{
  giftRegistry(uid: "existing-gift-registry-uid") {
    items {
      uid
      quantity
      product {
        sku
      }
      customizable_options {
        uid
      }
      ... on BundleGiftRegistryItem {
        bundle_options {
          values {
            child_sku
            quantity
          }
        }
      }
        ... on ConfigurableGiftRegistryItem {
        child_sku
        configurable_options {
          uid
        }
      }
      ... on DownloadableGiftRegistryItem {
        links {
          uid
        }
      }
      ... on GiftCardGiftRegistryItem {
        uid
      }
    }
  }
}
```

### Gift registry owner removes items from an existing gift registry

```graphql
mutation RemoveGiftRegistryItem($giftRegistryUid: ID!, $itemUids: [ID!]!) {
  removeGiftRegistryItems(giftRegistryUid: $giftRegistryUid, itemUids: $itemUids) {
    gift_registry {
      items {
        uid
      }
    }
  }
}
```

Query variables:
```json
{
  "giftRegistryUid": "existing-gift-registry-uid",
  "itemIds": ["item-one-uid", "item-two-uid"]
}
```

### Gift registry visitor searches a gift registry by the recipient name

First, storefront application retrieves gift registry search form metadata:

```graphql
{
  giftRegistryTypes {
    uid
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

Search by registrant name and dynamic attributes.
 
Explicitly excluded scenarios: search by ID and by email.

```graphql
{
  giftRegistrySearch(
    registrantFirstname: "John", 
    registrantLastname: "Roller", 
    giftRegistryTypeUid: "2",
    searchableDynamicAttributes: [{code: "event_country", value: "US"}]
  ) {
    uid
    event_name
    dynamic_attributes {
      code
      group
      label
      value
    }
  }
}
```

### Gift registry owner shares a gift registry with friends

When gift registry is shared with the invitees, the email they receive will contain a link. 
This link will contain gift registry hash as query parameter and should lead to the page processed by the storefront application. 
The application should parse the URL, extract gift registry ID hash and query gift registry details by ID. 

```graphql
mutation ShareGiftRegistry($giftRegistryUid: ID!, $sender: ShareGiftRegistrySenderInput!, $invitees: [ShareGiftRegistryInviteeInput!]!) {
  shareGiftRegistry(giftRegistryUid: $uid, sender: $sender, invitees: $invitees) {
    is_shared
  }
}
```

The following JSON should be provided as query variables:

```json
{
  "uid": "existing-gift-registry-uid",
  "sender": {
    "message": "Hi, please come to my birth day",
    "name": "John Roller"
  },
  "invitees": [
     {
      "email": "invitee@example.com",
      "name": "John Doe"
    }
  ]
}
```

### Gift registry visitor opens a gift registry using the link from email

```graphql
{
  giftRegistry(uid: "ID obtained from the invitation link in email") {
    event_name
    message
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
```
