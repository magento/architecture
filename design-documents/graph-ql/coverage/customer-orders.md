# Customer Orders API

## Overview

The GraphQL API should provide a possibility to retrieve orders, shipments, invoices, credit memos for the logged in customer. The current schema allows fetching only simple order details and doesn't provide a possibility to fetch order details by a number.

The proposed queries might look like:

```graphql
# query to return a list of all customer orders
type Query {
    customerOrders (
        currentPage: Int = 1 # current page of the customer order list. default is 1.
        pageSize: Int = 20 # page size for the customer orders list. default is 20.
    ): CustomerOrders
}

# collection of customer orders that contains pagination and individual order details.
type CustomerOrders {
    items: [CustomerOrder] # collection of customer orders that contains individual order details.
    currentPage: Int # current page of the customer order list. default is 1.
    pageSize: Int # page size for the customer orders list. default is 20.
}

# query to return an individual order detail by order number.
type Query {
    customerOrder (
        order_number: String # order number for the query input.
    ): CustomerOrder   
}
```
## Order Type Schema

The proposed type for the customer order might look like:

```graphql
# order details
type Order {
    order_date: String! # date when the order was placed.
    status: String! # current status of the order.
    order_number: String! # order number.
    items: [OrderItem] # collection of all the items purchased
    order_pricing: OrderPricingInterface! # pricing details for the order.
    invoices: [Invoice] # invoice list for the order.
    credit_memos: [CreditMemo] # credit memo list for the order.
    shipments: [OrderShipment] # shipping list for the order.
    payment_method: [PaymentMethod] # payment details for the order.
}
```

The `grand_total`, `id`, `increment_id`, `created_at` will be marked as deprecated.

### Order Item

The order items will be presented as separate interface which will have multiple implementations for invoice, shipment and credit memo types.

```graphql
interface ProductItemInterface {
    name: String # name of the base product.
    sku: String! # sku of the base product.
    url: String # url of the base product.
    final_price: Float! # final price for the base product including all the child products and selected options.
    discounts: [Discount] # final discount information for the base product including discounts on options and child products.
    selected_options: [String!] # selected options for the base product. for e.g color, size etc.
    entered_options: [KeyValue] # entered option for the base product. for e.g logo image etc.
}
```

The `ProductItemInterface` will be implemented by the following types:

```graphql
# Order Product implementation of OrderProductInterface
type OrderItem implements ProductItemInterface {
    ordered: Float! # number of items items
    shipped: Float! # number of shipped items
    refunded: Float! # number of refunded items
    invoiced: Float! # number of invoided items
    children: [OrderChildItem]
}

type OrderChildItem implements ProductItemInterface{
    ordered: Float! # number of items items
    shipped: Float! # number of shipped items
    refunded: Float! # number of refunded items
    invoiced: Float! # number of invoided items
}
```

### Payment Method Schema

To provide more customization for different payment solutions, the payment method will be represented by own type instead of simple string:

```graphql
#  payment method used to pay for the order
type PaymentMethod {
    name: String! # payment method name for e.g Braintree, Authorize etc
    type: String! # payment method type used to pay for the order for e.g Credit Card, PayPal etc.
    additonal_data: [KeyValue] # additional data per payment method type
}
```

### Pricing Schema

As entities like order, invoice, credit memo might have complex pricing type:

```graphql
interface OrderPricingInterface {
    subtotal: Float! # order subtotal excluding, shipping, discounts and tax
    discounts: [Discount] # all the discounts applied to the order
    shipping_handling: Float! # shipping and hnadling for the order
    tax: Float! # tax applied to the order
    grand_total: Float! # final order total including shipping and taxes
}
​
type OrderPricing implements OrderPricingInterface {
​
}
```

## Invoice Type Schema

The invoice entity will have the similar to the order schema:

```graphql
type Invoice {
    number: String! # user friendly identifier for the invoice
    pricing: OrderPricingInterface! # invoice pricing details
    items: [InvoiceItem]! # invoiced product details
}

type InvoiceItem implements ProductItemInterface{
    invoiced: Float! # number of invoided items
    children: [InvoiceChildItem]
}

type InvoiceChildItem implements ProductItemInterface{
    invoiced: Float! # number of invoided items
}

type InvoicePricing implements OrderPricingInterface {
  
}
```

## Refund Type Schema

The credit memo entity will have the similar to the order and invoice schema:

```graphql
type CreditMemo {
    number: String!
    items: [CreditMemoItem]! # items refunded
    pricing: OrderPricingInterface! # refund pricing details
}

type CreditMemoItem implements ProductItemInterface{
    refunded: Float! # number of refunded items
    children: [CreditMemoChildItem]
}
type CreditMemoChildItem implements ProductItemInterface{
    refunded: Float! # number of refunded items
}


type CreditMemoPricing implements OrderPricingInterface {

}
```

## Shipment Type Schema

```graphql
type OrderShipment {
    shipping_method: String! # shipping method for the order
    shipping_address: CustomerAddress! # shipping address for the order
    tracking_link: String # tracking link for the order
    shipped_items: [ShipmentItem] # items included in the shipment
}
 
type ShipmentItem implements ProductItemInterface{
    shipped: Float! #number of shipped items
    children: [ShipmentChildItem]
}

type ShipmentChildItem implements ProductItemInterface{
    shipped: Float! # umber of shipped items
}
```

## Additional Types

The `KeyValue` type will provide a possibility to use key-value pairs:

```graphql
type KeyValue {
    name: String # name part of the name/value pair
    value: String # value part of the name/value pair
}
```