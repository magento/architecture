# Use cases

## User scenario (Guest/ Logged in) 

Use createCompareList mutation to create a new compare list. The server should create a new list with the items added and return the list_id. The clients can use this list_id for futher operations.

# Create Compare List
```graphql
{
    mutation {
        createCompareList(items: ["100123", "234567", "874321"]) #items optional
    } {
        list_id
        items {
          sku
        }
    }
}
```
* For a guest user, new list will be created
* For a logged in user, exisiting list_id will be returned (Can be extended in future ee - to support multiple compare list per customer)


```graphql
{
    mutation {
        addItemsToCompareList(id: "aefe3cc0-89ec-4e92-8eb0-ae6ac545c5e8", items: ["100123", "234567", "874321"])
    }
}
```
* Registered customer can retrieve an active list ID, as well as item in the list, from the customer query.
```graphql
{
    query {
        customer {
            compare_list(id) { # id filter is optional. Will return an array of compare lists (Future extensibility)
                list_id
                items {
                    sku
                }
            }
        }
    }
}
```
* If the registered customer does not have an active list then null will be returned.

```
assignCompareListToCustomer(customerId: ID!, listId: ID!): CompareList 
```
mutation can be used to assign a guest compare list to a registered customer.

* A buyer can modify the existing list by calling mutation: 
  * `addItemsToCompareList` to add new items to compare list.
  * `removeItemsFromCompareList` to remove items from compare list.

* Compare list query returns information about 
products name, price, SKU, url and the list of attributes
preconfigured at the backoffice.
  
```graphql
{
    query {
        customer {
            compare_list {
                list_id
                attributes {
                    code
                    title
                }
                items {
                    sku
                    name
                    canonical_url
                    priceRange {
                        minimumPrice {
                            final_price
                        }
                    }
                    attributes {
                        code
                        values
                    }
                }
            }
        }
    }
}
```

* Compare list could be assigned to the registered customer after login or account creation. 

## Removing stale comparison list
* Introduce a mutation to removeComparisonList(id: ID!): Boolean, which clients can use to remove the list once the session expires
* A cron job

![compare-list.graphqls](compare-list/compare-list.png)

# Non functional requirements:
* Compare list should return enough data to perform the following command with no additional calls: 
  * adding product to cart.
  * adding product to wishlist.
  
Guest compare list business logic not implemented yet. Additional development required.

## Current limitations

Existing table structure for compare list
```
catalog_compare_item
------------------------------------------------------------------------------
| catalog_compare_item_id | visitor_id | customer_id | product_id | store_id |
==============================================================================
```
The visitor_id (non nullable) is tied to session \Magento\Customer\Model\Visitor in Luma. GraphQl layer should not be tied to Luma's session. So Compare-lists via GraphQl will have to differ/deviate from Luma.

This dependency can be solved by managing the compare list state in a new table
```
catalog_compare_list
------------------------------------------------------------------------------
| entity_id (int) (primary)| masked_id (varchar)(indexed) | customer_id (int) (nullable)(indexed)| product_ids (json) (non nullable)| store_id (int) (non nullable)| last_modfied (date)
==============================================================================
```
* On first client request, a row will be populated with an entity_id, server generated masked_id, customer_id (? if logged in), product_id and store_id.
* masked_id will be used for client communications
* product_ids is a json field for reducing storage complexity. 
* introducing entity_id for the list will enable customers to have more than one compare list (future extensibility)
