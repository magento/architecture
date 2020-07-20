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

#### Submit a return with multiple items and comments

```graphql
mutation {
  createReturn(
    input: {
      items: [
        {
          order_item_id: "0000004", 
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
        product {
          sku
          name
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

### Leave a return comment

```graphql
mutation {
  addReturnComment(
    input: {
      return_id: "000000001", 
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

When return is authorized, the customer should specify shipping and tracking information
