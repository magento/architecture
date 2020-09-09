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
    number: FilterStringTypeInput @doc("Order number. Allows to filter orders by fully or partial entered number")
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

@doc( "Defines a filter for an input string.")
input FilterStringTypeInput  {
    in: [String] @doc("Filters items that are exactly the same as entries specified in an array of strings.")
    eq: String @doc("Filters items that are exactly the same as the specified string.")
    match: String @doc("Defines a filter that performs a fuzzy search using the specified string.")
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
    items: [OrderItemInterface] @doc("collection of all the items purchased")
    total: OrderTotal @doc("total amount details for the order")
    invoices: [Invoice] @doc("invoice list for the order")
    credit_memos: [CreditMemo] @doc("credit memo list for the order")
    shipments: [OrderShipment] @doc("shipment list for the order")
    payment_methods: [OrderPaymentMethod] @doc("payment details for the order")
    shipping_address: OrderAddress @doc("shipping address for the order")
    billing_address: OrderAddress @doc("billing address for the order")
    carrier: String @doc("shipping carrier for the order delivery")
    shipping_method: String @doc("shipping method for the order")
    comments: [SalesCommentItem] @doc("comments on the order")
}
```

The `id` will be a `base64_encode(increment_id)` which in future can be replaced by UUID.

> The order `status` should be filtered in the same way as for Luma via `Order Status` and `Visible On Storefront` configuration 

### Order Item

```graphql
interface OrderItemInterface @doc("Order item details") {
    id: ID! @doc("Order item unique identifier") #base64encode(orderItemId)
    product_name: String @doc("name of the base product")
    product_sku: String! @doc("SKU of the base product")
    product_url_key: String @doc("URL key of the base product")
    product_type: String @doc("Type of product (e.g. simple, configurable, bundle)")
    status: String @doc("the status of order item")
    product_sale_price: Money! @doc("sale price for the base product including selected options")
    discounts: [Discount] @doc("final discount information for the base product including discounts on options")
    selected_options: [OrderItemOption] @doc("selected options for the base product. for e.g color, size etc.")
    entered_options: [OrderItemOption] @doc("entered option for the base product. for e.g logo image etc.")
    quantity_ordered: Float @doc("number of items")
    quantity_shipped: Float @doc("number of shipped items")
    quantity_refunded: Float @doc("number of refunded items")
    quantity_invoiced: Float @doc("number of invoiced items")
    quantity_canceled: Float @doc("number of cancelled items")
    quantity_returned: Float @doc("number of returned items")
}

type OrderItem implements OrderItemInterface {
}

type BundleOrderItem implements OrderItemInterface {
    bundle_options: [ItemSelectedBundleOption] @doc("A list of bundle options that are assigned to the bundle product")
}

type ItemSelectedBundleOption {
    id: ID! @doc(description: "The unique identifier of the option")
    label: String! @doc(description: "The label of the option")
    values: [ItemSelectedBundleOptionValue] @doc(description: "A list of products that represent the values of the parent option")
}

type ItemSelectedBundleOptionValue {
    id: ID! @doc("unique identifier of option value")
    product_name: String! @doc("product name for option value")
    product_sku: String! @doc("product sku for option value")
    quantity: Float! @doc("quantitity of value selected")
    price: Money! @doc("Option value price. price for single quantity")
}

type DownloadableOrderItem implements OrderItemInterface {
    downloadable_links: [DownloadableItemsLinks] @doc(description: "A list of downloadable links that are ordered from the downloadable product")
}

type DownloadableItemsLinks @doc(description: "DownloadableProductLinks defines characteristics of a downloadable product") {
    title: String @doc(description: "The display name of the link")
    sort_order: Int @doc(description: "A number indicating the sort order")
    uid: ID! @doc(description: "A string that encodes option details.")
}

type GiftCardOrderItem implements OrderItemInterface {
    gift_card: GiftCardItem @doc(description: "Selected gift card properties for an order item")
}
type GiftCardItem {
    sender_name: String @doc(description: "Entered gift card sender name")
    sender_email: String @doc(description: "Entered gift card sender email")
    recipient_name: String @doc(description: "Entered gift card recipient name")
    recipient_email: String @doc(description: "Entered gift card recipient email")
    message: String @doc(description: "Entered gift card message intended for the recipient")
}

