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
    return(id: "23as452gsa") {
      number
      order {
        number
      }
      creation_date
      customer_email
      customer_name
      status
      comments {
        id
        text
        created_at
        created_by
      }
      items {
        id
        order_item {
          product_sku
          product_name
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
      shipping {
        tracking {
          id
          carrier {
            label
          }
          tracking_number
        }
        address {
          contact_name
          street
          city
          region {
            code
          }
          country {
            full_name_english
          }
          postcode
          telephone
        }
      }
    }
  }
}
```

### Create a return request

#### Determine whether any order items are eligible for return

There is a need to know if any order items are eligible for return. In the Luma example this is dictating whether "Return" link will be displayed on the order details page.

```graphql
{
  customer {
    orders(filter: {number: {eq: "00000008"}}) {
      items {
        items_eligible_for_return {
          id
          product_name
        }
      }
    }
  }
}
```

Alternative approach is to use a flag added to order items:

```graphql
{
  customer {
    orders(filter: {number: {eq: "00000008"}}) {
      items {
        items {
          id
          eligible_for_return
        }
      }
    }
  }
}
```

#### Render return form with dynamic RMA attributes

```graphql
{
  pageSpecificCustomAttributes(page_type: RETURN_ITEM_EDIT_FORM) {
    items {
      id
      attribute_code
      attribute_type
      input_type
      attribute_options {
        id
        label
        value
      }
    }
  }
}
```

Existing schema of `Attribute` and `AttributeOption` must be extended to provide `ID` field, which will be used in mutations to specify custom attribute values for returns.

#### Determine which order items are eligible for return

```graphql
{
  customer {
    orders(filter: {number: {eq: "00000008"}}) {
      items {
        items_eligible_for_return {
          id
          product_name
        }
      }
    }
  }
}
```

#### Submit a return request with multiple items and comments

```graphql
mutation {
  requestReturn(
    input: {
      order_id: 12345
      contact_email: "returnemail@magento.com"
      items: [
        {
          order_item_id: "absdfj2l3415", 
          quantity_to_return: 1, 
          selected_custom_attributes: ["encoded-custom-select-attribute-value-id"], 
          entered_custom_attributes: [{id: "encoded-custom-text-attribute-id", value: "Custom attribute value"}]
        }
      ], 
      comment_text: "Return comment"
    }
  ) {
    return {
      id
      items {
        id
        quantity
        request_quantity
        order_item {
          product_sku
          product_name
        }
        custom_attributes {
          id
          label
          value
        }
      }
      comments {
        created_at
        created_by
        text
      }
    }
  }
}
```

Alternatively, the collection of returns associated with the order can be requested.

### Leave a return comment

```graphql
mutation {
  addReturnComment(
    input: {
      return_id: "23as452gsa", 
      comment_text: "Another return comment"
    }
  ) {
    return {
      id
      comments {
        created_at
        created_by
        text
      }
    }
  }
}
```

### Create a return for guest order

Guest orders are not accessible via GraphQL yet, but the schema of returns will be identical to the one for customer orders.

### Specify shipping and tracking

When return is authorized by the admin user, the customer can specify shipping and tracking information.

First, the client needs to get shipping carriers that can be used for returns:

```graphql
{
  customer {
    return(id: "asdgah2341") {
      available_shipping_carriers {
        id
        label
      }
    }
  }
}
```

Then tracking information can be submitted:

```graphql
mutation {
  addReturnTracking(
    input: {
      return_id: "23as452gsa", 
      carrier_id: "carrier-id", 
      tracking_number: "4234213"
    }
  ) {
    return {
      shipping {
        tracking {
          id
          carrier {
            label
          }
          tracking_number
        }
      }
    }
  }
}
```

If the user decides to view the status of the specific tracking item, it can be retrieved using the following query:

```graphql
{
  customer {
    return(id: "23as452gsa") {
      shipping {
        tracking(id: "return-tracking-id") {
          id
          carrier {
            label
          }
          tracking_number
          status {
            text
            type
          }         
        }
      }
    }
  }
}
```

In case the return shipping needs to be removed, the following mutation can be used:

```graphql
mutation {
  removeReturnTracking(
    input: {
      return_shipping_tracking_id: "return-tracking-id"
    }
  ) {
    return {
      shipping {
        tracking {
          id
          carrier {
            label
          }
          tracking_number
        }
      }
    }
  }
}

```
