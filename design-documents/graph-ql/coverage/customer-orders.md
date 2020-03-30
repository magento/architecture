# Customer Orders API

# Overview

The GraphQL API should provide a possibility to retrieve orders, shipments, invoices, credit memos for the logged in customer. The current schema allows fetching only simple order details and does not provide a possibility to fetch order details by a number.

The proposed solution is deprecation of `customerOrders` query and addition of `orders` field to the `customer` query:

```graphql
@doc("Query to return a list of all customer orders")
type Customer {
    orders (
        filter: CustomerOrdersFilterInput
        currentPage: Int = 1 @doc("current page of the customer order list. default is 1")
        pageSize: Int = 20 @doc("page size for the customer orders list. default is 20")
    ): CustomerOrders
}

@doc("Collection of customer orders")
type CustomerOrders {
    items: [CustomerOrder]! @doc("collection of customer orders that contains individual order details.")
    page_info: SearchResultPageInfo
    total_count: Int @doc("the total count of customer orders")
}
```

```graphql
@doc("Allows to extend the list of search criteria for customer orders")
input CustomerOrdersFilterInput {
    number: String @doc("Order number. Allows to filter orders by fully or partial entered number")
    status: String @("Order status")
    createdDate: FilterRangeTypeInput
    total: CustomerOrdersAmountFilterInput
    salesItem: SalesItemFilterInput
}

@doc("Provides order total amount search filter")
input CustomerOrdersAmountFilterInput {
    min: Float @doc("Minimum total order amount in store view currency")
    max: Float @doc("Maximum total order amount in store view currency")
}

@doc("Allows to extend the list of search criteria for order items")
input SalesItemFilterInput {
    name: String @doc("Order item name. Allows to filter orders by fully or partial entered order item name")
    sku: String @doc("Order item SKU. Allows to filter orders by fully or partial entered order item SKU")
}
```

> Right now, we don't introduce the filter input for such entities like invoice, credit memo, shipment as all these entities are related to the same order and GraphQL resolvers anyway at first resolve the order and after that other child entities. But, in future, such filter input can be introduced as optional argument.

## Order Type Schema

The proposed type for the customer order might look like:

```graphql
@doc("Customer order details")
type CustomerOrder {
    id: ID! @doc("the ID of the order, used for API purposes")
    order_date: String! @doc("date when the order was placed")
    status: String! @doc("current status of the order")
    number: String! @doc("sequential order number")
    items: [OrderItem]! @doc("collection of all the items purchased")
    total: OrderTotal! @doc("total amount details for the order")
    invoices: [Invoice]! @doc("invoice list for the order")
    credit_memos: [CreditMemo]! @doc("credit memo list for the order")
    shipments: [OrderShipment]! @doc("shipment list for the order")
    payment_methods: [PaymentMethod]! @doc("payment details for the order")
    shipping_address: CustomerAddress! @doc("shipping address for the order")
    billing_address: CustomerAddress! @doc("billing address for the order")
}
```

The `id` will be a `base64_encode(increment_id)` which in future can be replaced by UUID.

> The order `status` should be filtered in the same way as for Luma via `Order Status` and `Visible On Storefront` configuration 

### Order Item

The order items will be presented as separate interface which will have multiple implementations for invoice, shipment and credit memo types.

```graphql
@doc("Interface to reprent order/invoice/shipment/credit memo items")
interface SalesItemInterface {
    product_name: String @doc("name of the base product")
    product_sku: String! @doc("SKU of the base product")
    product_url: String @doc("URL of the base product")
    product_sale_price: Money! @doc("sale price for the base product including selected options")
    discounts: [Discount] @doc("final discount information for the base product including discounts on options")
    parent_product_name: String @doc("name of parent product like configurable or bundle")
    parent_product_sku: String @doc("SKU of parent product like configurable or bundle")
    parent_product_url: String @doc("URL of parent product in the catalog")
    selected_options: [SalesItemSelectedOption] @doc("selected options for the base product. for e.g color, size etc.")
    entered_options: [SalesItemEnteredOption] @doc("entered option for the base product. for e.g logo image etc.")
}

@doc("Represents sales item selected options")
type SalesItemSelectedOption {
    id: ID! @doc("ID of the option")
    label: String! @doc("name of the option")
    value_labels: [String]! @doc("list of option value labels")
}

@doc("Represents sales item entered options")
type SalesItemEnteredOption {
    id: ID! @doc("ID of the option")
    label: String! @doc("name of the option")
    value: String! @doc("value of the option")
}
```

