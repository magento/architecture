## Configuration 

The following settings should be accessible via `storeConfig` query:
- Returns functionality status on the storefront: enabled/disabled

Scenarios which may need these settings include:
- Rendering of the returns section in the customer account

```graphql
{
  storeConfig {
    sales_magento_rma_enabled
  }
}
```

## Use cases

### View return list with pagination in customer account

```graphql
{
  customer {
    returns(pageSize: 10, currentPage: 2) {
      items {
        id
        creation_date
        customer_name
        status
      }
      page_info {
        current_page
        page_size
        total_pages
      }
      total_count
    }
  }
}
```

### View return list with pagination in order details

```graphql
{
  customer {
    orders(filter: {number: {eq: "00000008"}}) {
      items {
        returns(pageSize: 10, currentPage: 1) {
          items {
            id
            creation_date
            customer_name
            status
          }
          page_info {
            current_page
            page_size
            total_pages
          }
          total_count
        }
      }
    }
  }
}
```

### View return details

```graphql
{
  customer {
    return(id: "0000003") {
      id
      order_id
      creation_date
      customer_email
      customer_name
      status
      shipping {
        tracking {
          id
          carier
          shipping_method
          tracking_number
          status
        }
        address {
          contact_name
          street
          city
          region {
            name
          }
          postcode
          country {
            full_name_locale
          }
          telephone
        }
      }
      comments {
        id
        text
        created_at
        created_by
      }
      items {
        id
        product {
          sku
          name
        }
        custom_attributes {
          id
          label
          value
        }
        request_quantity
        quantity
        status
      }
    }
  }
}
```

### Create a return

#### Determine whether any order items are eligible for return

First, we need to know if there are any order items eligible for return. In the Luma example this is dictating whether "Return" link will be displayed on the order details page.

#### Render return form with dynamic RMA attributes

#### Determine which order items are eligible for return

#### Add more items to the return

#### Submit return

### Leave a return comment

### Create a return for guest order

Guest orders are not accessible via GraphQL yet, but the schema of returns will be identical to the one for customer orders.

### Specify shipping and tracking

When return is authorized, the customer should specify shipping and tracking information
