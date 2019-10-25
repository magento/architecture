 ## Problem statement   

Once a customer logs in (uses it's token), an existing guest cart needs to be transferred/merged/imported to the customer cart.
Also the registered customer should be able to work with the same shopping cart on multiple devices that we use that token once we 'login', for example, add products to cart using a laptop and complete checkout using a smartphone using the same customer.
We used the term 'login' in quotes, because there's really no state/session in graphql.

 ## Proposed solution

To achieve desired behavior we need two operations: 

 1. **Get customer cart ID:** There should be a way to retrieve active customer cart ID from the server without using the actual cart Id as a parameter when we use a customer. This only works if we have only one active cart per customer at any given moment.
 Once we get this id it can be used on the rest of the cart mutations/queries. 
 
 We have to add `cart_id` part of the Cart object, because we need to return the cart id in case of `customerCart()` and avoid a round trip call to `cart(cart_id)` if we were just to return a string from `customerCart()`
 
 
 The proposal for this scenario reflected in schema is the following query:
```graphql
query {
  customerCart(): Cart! # where Cart! is the customer cart
}
```

### Scenario for point 1.
 
 In the following scenario a logged in user adds a product to the cart, and had a empty cart as a guest that's disregarded on login. After logging in to his account from the smartphone the cart still contains added product (actual GraphQL queries are replaced with pseudocode for simplicity): 
 
#### Precondition
Existing guest cart without products, so we don't really care about it.

| Step | Device     | Operation                                                                        | Headers/Logged-in status | Response                                          |
|------|------------|----------------------------------------------------------------------------------|----------------|---------------------------------------------------|
| 1    | Laptop     | cart(cart_id: "GuestCart456e8901") { cart_items {sku, quantity} }                                                         | Not Logged In | { cart_items: [] }    |

#### Steps
Getting customer cart when we login and using it to add products and seeing the same products on another device that has the same customer logged in.

| Step | Device     | Operation                                                                        | Headers/Logged-in status | Response                                          |
|------|------------|----------------------------------------------------------------------------------|----------------|---------------------------------------------------|
| 1    | Laptop     | customerCart() { cart_id }                                                        | customer-token | { cart_id: "CustomerCart123e4567" }               |
| 2    | Laptop     | addProductsToCart("CustomerCart123e4567", [{"sku": "productA", "quantity": 2}]) {cart_id, cart_items {sku, quantity} }    | customer-token | { cart_id: "CustomerCart123e4567", cart_items: [{"sku": "productA", "quantity": 2}]}|
| 3    | Smartphone | customerCart() { cart_id, cart_items {sku, quantity} }                           | customer-token | { cart_id: "CustomerCart123e4567", cart_items: [{"sku": "productA", "quantity": 2}] }|

#### Postcondition
| Step | Device     | Operation                                                                        | Headers/Logged-in status | Response                                          |
|------|------------|----------------------------------------------------------------------------------|----------------|---------------------------------------------------|
| 1    | Smartphone | cart("CustomerCart123e4567") { cart_id, cart_items {sku, quantity} }             | customer-token | { cart_id: "CustomerCart123e4567", cart_items: [{"sku": "productA", "quantity": 2}] }|
 
*Note 1*: customerCart will return the same cart object representing the same cart_id assigned to the customer, until an order is completed. If an active customer cart wasn't created, then one will be instantly created and always returned. This still qualifies as a query as this creation of cart is invisible to the api.

*Note 2*: This operation will only work for registered customers (valid customer token must be provided in headers).


 2. **Merge guest cart into customer cart:** There should be a way to transfer/merge products from a guest cart to a the customer cart. To achieve this we choose to add a mutation that accepts an active guest cart Id, and reflected merged products will be shown directly in the cart response object.
 Proposal for this scenario reflected in schema is the following query:
```graphql
mutation {
  mergeCarts(source_cart_id: String, destination_cart_id): Cart!  # where source_cart_id is the guest cart id and destination_cart_id is the customer cart
}
```

### Scenario for point 2.

In the following scenario a guest cart already exists, and the guest user already added products to the cart. After logging in to his account from any device the cart needs to import/merge the gust cart and then destroy to guest cart (actual GraphQL queries are replaced with pseudocode for simplicity): 


#### Precondition
Existing guest cart with products

| Step | Logged-in status| Operation                                                                        | Headers        | Response                                          |
|------|--------------|----------------------------------------------------------------------------------|----------------|---------------------------------------------------|
| 1    | Not Logged In| cart(cart_id: "GuestCart456e8901") { cart_items {sku, quantity} }                |                | { cart_items: [{"sku": "productA", "quantity": 2}] }|


#### Steps
Guest cart is merged into customer cart immediately after login by calling the a separate mutation and getting the customerCart form a query, so using 2 calls to GraphqQL after login.

| Step | Logged-in status| Operation                                                                        | Headers        | Response                                          |
|------|--------------|----------------------------------------------------------------------------------|----------------|---------------------------------------------------|
| 1    | Logged In    | customerCart() { cart_id, cart_items {sku, quantity} }                           | customer-token | { cart_id: "CustomerCart123e4567", cart_items: []}|
| 2    | Logged In    | mergeCarts(source_cart_id: "GuestCart456e8901",destination_cart_id: "CustomerCart123e4567") { cart_id, cart_items {sku, quantity} }| customer-token | { cart_id: "CustomerCart123e4567", cart_items: [{"sku": "productA", "quantity": 2}] }|

#### Postcondition
Guest cart is destroyed

| Step | Logged-in status| Operation                                                                        | Headers        | Response                                          |
|------|--------------|----------------------------------------------------------------------------------|----------------|---------------------------------------------------|
| 1    | Not Logged In| cart(cart_id: "GuestCart456e8901") { cart_items {sku, quantity} }                |                | {"errors": [{"message": "Could not find a cart with ID"}]} |


*Note 1*: This operation will initially only for registered customers and existing guest carts (valid customer token must be provided in headers). This leaves a possibility to combine multiple guest or customer carts together, wihtout being specific. Also, this way we segregate operations as `customerCart()` as query and `mergeCarts()` as mutation and their intent is very clear. A developer can choose 1st or the 2nd operation depending if the user had products in the cart or not. In both situations you will get the resulting full cart Cart object. Also see alternative for #2.
*Note 2*: Destroying the guest cart actually creates a problem with the same guest on multiple devices and solution for this this possibility is not covered in this document. We may create another guest cart from a disabled cart if this situation occurs.

 ## Alternatives
 
 1. It is possible to make `cart_id` optional argument and rely on `cart(cart_id: "{cart_id}")` query for fetching customer cart ID. When used by customer, ID is not required. When used by guest - ID is required.
    - guest still need to use `createEmptyCart` to obtain cart ID
    - `optional` argument makes semantics less clear
    
 2. It is possible to achieve the merging of guest cart by adding an optional argument to `customerCart`, called `guest_cart_id_to_merge`. When used it will merge the guest cart into the Customer Cart. This makes an unclear API, also doesn't respect responsibilities, joins a query and a mutation together and could confuse the developer to understand it's intent. The benefit it adds is that we only need 1 call for both operations as converting guest cart into customer cart is a common operation but executed just one time per login.
 
#### Precondition
Existing guest cart with products

| Step | Logged status| Operation                                                                        | Headers        | Response                                          |
|------|--------------|----------------------------------------------------------------------------------|----------------|---------------------------------------------------|
| 1    | Not Logged In| cart(cart_id: "GuestCart456e8901") { cart_items {sku, quantity} }                                                         |  | { cart_items: [{"sku": "productA", "quantity": 2}] }    |

#### Steps
Guest cart is merged into customer cart immediately after login by calling the same mutation/query

| Step | Logged status| Operation                                                                        | Headers        | Response                                          |
|------|--------------|----------------------------------------------------------------------------------|----------------|---------------------------------------------------|
| 1    | Logged In    | customerCart() { cart_id , cart_items {sku, quantity} }                                                         | customer-token | { cart_id: "CustomerCart123e4567", cart_items:[] }               |
| 2    | Logged In    | customerCart(guest_cart_id_to_merge: "GuestCart456e8901") { cart_id, cart_items {sku, quantity} }                     | customer-token | { cart_id: "CustomerCart123e4567", cart_items: [{"sku": "productA", "quantity": 2}] }|

#### Postcondition
Guest cart is destroyed

| 1    | Not Logged In| cart(cart_id: "GuestCart456e8901") { cart_items {sku, quantity} }                         |  | {"errors": [{"message": "Could not find a cart with ID"}]} |
