# Customer Orders API

## Overview

The GraphQL API should provide a possibility to retrieve orders, shipments, invoices, credit memos for the logged in customer. The current schema allows fetching only simple order details and doesn't provide a possibility to fetch order details by a number.

The proposed solution is deprecation of `customerOrders` query and `orders` field to the `customer` query:

```graphql
# query to return a list of all customer orders
type Customer {
    orders (
        filter: CustomerOrdersFilterInput
        currentPage: Int = 1 # current page of the customer order list. default is 1.
        pageSize: Int = 20 # page size for the customer orders list. default is 20.
    ): CustomerOrders
}

input CustomerOrdersFilterInput {
    order_number: String!
}

# collection of customer orders.
type CustomerOrders {
    items: [CustomerOrder]! # collection of customer orders that contains individual order details.
    page_info: SearchResultPageInfo
    total_count: Int
}
```

> Right now, we don't introduce the filter input for such entities like invoice, credit memo, shipment as all these entities are related to the same order and GraphQL resolvers anyway at first resolve the order and after that other child entities. But, in future, such filter input can be introduced as optional argument.

## Order Type Schema

The proposed type for the customer order might look like:

```graphql
# order details
type CustomerOrder {
    order_date: String! # date when the order was placed.
    status: String! # current status of the order.
    order_number: String! # order number.
    items: [OrderItem]! # collection of all the items purchased
    prices: SalesPricesInterface! # prices details for the order.
    invoices: [Invoice]! # invoice list for the order.
    credit_memos: [CreditMemo]! # credit memo list for the order.
    shipments: [OrderShipment]! # shipping list for the order.
    payment_methods: [PaymentMethod]! # payment details for the order.
}
```

> The order `status` should be filtered in the same way as for Luma via `Order Status` and `Visible On Storefront` configuration 

### Order Item

The order items will be presented as separate interface which will have multiple implementations for invoice, shipment and credit memo types.

```graphql
interface SalesItemInterface {
    name: String # name of the base product.
    sku: String! # sku of the base product.
    url: String # url of the base product.
    sale_price: Float! # sale price for the base product including all the child products and selected options.
    discounts: [Discount] # final discount information for the base product including discounts on options and child products.
    selected_options: [String!] # selected options for the base product. for e.g color, size etc.
    entered_options: [SalesItemOption] # entered option for the base product. for e.g logo image etc.
}

type SalesItemOption {
    id: String!
    value: String!
}
```

The `SalesItemInterface` will be implemented by the following types:

```graphql
# Order Product implementation of OrderProductInterface
type OrderItem implements SalesItemInterface {
    quantity_ordered: Float! # number of items
    quantity_shipped: Float! # number of shipped items
    quantity_refunded: Float! # number of refunded items
    quantity_invoiced: Float! # number of invoiced items
    quantity_backordered: Float! # number of back ordered items
    quantity_canceled: Float! # number of cancelled items
    quantity_returned: Float! # number of returned items
    status: String! # the status of order item
    children: [OrderChildItem]
}

type OrderChildItem implements SalesItemInterface{
    quantity_ordered: Float! # number of items items
    quantity_shipped: Float! # number of shipped items
    quantity_refunded: Float! # number of refunded items
    quantity_invoiced: Float! # number of invoiced items
    quantity_backordered: Float! # number of back ordered items
    quantity_canceled: Float! # number of cancelled items
    quantity_returned: Float! # number of returned items
    status: String! # the status of order item
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

> The payment `additional_data` should be filtered in the same way as for Luma via `privateInfoKeys` and `paymentInfoKeys` to not expose sensitive information.

### Prices Schema

As entities like order, invoice, credit memo might have complex prices type:

```graphql
interface SalesPricesInterface {
    subtotal: Float! # subtotal amount excluding, shipping, discounts and tax
    discounts: [Discount] # applied discounts
    tax: Float! # applied taxes
    grand_total: Float! # final total amount including shipping and taxes
    base_grand_total: Float! # final total amount in base currency
}
​
type OrderPrices implements SalesPricesInterface {
​   shipping_handling: Float! # shipping and handling for the order
}
```

## Invoice Type Schema

The invoice entity will have the similar to the order schema:

```graphql
type Invoice {
    number: String! # user friendly identifier for the invoice
    prices: InvoicePrices! # invoice prices details
    items: [InvoiceItem]! # invoiced product details
}

type InvoiceItem implements SalesItemInterface{
    quantity_invoiced: Float! # number of invoiced items
    children: [InvoiceChildItem]
}

type InvoiceChildItem implements SalesItemInterface{
    quantity_invoiced: Float! # number of invoiced items
}

type InvoicePrices implements SalesPricesInterface {
  
}
```

## Refund Type Schema

The credit memo entity will have the similar to the order and invoice schema:

```graphql
type CreditMemo {
    number: String!
    items: [CreditMemoItem]! # items refunded
    prices: CreditMemoPrices! # refund prices details
}

type CreditMemoItem implements SalesItemInterface{
    quantity_refunded: Float! # number of refunded items
    children: [CreditMemoChildItem]
}
type CreditMemoChildItem implements SalesItemInterface{
    refunded: Float! # number of refunded items
}

type CreditMemoPrices implements SalesPricesInterface {

}
```

## Shipment Type Schema

```graphql
type OrderShipment {
    shipping_method: String! # shipping method for the order
    shipping_address: CustomerAddress! # shipping address for the order
    tracking_link: String # tracking link for the order
    shipped_items: [ShipmentItem]! # items included in the shipment
}
 
type ShipmentItem implements SalesItemInterface{
    quantity_shipped: Float! #number of shipped items
    children: [ShipmentChildItem]
}

type ShipmentChildItem implements SalesItemInterface{
    quantity_shipped: Float! # umber of shipped items
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