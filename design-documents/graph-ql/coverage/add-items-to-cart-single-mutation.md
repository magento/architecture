# Single mutation for adding products to cart
In order to simplify the flow of adding different type of products to wishlist and cart it is proposed to use single mutation for all product types.

*Proposed schema for adding products to cart* - [AddProductsToCart](AddProductsToCart.graphqls)

*Proposed schema for adding products to wish list* - [Wishlist](Wishlist.graphqls)

## Definition

- Bundle dynamic, grouped and configurable product are just templates which help merchant to select a set of simple products.
- Product may have selected options and entered options (buyer input).
- Options are sets of predefined values that a buyer can add to cart/wishlist in addition to a product.
- The difference between wish list and cart payloads is that all fields in wishlist are optional.

### Options 

Each product may have options. Option can be of 2 types (see example below):
  - `selected_options` - predefined and selected by customer option. E.g. it can be customizable option: `color: green` or bundle option: `Memory: 24M, Warranty: 1y`:
``` 
    "selected_options" : [
         "Y3VzdG9tLW9wdGlvbi8yNC80Mg=="
     ],
```
  - `entered_options` - option entered by customer like: text field, image, etc.
``` 
    "entered_options": [
      {
        id: "Y3VzdG9tLW9wdGlvbi8zMQ==",
        value: "Vasia's awesome cup"
      }
    ]
```

We can consider "Selected Option" and "ID for Entered Option" as UUID. They meet the criteria:
- Represent specific option. 
- Must be unique across different options
- Returned from server
- Used by client as is

Selected options can be used for:
- Customizable options such as dropdwon, radiobutton, checkbox, etc
- Configugrable product
- Bundle Product
- Downloadable product
- Groupped product

Entered options:
- Customizable options such as text field, file, etc
- Gift Card (amount)
  
#### Option implementation
  
Product schema should be extended in order to provide option UUID.

Option UUID is `base64` encoded string, that encodes details for each option and in most cases can be presented as 
`base64("<option-type>/<option-id>/<option-value-id>")`
For example, for customizable drop-down option "Color(id = 1), with values Red(id = 1), Green(id = 2)" UUID for Color:Red will looks like `"Y3VzdG9tLW9wdGlvbi8xLzE=" => base64("custom-option/1/1")` 

Here is a GQL query that shows how to add a new field "uuid: String!" to cover existing cases:


``` graphql
query {
  products(filter: { price: { from: "0.00001" } }) {
    items {
      sku
      ... on CustomizableProductInterface {
        options {
          option_id
          title
          ... on CustomizableRadioOption {
            title
            value {
              uuid # introduce new UUID field in CustomizableRadioValue
              option_type_id
              title
            }
          }
          ... on CustomizableDropDownOption {
            title
            value {
              uuid # introduce new UUID field in CustomizableDropDownValue
              # see \Magento\QuoteGraphQl\Model\Cart\BuyRequest\CustomizableOptionsDataProvider
              option_type_id
              title
            }
          }
         # ... on all other custom options types
        }
      }      ... on ConfigurableProduct {
        variants {
          attributes {
            uuid # introduce new UUID field in ConfigurableAttributeOption  (format: configurable/<attribute-id>/<value_index>)
            # see \Magento\ConfigurableProductGraphQl\Model\Cart\BuyRequest\SuperAttributeDataProvider
            code
            value_index
          }
        }
      }      ... on DownloadableProduct {
        downloadable_product_links {
          uuid #  introduce new UUID field in DownloadableProductLinks (format: downloadable/link/<link_id>)
          #  see \Magento\DownloadableGraphQl\Model\Cart\BuyRequest\DownloadableLinksDataProvider
          title
        }
      }      ... on BundleProduct {
        items {
          sku
          title
          options {
            uuid #  introduce new UUID field in BundleItemOption (format: bundle/<option-id>/<option-value-id>/<option-quantity>)
            # see \Magento\BundleGraphQl\Model\Cart\BuyRequest\BundleDataProvider
            id
            label
          }
        }
      }      ... on GiftCardProduct {
        giftcard_amounts {
          uuid # introduce new UUID field in GiftCardAmounts (format: giftcard/...TBD)
          # see \Magento\GiftCard\Model\Quote\Item\CartItemProcessor::convertToBuyRequest
          value_id
          website_id
          value
          attribute_id
          website_value
        }
      }
    }
  }
}
```

## Examples

