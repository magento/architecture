 ## Problem statement   
 
Registered customer should be able to work with the same shopping cart using multiple devices. For example, add products to cart using a laptop and complete checkout using a smartphone.


 ## Proposed solution

To achieve desired behavior there should be a way to retrieve active customer cart ID from the server. Based on the assumption that there could be only one active cart per customer at any given moment this can be done as follows:
```graphql
customerCartId(): String!
```
This operation will only work for registered customers (valid customer token must be provided in headers).

 ### Scenario
 
 In the following scenario a logged in user adds a product to the cart. After logging in to his account from the smartphone the cart still contains added product (actual GraphQL queries are replaced with pseudocode for simplicity): 
 
| Step | Device     | Operation                                                                        | Headers        | Response                                          |
|------|------------|----------------------------------------------------------------------------------|----------------|---------------------------------------------------|
| 1    | Laptop     | customerCartId()                                                                 | customer-token | 123e4567              |
| 2    | Laptop     | addProductsToCart("123e4567", [{"sku": "productA", "quantity": 2}]) {cart_id}    | customer-token | 123e4567              |
| 3    | Smartphone | customerCartId()                                                                 | customer-token | 123e4567              |
| 4    | Smartphone | getCart("123e4567") {cart_items {sku, quantity}}                                 | customer-token | {cart_items: [{"sku": "productA", "quantity": 2]} |


 ## Alternatives
 
 - It is possible to make `cartId` optional argument and rely on `getCart {cart_id}` query for fetching customer cart ID. When used by customer, ID is not required. When used by guest - ID is required.
    - guest still need to use `createEmptyCart` to obtain cart ID
    - `optional` argument makes semantics less clear
