 ## Problem statement   

Once a customer logs in (uses it's token), an existing guest cart needs to be transferred/merged/imported to the customer cart.
Also the registered customer should be able to work with the same shopping cart on multiple devices that we use that token once we 'login', for example, add products to cart using a laptop and complete checkout using a smartphone using the same customer.
We used the term 'login' in quotes, because there's really no state/session in graphql.

 ## Proposed solution

To achieve desired behavior we need two operations: 

 1. **Get customer cart ID:** There should be a way to retrieve active customer cart ID from the server without using the actual cart Id as a parameter when we use a customer. This only works if we have only one active cart per customer at any given moment.
 Once we get this id it can be used on the rest of the cart mutations/queries. The proposal for this scenario reflected in schema is the following query:
```graphql
customerCart(): Cart! // where Cart! is the customer cart
```
Note: This operation will only work for registered customers (valid customer token must be provided in headers).

### Scenario for point 1.
 
 In the following scenario a logged in user adds a product to the cart. After logging in to his account from the smartphone the cart still contains added product (actual GraphQL queries are replaced with pseudocode for simplicity): 
 
| Step | Device     | Operation                                                                        | Headers        | Response                                          |
|------|------------|----------------------------------------------------------------------------------|----------------|---------------------------------------------------|
| 1    | Laptop     | customerCartId()                                                                 | customer-token | CustomerCart123e4567              |
| 2    | Laptop     | addProductsToCart("CustomerCart123e4567", [{"sku": "productA", "quantity": 2}]) {cart_id}    | customer-token | CustomerCart123e4567              |
| 3    | Smartphone | customerCartId()                                                                 | customer-token | CustomerCart123e4567              |
| 4    | Smartphone | getCart("CustomerCart123e4567") {cart_items {sku, quantity}}                                 | customer-token | {cart_items: [{"sku": "productA", "quantity": 2]} |


 2. **Merge guest cart:** There should be a way to transfer/merge products from a guest cart to a the customer cart. To achieve this we choose to add a mutation that accepts an active guest cart Id, and reflected merged products will be shown directly in the cart response object.
 Proposal for this scenario reflected in schema is the following query:
```graphql
mergeGuestIntoCustomerCart(guestCartId: String): Cart! // where Cart! is the customer cart
```
Note: This operation will only work for registered customers and existing guest carts (valid customer token must be provided in headers). Also, this way se segregate operations as `customerCartId()` as query and `importGuestCart()` as mutation and their intent is very clear. A developer can choose 1st or the 2nd operation depending if the user had products in the cart or not. In both situations you will get the Customer Cart . Also see alternative for #2.
  

 ## Alternatives
 
 1. It is possible to make `cartId` optional argument and rely on `getCart {cart_id}` query for fetching customer cart ID. When used by customer, ID is not required. When used by guest - ID is required.
    - guest still need to use `createEmptyCart` to obtain cart ID
    - `optional` argument makes semantics less clear
    
 2. It is possible to achieve the merging of guest cart by adding an optional argument to `customerCart`, called `guestCartToMerge`. When used it will merge the guest cart into the Customer Cart. This makes the API not very clear, joins a query and a mutation together and could confuse the developer to understand it's intent. The benefit it adds is that we only need 1 call for both operations as converting guest cart into customer cart is a common operation but executed just one time per login.
 