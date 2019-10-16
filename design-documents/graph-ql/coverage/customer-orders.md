# Customer Orders API

# Overview

The GraphQL API should provide a possibility to retrieve orders, shipments, invoices, credit memos for the logged in customer. The current schema allows fetching only simple order details and doesn't provide a possibility to fetch order details by a number.

The proposed solution is deprecation of `customerOrders` query and `orders` field to the `customer` query:

```graphql
@doc("Query to return a list of all customer orders")
type Customer {
    orders (
        filter: CustomerOrdersFilterInput
        currentPage: Int = 1 @doc("current page of the customer order list. default is 1")
        pageSize: Int = 20 @doc("page size for the customer orders list. default is 20")
    ): CustomerOrders
}

@doc("Allows to extend the list of search criteria for customer orders")
input CustomerOrdersFilterInput {
    order_number: String!
}

@doc("Collection of customer orders")
type CustomerOrders {
    items: [CustomerOrder]! @doc("collection of customer orders that contains individual order details.")
    page_info: SearchResultPageInfo
    total_count: Int @doc("the total count of customer orders")
}
```

> Right now, we don't introduce the filter input for such entities like invoice, credit memo, shipment as all these entities are related to the same order and GraphQL resolvers anyway at first resolve the order and after that other child entities. But, in future, such filter input can be introduced as optional argument.

## Order Type Schema

The proposed type for the customer order might look like:

```graphql
@doc("Customer order details")
type CustomerOrder {
    order_date: String! @doc("date when the order was placed")
    status: String! @doc("current status of the order")
    order_number: String! @doc("order number")
    items: [OrderItem]! @doc("collection of all the items purchased")
    prices: SalesPricesInterface! @doc("prices details for the order")
    invoices: [Invoice]! @doc("invoice list for the order")
    credit_memos: [CreditMemo]! @doc("credit memo list for the order")
    shipments: [OrderShipment]! @doc("shipment list for the order")
    payment_methods: [PaymentMethod]! @doc("payment details for the order")
}
```

> The order `status` should be filtered in the same way as for Luma via `Order Status` and `Visible On Storefront` configuration 

### Order Item

The order items will be presented as separate interface which will have multiple implementations for invoice, shipment and credit memo types.

```graphql
@doc("Interface to reprent order/invoice/shipment/credit memo items")
interface SalesItemInterface {
    name: String @doc("name of the base product")
    sku: String! @doc("sku of the base product")
    url: String @doc("url of the base product")
    sale_price: Float! @doc("sale price for the base product including selected options")
    discounts: [Discount] @doc("final discount information for the base product including discounts on options")
    parent_name: String @doc("name of parent product like configurable or bundle")
    parent_sku: String @doc("SKU of parent product like configurable or bundle")
    parent_url: String @doc("URL of parent product in the catalog")
    selected_options: [SalesItemOption] @doc("selected options for the base product. for e.g color, size etc.")
    entered_options: [SalesItemOption] @doc("entered option for the base product. for e.g logo image etc.")
}

@doc("Represents sales item options like selected or entered")
type SalesItemOption {
    id: String! @doc("name of the option")
    value: String! @doc("value of the option")
}
```

The `SalesItemInterface` will be implemented by the following types:

```graphql
@doc("Order Product implementation of OrderProductInterface")
type OrderItem implements SalesItemInterface {
    quantity_ordered: Float! @doc("number of items")
    quantity_shipped: Float! @doc("number of shipped items")
    quantity_refunded: Float! @doc("number of refunded items")
    quantity_invoiced: Float! @doc("number of invoiced items")
    quantity_backordered: Float! @doc("number of back ordered items")
    quantity_canceled: Float! @doc("number of cancelled items")
    quantity_returned: Float! @doc("number of returned items")
    status: String! @doc("the status of order item")
}
```

### Payment Method Schema

To provide more customization for different payment solutions, the payment method will be represented by own type instead of simple string:

```graphql
@doc("Payment method used to pay for the order")
type PaymentMethod {
    name: String! @doc("payment method name for e.g Braintree, Authorize etc.")
    type: String! @doc("payment method type used to pay for the order for e.g Credit Card, PayPal etc.")
    additional_data: [KeyValue] @doc("additional data per payment method type")
}
```

> The payment `additional_data` should be filtered in the same way as for Luma via `privateInfoKeys` and `paymentInfoKeys` to not expose sensitive information.

### Prices Schema

As entities like order, invoice, credit memo might have complex prices type:

```graphql
@doc("Interface to provide sales prices")
interface SalesPricesInterface {
    subtotal: Float! @doc("subtotal amount excluding, shipping, discounts and tax")
    discounts: [Discount] @doc("applied discounts")
    tax: Float! @doc("applied taxes")
    grand_total: Float! @doc("final total amount including shipping and taxes")
    base_grand_total: Float! @doc("final total amount in base currency")
}
​
@doc("Order prices details")
type OrderPrices implements SalesPricesInterface {
​   shipping_handling: Float! @doc("shipping and handling for the order")
}
```

## Invoice Type Schema

The invoice entity will have the similar to the order schema:

```graphql
@doc("Invoice details")
type Invoice {
    number: String! @doc("user friendly identifier for the invoice")
    prices: InvoicePrices! @doc("invoice prices details")
    items: [InvoiceItem]! @doc("invoiced product details")
}

@doc("Invoice item details")
type InvoiceItem implements SalesItemInterface{
    quantity_invoiced: Float! @doc("number of invoiced items")
}

@doc("Invoice prices details")
type InvoicePrices implements SalesPricesInterface {
  
}
```

## Refund Type Schema

The credit memo entity will have the similar to the order and invoice schema:

```graphql
@doc("Credit memo details")
type CreditMemo {
    number: String!
    items: [CreditMemoItem]! @doc("items refunded")
    prices: CreditMemoPrices! @doc("refund prices details")
}

@doc("Credit memo item details")
type CreditMemoItem implements SalesItemInterface{
    quantity_refunded: Float! @doc("number of refunded items")
}

@doc("Credit memo price details")
type CreditMemoPrices implements SalesPricesInterface {

}
```

## Shipment Type Schema

```graphql
@doc("Order shipment details")
type OrderShipment {
    shipping_method: String! @doc("shipping method for the order")
    shipping_address: CustomerAddress! @doc("shipping address for the order")
    tracking_link: String @doc("tracking link for the order")
    shipped_items: [ShipmentItem]! @doc("items included in the shipment")
}

@doc("Order shipment item details")
type ShipmentItem implements SalesItemInterface{
    quantity_shipped: Float! @doc("number of shipped items")
}

```

## Additional Types

The `KeyValue` type will provide a possibility to use key-value pairs:

```graphql
@doc("The key-value type")
type KeyValue {
    name: String @doc("name part of the name/value pair")
    value: String @doc("value part of the name/value pair")
}
```