**Overview**

:warning: Current proposal is deprecated in favor of [add-items-to-cart-single-mutation.md](add-items-to-cart-single-mutation.md)

As a Magento developer, I need to manipulate the shopping cart via GraphQL so that I can programmatically create orders on behalf of a shopper.

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
- Server-side asynchronous mutations can be supported in the future on framework level in a way similar to Async REST.

**Proposed schema for adding items to cart:**

- [AddSimpleProductToCart](add-items-to-cart/AddSimpleProductToCart.graphqls)
- [AddBundleProductToCart](add-items-to-cart/AddBundleProductToCart.graphqls)
- [AddConfigurableProductToCart](add-items-to-cart/AddConfigurableProductToCart.graphqls)
- [AddDownloadableProductToCart](add-items-to-cart/AddDownloadableProductToCart.graphqls)
- [AddGiftCardProductToCart](add-items-to-cart/AddGiftCardProductToCart.graphqls)
- [AddGroupedProductToCart](add-items-to-cart/AddGroupedProductToCart.graphqls)
- [AddVirtualProductToCart](add-items-to-cart/AddVirtualProductToCart.graphqls)


**My Account area impacted:** 
- Cart
- Minicart
