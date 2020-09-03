## Configuration 

The following settings should be accessible via `storeConfig` query:
- Returns functionality status on the storefront: enabled/disabled

Scenarios which may need these settings include:
- Rendering of the MSRP price returns a 'Money' value or not

```graphql
{
  storeConfig {
    msrp_enabled
  }
}
```

## Use cases

### Querying products and specifially pricing should return MSRP if feature is enabled

```graphql
{
  products(filter: {sku: {eq: "24-WB04"}}, sort: {name: ASC}) {
    items {
      name
      sku
      price_range {
        minimum_price {
          regular_price {
            value
            currency
          }
          final_price {
            value
            currency
          }
          discount {
            amount_off
            percent_off
          }
          msrp {
            value
            currency
          }
        }
        maximum_price {
          regular_price {
            value
            currency
          }
          final_price {
            value
            currency
          }
          discount {
            amount_off
            percent_off
          }
          msrp {
            value
            currency
          }  
        }
      }
    }
  }
}
```
