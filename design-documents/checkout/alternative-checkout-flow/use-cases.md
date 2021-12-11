## Checkout use cases

The following use cases must be used as a check list to verify the alternative checkout design. The list may be extended when additional use cases are identified.

 - Based on multiple-shipping address, different stocks, etc. we can estimate multiple quotes at the same time
 - The Cart it's just a storage for items and might have only totals estimation
 - Quote and cart will have own TTL
 - The totals calculation is concentrated into a separate independent component
 - Cart might receive updates about Catalog price changes
 - Digital products (skipped shipping information and method)
 - Physical products
 - Buying products for someone (shipping and billing are different)
 - Guest checkout
 - Do not check user context in the server, this should be solved on the controller level
 - Localize currency selector, zip-code, region etc based on the selected country
 - Unified storefront-backend checkout field validation (excluding the one which requires access to the storage)
    - Use some config which will be used for field validation rules in resolvers/JS/etc
 - Cart summary on the “Order success” page, including Guest checkout
 - Tier prices in cart (just rely on tier prices from catalog API)
 - Independent totals calculation for quote and cart
 - Avoid surprise cost increase on the later checkout stages, be as transparent as possible on every step
     - Include default shipping estimate, maybe tax estimate based on geolocation?
     - Make it clear that taxes are not included into cart estimate
     - Give note about taxes/shipping being calculated at the next steps
     - Cart price rules should be calculated during checkout, not just before placing the order
 - Give an option to hide coupon code field by default (replace with link to get the field)
 - Provide applied coupon name in the quote totals summary
 - Provide shipping time estimate
 - Secure payment logo (lock image)
 - Redirect to cart after product was added to cart, display up-sells (by default)
 - Address autocomplete service (based on street complete other fields) via server-side call to google
 - Display currency vs base currency (store exchange rate instead of set of prices in display currency)
 - Enter coupon on the last step, just before placing order. Store credit and rewards just before payment method selection
 - Store pickup shipping method
 - Multi quote (each quote is converted into a separate order)
 - Push notifications do not make sense since checkout is usually pretty quick. The chance of price changing during checkout is slim
 
Questions:
- Are there any legal restrictions on when to display tax in checkout
