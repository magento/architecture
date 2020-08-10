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
        created_at
        created_by
        status
        total {
          currency
          value
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
        created_at
        created_by
        status
        total {
          currency
          value
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
        created_at
        created_by
        status
        total {
          currency
          value
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

#### Items

Should support pagination.

#### Basic details
#### Approval Flow
#### Comments
#### History Log
#### Totals
#### Shipping Address
#### Billing Address
#### Payment Method
#### Shipping Method

### Add items to cart from purchase order

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



