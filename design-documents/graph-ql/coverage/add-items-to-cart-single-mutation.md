# Single mutation for adding products to cart
In order to simplify the flow of adding different type of products to wishlist and cart it is proposed to use single mutation for all product types.

*Proposed schema for adding products to cart* - [AddProductsToCart](AddProductsToCart.graphqls)

*Proposed schema for adding products to wish list* - [AddProductsToWishlist](AddProductsToWishlist.graphqls)

## Definition

- Bundle dynamic, grouped and configurable product are just templates which help merchant to select a set of simple products.
- Product may have selected options and entered options (buyer input).
- Options are sets of predefined values that a buyer can add to cart/wishlist in addition to a product.
- The difference between wishlist and cart payloads is that all fields in wishlist are optional.

### Examples

#### Add simple product to cart
```
[
  {
    "sku": "Cup",
    "qty": 1,
    "selected_options" : [
      "Y29uZmlndXJhYmxlLzI0LzQy" //base64_encode('configurable/24/42')
    ],
    "entered_options": [
      {
        id: "Y3VzdG9tLW9wdGlvbi8zMQ==", //base64_encode("custom-option/31")
        value: "VmFzaWEncyBhd2Vzb21lIGN1cA==" // base64_encode("Vasia's awesome cup")
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
    | option-type  | configurable   |
    | option-id  | 24 | color |
    | option-value-id  | 42  | blue |
- `option-type` - predefined list of option types, e.g. downloadable, configurable, bundle, customizable.
- `entered_options` - entered by customer and encoded options.

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
