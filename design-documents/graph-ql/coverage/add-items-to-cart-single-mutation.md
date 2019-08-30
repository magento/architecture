# Single mutation for adding products to cart
In order to simplify the flow of adding different type of products to wishlist and cart it is proposed to use single mutation for all product types.

*Proposed schema for adding products to cart* - [AddProductsToCart](AddProductsToCart.graphqls)

*Proposed schema for adding products to wish list* - [AddProductsToWishlist](AddProductsToWishlist.graphqls)

## Definition

- Bundle dynamic, grouped and configurable product are just templates which help merchant to select a set of simple products.
- Product may have selected options and entered options (buyer input).
- Options are sets of predefined values that a buyer can add to cart/wishlist in addition to a product.
- Some of the options can be required by its nature but not by API signature.
- The differences in payloads between wishlist and cart is that all fields in wishlist are optional.

### Examples

#### Add simple product to cart
```
[
  {
    "sku": "Cup",
    "qty": 1,
    "selected_options" : [
      base64_encode('configurable/24/42')
    ],
    "entered_options": [
      {
        id: 31,
        value: base64_encode('Vasia's awesome cup')
      }
    ]
  }
]
```

In this example we want to add _personalized blue cup to cart_.

 - `selected_options` - predefined and selected by customer options. `base64` encoding will help to use UUID in future. In this example values will be following:

    | Name  | Value | Real Data |
    | ------------- | ------------- | ------------- |
    | option-type  | configurable   |
    | option-id  | 24 | color |
    | option-value-id  | 42  | blue |
- `option-type` - predefined list of option types, e.g. downloadable link, configurable option, bundle option, customizable option.
- `entered_options` - entered by customer options.

This example is suitable for virtual product and gift card as well.

#### Add downloadable product to cart
Here `selected_options` contain `<option-type>/<link-id>`.
```
[
  {
    "sku": "Video tutorial",
    "qty": 1,
    "selected_options" : [
      base64_encode('downloadable/13')
    ]
  }
]
```

#### Add bundle product to cart
```
[
  {
    "sku": "Cup",
    "qty": 1,
    "parent_sku": "Kit",
  },
  {
    "sku": "Plate",
    "qty": 1,
    "parent_sku":  "Kit",
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
      base64_encode(configurable/42/24)
    ],
  },
  {
    "sku": "Plate",
    "qty": 1,
    "parent_sku": "Kit",
    "selected_options" : [
      base64_encode(configurable/42/24)
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
      base64_encode(configurable/42/24)
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
      base64_encode(configurable/42/24)
    ],
  }
]
```

#### Add grouped product to cart
```
[
  {
    "sku": "Cup Small",
    "qty": 1.
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
        value: base64_encode('Vasia's awesome cup')
      }
    ]
  }
]
```
In this example with partial selection, other required options like color should be added when product is moved from wishlist to cart.
