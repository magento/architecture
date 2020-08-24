# ID Improvement Plan

We've recently agreed on some standardization to how object identifiers are represented in Magento GraphQL schemas, which will require some changes to existing code to reach the desired state.

This work _must not introduce any breaking changes_.

## Approved Proposals

- [Propose renaming id_v2 to something more permanent, and change type #396](https://github.com/magento/architecture/pull/396)
- [Add document with suggestions for ID fields #395](https://github.com/magento/architecture/pull/395)

Between these 2 proposals, agreement was reached on the following guidelines:

- Identifier fields and arguments _must_ use the `ID` scalar type
- Identifier fields in Object Types _must_ be non-nullable (`!`)
- Identifier fields in Object Types _must_ have the field name `uid`
- Arguments that accept a `uid` value (either `Query` or `Mutation`) _must_ have `uid` in the argument name_
- All objects representing entities that _can_ be addressed by ID _must_ have a `uid` field
- All `ID` values _must_ be unique to their type (no collisions on IDs)

## Terminology
- **Primary Identifier**: An ID owned by the current Object Type (ex: `Product.id`)
- **Foreign Identifier**: An ID referencing another Object Type (ex: `PaymentMethodInput.code`)

## Work that needs to get done

### Object Types and Interfaces with a _Primary Identifier_

I found 51 Objects/Interfaces in the Open-Source schema that will need these changes.

#### Changes needed
- Add a new, non-nullable field named `uid` with type `ID`
- Resolve the `uid` type to the same value as the object's existing _primary identifier_ field
- Deprecate the existing _primary identifier_ field, with a message directing developers to the `uid` field

#### Example Change
```diff
interface ProductInterface {
-    id: Int
+    id: Int @deprecated(reason: "Use the 'uid' field instead")
+    uid: ID!
}
```

### Fields with 1 or more _Foreign Identifier_ argument

I found 10 arguments across 8 fields in the Open-Source schema that will need these changes.

#### Changed Needed
- For each argument that's a _foreign identifier_
    - Add a new argument with a name of the format `{ForeignObject}_uid`, where `ForeignObject` is the name of the Object Type with the matching `uid` field
    - Deprecate the existing argument, with a message directing developers to the new argument
    - Update the resolver to use _either_ the new or old argument, but throw a validation error when both are used
    - The new argument must have the same nullability as the old argument

#### Example Change
```diff
type Query {
-   cart(cart_id: String!): Cart
+   cart(
+       cart_id: String! @deprecated(reason: "Use the 'cart_uid' argument instead")
+       cart_uid: ID! @doc(description: "ID from the Cart.uid field")
+    ): Cart
}
```

### Input Object Types with 1 or more _Foreign Identifier_ fields

I found 49 fields across 44 Input Object Types in the Open-Source schema that will need these changes.

#### Changes Needed
- For each field that's a _foreign identifier_
    - Add a new field with a name of the format `{ForeignObject}_uid`, where `ForeignObject` is the name of the Object Type with the matching `uid` field
    - Deprecate the existing field, with a message directing developers to the new field
    - Update all related resolvers to use _either_ the new or old field, but throw a validation error when both are used
    - The new field must have the same nullability as the old field

#### Example Change
```diff
input CartItemInput {
-   sku: String!
+   sku: String! @deprecated(reason: "Use the CartItemInput.product_uid field instead")
+   product_uid: ID! @doc(description: "ID from the ProductInterface.uid field")
}
```

## Suggested Grouping of Work

Ideally, whether this work is done all at once or incrementally, we'll make sure that changes are made in groups. That is, anytime a _primary identifier_ field changes on an Object/Interface, the same changeset should include changes to all places the matching _foreign identifier_ field/argument is used.

For example, when the `uid` field is added to the `Cart` type in the `Magento/QuoteGraphQl` module, all input objects and arguments that reference the old `cart_id` should be updated to include the new `cart_uid` field.

### Example Breakdown of Work
<details>
<summary>Click to Expand</summary>

| Object/Interface  | Primary Identifier Field | References                         | 
|-------------------|--------------------------|------------------------------------| 
| Cart              | id                       | Mutation.mergeCarts                | 
|                   |                          | Query.cart                         | 
|                   |                          | AddBundleProductsToCartInput       | 
|                   |                          | AddConfigurableProductsToCartInput | 
|                   |                          | AddDownloadableProductsToCartInput | 
|                   |                          | AddSimpleProductsToCartInput       | 
|                   |                          | AddVirtualProductsToCartInput      | 
|                   |                          | ApplyCouponToCartInput             | 
|                   |                          | ApplyGiftCardToCartInput           | 
|                   |                          | ApplyStoreCreditToCartInput        | 
|                   |                          | createEmptyCartInput               | 
|                   |                          | HostedProUrlInput                  | 
|                   |                          | PayflowLinkTokenInput              | 
|                   |                          | PayflowProResponseInput            | 
|                   |                          | PayflowProTokenInput               | 
|                   |                          | PaypalExpressTokenInput            | 
|                   |                          | PlaceOrderInput                    | 
|                   |                          | RemoveCouponFromCartInput          | 
|                   |                          | RemoveGiftCardFromCartInput        | 
|                   |                          | RemoveItemFromCartInput            | 
|                   |                          | RemoveStoreCreditFromCartInput     | 
|                   |                          | SetBillingAddressOnCartInput       | 
|                   |                          | SetGuestEmailOnCartInput           | 
|                   |                          | SetPaymentMethodAndPlaceOrderInput | 
|                   |                          | SetPaymentMethodOnCartInput        | 
|                   |                          | SetShippingAddressesOnCartInput    | 
|                   |                          | SetShippingMethodsOnCartInput      | 
|                   |                          | UpdateCartItemsInput               | 
|                   |                          |                                    | 
| ProductInterface  | id                       | CartItemInput                      | 
|                   |                          | ProductSortInput                   | 
|                   |                          | SendEmailToFriendInput             | 
|                   |                          | ConfigurableProductCartItemInput   | 
|                   |                          | ProductFilterInput                 | 
|                   |                          | BundleProduct                      | 
|                   |                          | ConfigurableProduct                | 
|                   |                          | DownloadableProduct                | 
|                   |                          | GiftCardProduct                    | 
|                   |                          | GroupedProduct                     | 
|                   |                          | SimpleProduct                      | 
|                   |                          | VirtualProduct                     | 
|                   |                          | ProductAttributeFilterInput        | 
|                   |                          | ConfigurableProductOptions         | 
|                   |                          | CustomizableAreaOption             | 
|                   |                          | BundleItem                         | 
|                   |                          | CustomizableAreaValue              | 
|                   |                          | CustomizableCheckboxValue          | 
|                   |                          | CustomizableDateOption             | 
|                   |                          | CustomizableDateValue              | 
|                   |                          | CustomizableDropDownValue          | 
|                   |                          | CustomizableFieldOption            | 
|                   |                          | CustomizableFieldValue             | 
|                   |                          | CustomizableFileOption             | 
|                   |                          | CustomizableFileValue              | 
|                   |                          | CustomizableMultipleValue          | 
|                   |                          | CustomizableRadioValue             | 
|                   |                          | ProductLinks                       | 
|                   |                          | ProductLinksInterface              | 
|                   |                          |                                    | 
|                   |                          |                                    | 
| CategoryInterface | id                       | CategoryTree                       | 
|                   |                          | ProductFilterInput                 | 
|                   |                          | CategoryFilterInput                | 
|                   |                          | StoreConfig                        | 
|                   |                          | Breadcrumb                         | 
|                   |                          | ProductAttributeFilterInput        | 
|                   |                          |                                    | 
| CartItemInterface | id                       | ConfigurableCartItem               | 
|                   |                          | DownloadableCartItem               | 
|                   |                          | SimpleCartItem                     | 
|                   |                          | VirtualCartItem                    | 
|                   |                          | BundleCartItem                     | 
|                   |                          | CartItemUpdateInput                | 
|                   |                          | RemoveItemFromCartInput            | 

</details>


## Breaking Change Risks

The following changes need to be avoided to ensure we're not breaking backwards compatibility:
- The nullability of existing fields/arguments _must not change_
- The return type of existing fields _must not change_
- Existing fields/arguments _must not be removed_

## Current State Breakdown (Open-Source)
<details>
<summary>Click to Expand</summary>

- 51 Object Types and Interfaces with a _primary identifier_
    | Primary Identifier Field Name | Count | 
    |-------------------------------|-------| 
    | id                            | 34    | 
    | option_id                     | 10    | 
    | option_type_id                | 4     | 
    | order_number                  | 1     | 
    | value_id                      | 1     | 
    | agreement_id                  | 1     | 

- 10 arguments across 8 different fields take a _foreign identifier_
    | Foreign Identifier Argument Name | Count | 
    |----------------------------------|-------| 
    | id                               | 4     | 
    | cart_id                          | 1     | 
    | identifier                       | 1     | 
    | identifiers                      | 1     | 
    | source_cart_id                   | 1     | 
    | destination_cart_id              | 1     | 
    | orderNumber                      | 1     |

- 49 Input Object fields across 44 different Input Objects take a _foreign identifier_
    | Foreign Identifier Field Name | Count | 
    |-------------------------------|-------| 
    | cart_id                       | 26    | 
    | sku                           | 3     | 
    | attribute_code                | 2     | 
    | cart_item_id                  | 2     | 
    | code                          | 2     | 
    | customer_address_id           | 2     | 
    | carrier_code                  | 1     | 
    | category_id                   | 1     | 
    | link_id                       | 1     | 
    | method_code                   | 1     | 
    | parent_sku                    | 1     | 
    | product_id                    | 1     | 
    | region_code                   | 1     | 
    | variant_sku                   | 1     | 

</details>