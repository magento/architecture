## Configuration 

The following settings should be accessible via `storeConfig` query:
- Returns functionality status on the storefront: enabled/disabled
- Enable returns on product level

Scenarios which may need these settings include:
- Return initiation for the specific product from the order
- Rendering of the returns section in the customer account

```graphql
{
  storeConfig {
    sales_magento_rma_enabled
    sales_magento_rma_enabled_on_product
  }
}
```

## Use cases

### View return list with pagination

### View return details

### Create a return

#### Render return form with dynamic RMA attributes

#### Add more items to the return

#### Submit return

### Leave a return comment
