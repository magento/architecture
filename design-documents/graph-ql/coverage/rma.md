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

### View return list with pagination in order details

### View return details

### Create a return

#### Determine whether any order items are eligible for return

First, we need to know if there are any order items eligible for return. In the Luma example this will dictate whether "Return" link should be displayed on the order details page.

#### Render return form with dynamic RMA attributes

#### Determine which order items are eligible for return

#### Add more items to the return

#### Submit return

### Leave a return comment

### Create a return for guest order