The `id` will be a `base64_encode(option_id)` which in future can be replaced by UUID.

The `SalesItemInterface` will be implemented by the following types:

```graphql
@doc("Order Product implementation of OrderProductInterface")
type OrderItem implements SalesItemInterface {
    quantity_ordered: Float @doc("number of items")
    quantity_shipped: Float @doc("number of shipped items")
    quantity_refunded: Float @doc("number of refunded items")
    quantity_invoiced: Float @doc("number of invoiced items")
    quantity_canceled: Float @doc("number of cancelled items")
    quantity_returned: Float @doc("number of returned items")
    status: String @doc("the status of order item")
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

### Amounts Schema

As entities like order, invoice, credit memo might have complex amounts type:

```graphql
@doc("Interface to provide sales amounts")
interface SalesTotalAmountInterface {
    subtotal: Money! @doc("subtotal amount excluding, shipping, discounts and tax")
    discounts: [Discount] @doc("applied discounts")
    tax: Money! @doc("applied taxes")
    grand_total: Money! @doc("final total amount including shipping and taxes")
    base_grand_total: Money! @doc("final total amount in base currency")
}
​
@doc("Order total amounts details")
type OrderTotal implements SalesTotalAmountInterface {
​   shipping_handling: Money! @doc("shipping and handling for the order")
}
```

## Invoice Type Schema

The invoice entity will have the similar to the order schema:

```graphql
@doc("Invoice details")
type Invoice {
    id: ID! @doc("the ID of the invoice, used for API purposes")
    number: String! @doc("sequential invoice number")
    total: InvoiceTotal! @doc("invoice total amount details")
    items: [InvoiceItem]! @doc("invoiced product details")
}

@doc("Invoice item details")
type InvoiceItem implements SalesItemInterface{
    quantity_invoiced: Float! @doc("number of invoiced items")
}

@doc("Invoice total amount details")
type InvoiceTotal implements SalesTotalAmountInterface {
  
}
```

The `id` will be a `base64_encode(increment_id)` which in future can be replaced by UUID.

## Refund Type Schema

The credit memo entity will have the similar to the order and invoice schema:

```graphql
@doc("Credit memo details")
type CreditMemo {
    id: ID! @doc("the ID of the credit memo, used for API purposes")
    number: String! @doc("sequential credit memo number")
    items: [CreditMemoItem]! @doc("items refunded")
    total: CredtiMemoTotal! @doc("refund total amount details")
}

@doc("Credit memo item details")
type CreditMemoItem implements SalesItemInterface{
    quantity_refunded: Float! @doc("number of refunded items")
}

@doc("Credit memo price details")
type CredtiMemoTotal implements SalesTotalAmountInterface {

}
```

The `id` will be a `base64_encode_encode(increment_id)` which in future can be replaced by UUID.

## Shipment Type Schema

```graphql
@doc("Order shipment details")
type OrderShipment {
    id: ID! @doc("the ID of the shipment, used for API purposes")
    number: String! @doc("sequential credit shipment number")
    tracking: [ShipmentTracking] @doc("shipment tracking details")
    items: [ShipmentItem]! @doc("items included in the shipment")
}

@doc("Order shipment item details")
type ShipmentItem implements SalesItemInterface{
    quantity_shipped: Float! @doc("number of shipped items")
}

@doc("Order shipment tracking details")
type ShipmentTracking {
    method: String! @doc("shipping method for the order")
    carrier: String! @doc("shipping carrier for the order delivery")
    number: String @doc("tracking number of the order shipment")
    link: String @doc("tracking link of the order shipment")
}
```

The `id` will be a `base64_encode(increment_id)` which in future can be replaced by UUID.

## Additional Types

The `KeyValue` type will provide a possibility to use key-value pairs:

```graphql
@doc("The key-value type")
type KeyValue {
    name: String @doc("name part of the name/value pair")
    value: String @doc("value part of the name/value pair")
}
```
