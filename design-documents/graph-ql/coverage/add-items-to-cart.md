**Overview**

As a Magento developer, I need to manipulate the shopping cart via GraphQL so that I can build basic ecommerce experiences for shoppers on the front-end using only GraphQL.

GraphQL needs to provide sufficient mutations (ways to create/update/delete data) for a developer to build out the storefront checkout experience for a shopper.

**Use cases:**
- Both guest and registered shoppers can add new items to cart
- Both guest and registered shoppers can update item qty in cart 
- Both guest and registered shoppers can remove items from cart
- Both guest and registered shoppers can update the configuration (for a configurable product) or quantity of a previously added configurable product in cart
    - Edit Item link > Product page > Update configuration or qty > Update Cart 

**Main decision points:**

- Separate mutations for each product type while adding items to cart. Each operation will be supporting bulk use case
- Uniform interface for guest vs customer
- Separate mutations for each checkout step
    - Create empty cart
    - Add items to cart
    - Set shipment method
    - Set payment method
    - Set addresses
    - Same granularity for updates and removals
- Possibility to combine mutations for checkout steps
- Can create "order in one call" mutation in the future if needed
- Hashed IDs for cart items
- Single input object
- Async nature of the flow must be supported on the client side (via AJAX calls)


**Open questions:**

- Do we want to implement server-side asynchronous mutations by default?

**Proposed schema for adding items to cart:**

- [AddSimpleProductToCart](AddSimpleProductToCart.graphqls)
- [AddBundleProductToCart](AddBundleProductToCart.graphqls)
- [AddConfigurableProductToCart](AddConfigurableProductToCart.graphqls)
- [AddDownloadableProductToCart](AddDownloadableProductToCart.graphqls)
- [AddGiftCardProductToCart](AddGiftCardProductToCart.graphqls)
- [AddGroupedProductToCart](AddGroupedProductToCart.graphqls)
- [AddVirtualProductToCart](AddVirtualProductToCart.graphqls)


**My Account area impacted:** 
- Cart
- Minicart
