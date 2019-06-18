# Alternative checkout flow

In scope of work on GraphQL and storefront APIs we have an opportunity to improve design and features of storefront checkout.

The purpose of this document is to discuss possible alternatives to current Magento checkout flow. 

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
       * Addresses
       * Selected Shipping Methods
     * Billing address (required for virtual products). How to deal with multi-payment?
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
     * Billing addresses
     * Shipping addresses
     * Coupons
     * Gift cards
     * Store credit
     * Customer (optional)
  6. Place Order: Order
     * Quote
     * PaymentMethods with Billing Addresses
