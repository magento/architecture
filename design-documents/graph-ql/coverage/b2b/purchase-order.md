## Use Cases

### View "My Purchase Orders" list

```graphql
{
  customer {
    purchase_orders(
      filter: {createdBy: {eq: "Active User Name"}}, 
      currentPage: 1, 
      pageSize: 10
    ) {
      items {
        uid
        number
        order {
          number
        }
        purchase_order_date
        created_by
        status
        total {
          grand_total {
            currency
            value
          }
        }
      }
      total_count
      page_info {
        current_page
        page_size
        total_pages
      }
    }
  }
}
```
### View "Requires My Approval" list of purchase orders

```graphql
{
  customer {
    purchase_orders(
      filter: {status: APPROVAL_REQUIRED},
      currentPage: 1, 
      pageSize: 10
    ) {
      items {
        uid
        number
        order {
          number
        }
        purchase_order_date
        created_by
        status
        total {
          grand_total {
            currency
            value
          }
        }
      }
      total_count
      page_info {
        current_page
        page_size
        total_pages
      }
    }
  }
}
``` 
### View "Company Purchase Orders" list

```graphql
{
  customer {
    purchase_orders(
      currentPage: 1, 
      pageSize: 10
    ) {
      items {
        uid
        number
        order {
          number
        }
        purchase_order_date
        created_by
        status
        total {
          grand_total {
            currency
            value
          }
        }
      }
      total_count
      page_info {
        current_page
        page_size
        total_pages
      }
    }
  }
}
```
### View purchase order details

The query should allow to fetch the following data:
 - Items with pagination
 - Basic details
 - Totals
 - Shipping Address
 - Billing Address
 - Payment Method
 - Shipping Method
 - Comments
 - History Log
 - Approval Flow
 
```graphql
{
  customer {
    purchase_order(uid: "abc234hsasdfa") {
      uid
      created_by
      purchase_order_date
      number
      order {
        number
      }
      status
      total {
        subtotal {
          currency
          value
        }
        estimated_taxes {
          amount {
            currency
            value
          }
          rate
          title
        }
        grand_total {
          currency
          value
        }
        shipping_handling {
          total_amount {
            currency
            value
          }
        }
      }
      items(currentPage: 1, pageSize: 10) {
        total_count
        page_info {
          current_page
          page_size
          total_pages
        }
        items {
          uid
          product_name
          product_sku
          product_url_key
          product_type
          product_sale_price {
            currency
            value
          }
          quantity
          selected_options {
            uid
            label
            value
          }
          entered_options {
            uid
            label
            value
          }
          discounts {
            amount {
              currency
              value
            }
            label
          }
        }
      }
      payment_methods {
        name
        type
        additional_data {
          name
          value
        }
      }
      billing_address {
        firstname
        lastname
        street
        city
        region {
          region
        }
        postcode
        country {
          full_name_locale
        }
        telephone
      }
      carrier
      shipping_method
      shipping_address {
        firstname
        lastname
        street
        city
        region {
          region
        }
        postcode
        country {
          full_name_locale
        }
        telephone
      }
      comments(currentPage: 2, pageSize: 10) {
        page_info {
          current_page
          page_size
          total_pages
        }
        total_count
        items {
          uid
          timestamp
          author
          text
        }
      }
      history_log(currentPage: 1, pageSize: 10) {
        page_info {
          current_page
          page_size
          total_pages
        }
        total_count
        items {
          uid
          timestamp
          description
        }
      }
      approval_flow {
        items {
          uid
          status
          title
          description
        }
      }
    }
  }
}
```

### Add items to cart from purchase order

Get details about purchase items order items that need to be added to cart:

```graphql
{
  customer {
    purchase_order(uid: "abc234hsasdfa") {
      uid
      items(currentPage: 1, pageSize: 1000) {
        items {
          uid
          product_sku
          quantity
          selected_options {
            uid
          }
          entered_options {
            uid
            value
          }
          ... on PurchaseOrderBundleItem {
            parent_product_sku
            parent_product_quantity
          }
        }
      }
    }
  }
}
``` 

Using the results from the previous request, execute the following mutation to add items to cart. This will work for all product types except for bundle.

```graphql
mutation {
  addProductsToCart(
    cartId: "existing-cart-id-id", 
    cartItems: [
      {
        sku: "simple-hat", 
        quantity: 2, 
        selected_options: [
          "hash based on custom option for the select type goes here"
        ], 
        entered_options: [
          {
            uid: "hash from custom phrase option ID", 
            value: "Custom Hat"
          }
        ]
      }
    ]
  ) {
    cart {
      items {
        id
      }
    }
  }
}
```

Bundle product is added to cart by specifying `parent_sku` and `parent_quantity`.
`CartItemInput` needs to be extended with a new field `parent_quantity` directly in `QuoteGraphQl` module.

```graphql
mutation {
  addProductsToCart(
    cartId: "existing-cart-id-id", 
    cartItems: [
      {
        sku: "fan-kit-hat", 
        parent_sku: "fan-kit",
        quantity: 2, 
        parent_quantity: 3,
        selected_options: [
          "hash based on custom option for the select type goes here. Must be identical for all bundle children"
        ], 
        entered_options: [
          {
            uid: "hash based on custom phrase option goes here. Must be identical for all bundle children", 
            value: "Custom Text"
          }
        ]
      },
      {
        sku: "fan-kit-scarf", 
        parent_sku: "fan-kit",
        quantity: 1, 
        parent_quantity: 3,
         selected_options: [
          "hash based on custom option for the select type goes here. Must be identical for all bundle children"
        ], 
        entered_options: [
          {
            uid: "hash based on custom phrase option goes here. Must be identical for all bundle children", 
            value: "Custom Text"
          }
        ]
      }
    ]
  ) {
    cart {
      items {
        id
      }
    }
  }
}
```

### Add purchase order comment

### Reject purchase order

### Cancel purchase order

### Approve purchase order

### Get a list of available actions

### Store config
 
Consider combining the following settings and exposing as a single field:
- Check if purchase order is enabled on global level
- If purchase order enabled on company level
- If comapnies are enabled on global level

### View a list of approval rules

Should support pagination.

### View approval rule details

### Create new approval rule

#### Get list of "Applies to" users

#### Get list of "Requires approval from" users

### Update approval rule

### Delete approval rule


## Implementation details

The purchase order GraphQL schema depends on Sales and Customer GraphQL schemas.