@doc("Represents order item options like selected or entered")
type OrderItemOption {
    label: String! @doc("name of the option")
    value: String! @doc("value of the option")
}
```
### Payment Method Schema

To provide more customization for different payment solutions, the payment method will be represented by own type instead of simple string:

```graphql
@doc("Payment method used to pay for the order")
type OrderPaymentMethod {
    name: String! @doc("payment method name for e.g Braintree, Authorize etc.")
    type: String! @doc("payment method type used to pay for the order for e.g Credit Card, PayPal etc.")
    additional_data: [KeyValue] @doc("additional data per payment method type")
}
```

> The payment `additional_data` should be filtered in the same way as for Luma via `privateInfoKeys` and `paymentInfoKeys` to not expose sensitive information.

### Amounts Schema

As entities like order, invoice, credit memo might have complex amounts type:

```graphql
@doc("Order total amounts details")
type OrderTotal {
    subtotal: Money! @doc("subtotal amount excluding, shipping, discounts and tax")
    discounts: [Discount] @doc("applied discounts")
    total_tax: Money! @doc("total tax amount")
    taxes: [TaxItem] @doc("order taxes details")
    grand_total: Money! @doc("final total amount including shipping and taxes")
    base_grand_total: Money! @doc("final total amount in base currency")
    total_shipping: Money! @doc("order shipping amount")
    shipping_handling: ShippingHandling @doc("shipping and handling for the order")
}