#### Add simple product to cart
```
[
  {
    "sku": "Cup",
    "qty": 1,
    "selected_options" : [
      "Y3VzdG9tLW9wdGlvbi8yNC80Mg==" //base64_encode('custom-option/24/42')
    ],
    "entered_options": [
      {
        id: "Y3VzdG9tLW9wdGlvbi8zMQ==", //base64_encode("custom-option/31")
        value: "Vasia's awesome cup"
      }
    ]
  }
]
```

In this example we want to add _personalized blue cup to cart_ to cart.

 - `selected_options` - predefined and selected by customer options. `base64` encoding will help to use UUID in future.
:warning: The encoded value will be returned from server and should be used by client as is. 

In this example values will be following:

| Name  | Value | Buyer Selection |
| ------------- | ------------- | ------------- |
| option-type  | custom-option   |
| option-id  | 24 | color |
| option-value-id  | 42  | blue |

- `option-type` - predefined list of option types, e.g. downloadable, configurable, bundle, customizable.
- `entered_options` - entered by customer (text, image, etc) and encoded options.

This example is suitable for virtual product and gift card as well.

#### Add virtual product to cart
In this example we want to add _personalized membership with expiration date_ to cart.

| Name  | Value | Buyer Selection |
| ------------- | ------------- | ------------- |
| option-type  | configurable   |
| option-id  | 105 | date |
| option-value-id  | 156  | 12/31/2020 |

```
[
  {
    "sku": "Membership",
    "selected_options" : [
      "Y29uZmlndXJhYmxlLzEwNS8xNTY="
    ],
    "entered_options": [
      {
        id: "Y3VzdG9tLW9wdGlvbi8zMQ==",
        value: "VmFzaWEncyBhd2Vzb21lIGN1cA=="
      }
    ]
  }
]
```

#### Add giftcard to cart
In this example we want to add _100$ gift card_ to cart.
`entered_options` will contain encoded info about gift card amount.
```
[
  {
    "sku": "Gift Card",
    "entered_options": [
      {
        id: "Z2lmdC1jYXJkLWFtb3VudA==",
        value: "MTAw"
      }
    ]
  }
]
```

#### Add downloadable product to cart
```
[
  {
    "sku": "Video tutorial",
    "qty": 1,
    "selected_options" : [
      "ZG93bmxvYWRhYmxlLzEz" //base64_encode('downloadable/13')
    ]
  }
]
```
Where `selected_options` contain link id.

#### Add bundle product to cart
In this example we want to add _dinnerware kit, that contains cup and saucer_ to cart.
```
[
  {
    "sku": "Cup",
    "qty": 1,
    "parent_sku": "Dinnerware Kit",
  },
  {
    "sku": "Saucer",
    "qty": 1,
    "parent_sku":  "Dinnerware Kit",
  }
]
```

:warning: If bundle product is added with options, they should be identical for all children:
```
[
  {
    "sku": "Cup",
    "qty": 1,
    "parent_sku": "Kit",
    "selected_options" : [
      "Y29uZmlndXJhYmxlLzQyLzI0"
    ],
  },
  {
    "sku": "Plate",
    "qty": 1,
    "parent_sku": "Kit",
    "selected_options" : [
      "Y29uZmlndXJhYmxlLzQyLzI0"
    ],
  },
]
```

#### Add configurable product to cart
In this example we want to add _glass blue cup to cart_.
```
[
  {
    "sku": "Glass Cup",
    "qty": 1,
    "selected_options" : [
      "Y29uZmlndXJhYmxlLzQyLzI0"
    ]
  }
]
```

Following approach will work with the partial option selection, for example for wish list purposes.
```
[
  {
    "sku": "Glass Cup Small",
    "qty": 1,
    "parent_sku": "Glass Cup",
    "selected_options" : [
      "Y29uZmlndXJhYmxlLzQyLzI0"
    ],
  }
]
```

#### Add grouped product to cart
```
[
  {
    "sku": "Cup Small",
    "qty": 1
  },
  {
    "sku": "Cup Big",
    "qty": 1
  }
]
```
:warning: Grouped product should be added to cart without parent, but to wish list - with parent.

Wishlist functionality will preserve the same behaviour with the only difference that all fields are optional.

#### Add product to wishlist
```
[
  {
    "sku": "Cup",
    "qty": 1,
    "entered_options": [
      {
        id: 31,
        value: "VmFzaWEncyBhd2Vzb21lIGN1cA=="
      }
    ]
  }
]
```
In this example with partial selection, other required options like color should be added when product is moved from wishlist to cart.
