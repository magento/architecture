# Alternative checkout flow

In scope of work on GraphQL and storefront APIs we have an opportunity to improve design and features of storefront checkout.

The purpose of this document is to discuss possible alternatives to current Magento checkout flow. 

In the proposed flow the cart is created as soon as user adds product to cart. The data from cart entity should be enough to render minicart and cart pages. Taxes and other adjustments are not calculated at these steps.

The quote, on the other hand, contains a full break down of all adjustments calculated. It provides the user with the total he has to pay for the items in the cart.

It is possible to split cart items into separate quotes. This can be done based on shipping addresses or shipping sources.

Each quote is used to create a separate order. Multiple payment methods (with independent billing address each) can be selected during each order creation.

  1. When Quote is created?
     * For physical products on Review & Payments step
     * For virtual products billing address has to be entered first
  1. Cart properties:
     * Line Items:
       * SKU
       * Selected options (custom/configurable)
       * Quantity
       * Regular price
       * Price
  4. Applicable cart rules calculator arguments
     * Cart
     * Addresses
     * Date & Time (admin preview)
  2. Quote factory arguments: Quote
     * !Line Items
     * Dimensions (weight & size) (based on LineItems)
     * Shipping (requited for physical products)
       * Address
       * Selected Shipping Method
     * Billing address (required for virtual products). Need use cases
     * Coupons
     * Gift cards
     * Store credit
     * Cart rules (calculated by Applicable catalog rules calculator)
     * Customer
  2. Totals calculator: Totals
     * QuoteArgumentDTO
     * PreviousTotals
  5. Quote
     * LineItems
       * Totals
       * Adjustments
           * Taxes
           * Cart rule discounts
     * Shipping addresses
     * Coupons
     * Gift cards
     * Store credit
     * Customer (optional)
  6. Place Order: Order
     * Quote
     * PaymentMethods with Billing Addresses

### Quote creation flow

![Quote Calculation](../img/alternative-quote-calculation.png)