@doc("Shipping handling details")
type ShippingHandling {
    total_amount: Money! @doc("shipping total amount")
    amount_including_tax: Money @doc("shipping amount including tax")
    amount_excluding_tax: Money @doc("shipping amount excluding tax")
    taxes: [TaxItem] @doc("shipping taxes details")
    discounts: [ShippingDiscount] @doc("The applied discounts to the shipping)
}

type ShippingDiscount @doc(description:"Defines an individual shipping discount. This discount can be applied to shipping.") {
    amount: Money! @doc(description:"The amount of the discount")
}

@doc("Tax item details")
type TaxItem {
    amount: Money! @doc("tax amount")
    title: String! @doc("tax item title")
    rate: Float @doc("tax item rate")
}
```

## Invoice Type Schema

The invoice entity will have the similar to the order schema:
The `id` will be a `base64_encode(increment_id)` which in future can be replaced by UUID.
```graphql
@doc("Invoice details")
type Invoice {
    id: ID! @doc("the ID of the invoice, used for API purposes")
    number: String! @doc("sequential invoice number")
    total: InvoiceTotal @doc("invoice total amount details")
    items: [InvoiceItemInterface] @doc("invoiced product details")
    comments: [SalesCommentItem] @doc("comments on the invoice")
}

@doc("Invoice item details")
interface InvoiceItemInterface {
    id: ID! @doc("invoice item unique identifier") #base64encode(invoiceItemId)
    order_item: OrderItemInterface @doc("associated order item")
    product_name: String @doc("name of the base product")
    product_sku: String! @doc("SKU of the base product")
    product_sale_price: Money! @doc("sale price for the base product including selected options")
    discounts: [Discount] @doc("final discount information for the base product including discounts on options")
    quantity_invoiced: Float @doc("number of invoiced items")
}

type InvoiceItem implements InvoiceItemInterface {
}

type BundleInvoiceItem implements InvoiceItemInterface {
    bundle_options: [ItemSelectedBundleOption] @doc("A list of bundle options that are assigned to the bundle product")
}

type DownloadableInvoiceItem implements InvoiceItemInterface {
    downloadable_links: [DownloadableItemsLinks] @doc(description: "A list of downloadable links that are invoiced from the downloadable product")
}

type GiftCardInvoiceItem implements InvoiceItemInterface {
    gift_card: GiftCardItem @doc(description: "Selected gift card properties for an invoice item")
}

@doc("Invoice total amount details")
type InvoiceTotal {
    subtotal: Money! @doc("subtotal amount excluding, shipping, discounts and tax")
    discounts: [Discount] @doc("applied discounts")
    total_tax: Money! @doc("total tax amount")
    taxes: [TaxItem] @doc("order taxes details")
    grand_total: Money! @doc("final total amount including shipping and taxes")
    base_grand_total: Money! @doc("final total amount in base currency")
    total_shipping: Money! @doc("order shipping amount")
    shipping_handling: ShippingHandling @doc("shipping and handling for the order")
}
```

## Refund Type Schema

The credit memo entity will have the similar to the order and invoice schema:
The `id` will be a `base64encode(increment_id)` which in future can be replaced by UUID.

```graphql
@doc("Credit memo details")
type CreditMemo {
    id: ID! @doc("the ID of the credit memo, used for API purposes")
    number: String! @doc("sequential credit memo number")
    items: [CreditMemoItemInterface] @doc("items refunded")
    total: CreditMemoTotal @doc("refund total amount details")
    comments: [SalesCommentItem] @doc("comments on the credit memo")
}

@doc("Credit memo item details")
interface CreditMemoItemInterface {
    id: ID! @doc("Credit memo item unique identifier") #base64encode(creditMemoItemId)
    order_item: OrderItemInterface @doc("associated order item")
    product_name: String @doc("name of the base product")
    product_sku: String! @doc("SKU of the base product")
    product_sale_price: Money! @doc("sale price for the base product including selected options")
    discounts: [Discount] @doc("final discount information for the base product including discounts on options")
    quantity_invoiced: Float @doc("number of invoiced items")
}

type CreditMemoItem implements CreditMemoItemInterface {
}

type BundleCreditMemoItem implements CreditMemoIntemInterface {
    bundle_options: [ItemSelectedBundleOption]
}

type DownloadableCreditMemoItem implements CreditMemoItemInterface {
    downloadable_links: [DownloadableItemsLinks] @doc(description: "A list of downloadable links that are refunded from the downloadable product")
}

type GiftCardCreditMemoItem implements CreditMemoItemInterface {
    gift_card: GiftCardItem @doc(description: "Selected gift card properties for a credit memo item")
}

@doc("Credit memo price details")
type CreditMemoTotal {
    subtotal: Money! @doc("subtotal amount excluding, shipping, discounts and tax")
    discounts: [Discount] @doc("applied discounts")
    total_tax: Money! @doc("total tax amount")
    taxes: [TaxItem] @doc("order taxes details")
    grand_total: Money! @doc("final total amount including shipping and taxes")
    base_grand_total: Money! @doc("final total amount in base currency")
}
```

## Shipment Type Schema
The `id` will be a `base64_encode(increment_id)` which in future can be replaced by UUID.
```graphql
@doc("Order shipment details")
type OrderShipment {
    id: ID! @doc("the ID of the shipment, used for API purposes")
    number: String! @doc("sequential credit shipment number")
    tracking: [ShipmentTracking] @doc("shipment tracking details")
    items: [ShipmentItemInterface] @doc("items included in the shipment")
    comments: [SalesCommentItem] @doc("comments on the shipment")
}

@doc("Order shipment item details")
interface ShipmentItemInterface {
    id: ID! @doc("Shipment item unique identifier") #base64encode(shipmentItemId)
    order_item: OrderItemInterface @doc("associated order item")
    product_name: String @doc("name of the base product")
    product_sku: String! @doc("SKU of the base product")
    product_sale_price: Money! @doc("sale price for the base product")
    quantity_shipped: Float! @doc("number of shipped items")
}

type ShipmentItem implements ShipmentItemInterface {
}

type BundleShipmentItem implements ShipmentItemInterface {
    bundle_options: [ItemSelectedBundleOption]
}

type GiftCardShipmentItem implements ShipmentItemInterface {
    gift_card: GiftCardItem @doc(description: "Selected gift card properties for a shipment item")
}

@doc("Order shipment tracking details")
type ShipmentTracking {
    title: String! @doc("shipment tracking title")
    carrier: String! @doc("shipping carrier for the order delivery")
    number: String @doc("tracking number of the order shipment")
}
```

## SalesCommentItem type
```graphql
type SalesCommentItem {
    timestamp: String! @doc("The timestamp of the comment")
    message: String! @doc("the comment message")
}
```

## OrderAddress type
```graphql

type OrderAddress @doc(description: "OrderAddress contains detailed information about the order billing and shipping addresses"){
    firstname: String! @doc(description: "The first name of the person associated with the shipping/billing address")
    lastname: String! @doc(description: "The family name of the person associated with the shipping/billing address")
    middlename: String @doc(description: "The middle name of the person associated with the shipping/billing address")
    region: String @doc(description: "The state or province name")
    region_id: ID @doc(description: "The unique ID for a pre-defined region")
    country_code: CountryCodeEnum @doc(description: "The customer's order country")
    street: [String!] @doc(description: "An array of strings that define the street number and name")
    company: String @doc(description: "The customer's order company")
    telephone: String! @doc(description: "The telephone number")
    fax: String @doc(description: "The fax number")
    postcode: String @doc(description: "The customer's order ZIP or postal code")
    city: String! @doc(description: "The city or town")
    prefix: String @doc(description: "An honorific, such as Dr., Mr., or Mrs.")
    suffix: String @doc(description: "A value such as Sr., Jr., or III")
    vat_id: String @doc(description: "The customer's Value-added tax (VAT) number (for corporate customers)")
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
