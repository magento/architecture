# Customer Orders API

## Overview

The GraphQL API should provide a possibility to retrieve orders, shipments, invoices, credit memos for the logged in customer. The current schema allows fetching only simple order details and doesn't provide a possibility to fetch order details by a number.

The proposed solution is deprecation of `customerOrders` query and `orders` field to the `customer` query:

```graphql
# query to return a list of all customer orders
type Customer {
    orders (
        filter: CustomerOrderFilterInput
        currentPage: Int = 1 # current page of the customer order list. default is 1.
        pageSize: Int = 20 # page size for the customer orders list. default is 20.
    ): CustomerOrders
}

input CustomerOrderFilterInput {
    order_number: String!
}

# collection of customer orders.
type CustomerOrders {
    items: [Order]! # collection of customer orders that contains individual order details.
    page_info: SearchResultPageInfo
    total_count: Int
}
```

> Right now, we don't introduce the filter input for such entities like invoice, credit memo, shipment as all these entities are related to the same order and GraphQL resolvers anyway at first resolve the order and after that other child entities. But, in future, such filter input can be introduced as optional argument.

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

### Order Item

The order items will be presented as separate interface which will have multiple implementations for invoice, shipment and credit memo types.

```graphql
interface SalesProductInterface {
    name: String # name of the base product.
    sku: String! # sku of the base product.
    url: String # url of the base product.
    final_price: Float! # final price for the base product including all the child products and selected options.
    discounts: [Discount] # final discount information for the base product including discounts on options and child products.
    selected_options: [String!] # selected options for the base product. for e.g color, size etc.
    entered_options: [KeyValue] # entered option for the base product. for e.g logo image etc.
}
```

The `SalesProductInterface` will be implemented by the following types:

```graphql
# Order Product implementation of OrderProductInterface
type OrderItem implements SalesProductInterface {
    ordered: Float! # number of items items
    shipped: Float! # number of shipped items
    refunded: Float! # number of refunded items
    invoiced: Float! # number of invoiced items
    children: [OrderChildItem]
}

type OrderChildItem implements SalesProductInterface{
    ordered: Float! # number of items items
    shipped: Float! # number of shipped items
    refunded: Float! # number of refunded items
    invoiced: Float! # number of invoiced items
}
```

### Payment Method Schema

To provide more customization for different payment solutions, the payment method will be represented by own type instead of simple string:

```graphql
#  payment method used to pay for the order
type PaymentMethod {
    name: String! # payment method name for e.g Braintree, Authorize etc
    type: String! # payment method type used to pay for the order for e.g Credit Card, PayPal etc.
    additional_data: [KeyValue] # additional data per payment method type
}
```

### Pricing Schema

As entities like order, invoice, credit memo might have complex pricing type:

```graphql
interface OrderPricingInterface {
    subtotal: Float! # order subtotal excluding, shipping, discounts and tax
    discounts: [Discount] # all the discounts applied to the order
    tax: Float! # tax applied to the order
    grand_total: Float! # final order total including shipping and taxes
}
​
type OrderPricing implements OrderPricingInterface {
​   shipping_handling: Float! # shipping and handling for the order
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

type InvoiceItem implements SalesProductInterface{
    invoiced: Float! # number of invoiced items
    children: [InvoiceChildItem]
}

type InvoiceChildItem implements SalesProductInterface{
    invoiced: Float! # number of invoiced items
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

type CreditMemoItem implements SalesProductInterface{
    refunded: Float! # number of refunded items
    children: [CreditMemoChildItem]
}
type CreditMemoChildItem implements SalesProductInterface{
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
 
type ShipmentItem implements SalesProductInterface{
    shipped: Float! #number of shipped items
    children: [ShipmentChildItem]
}

type ShipmentChildItem implements SalesProductInterface{
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