# Queries

Gift message is Open source functionality and should be implemented in scope of GiftMessageGraphQl module.
Gift wrapping is commerce functionality and should be covered in scope of GiftWrappingGraphQl module. 

It is suggested to use base64 encoded DB increment ID as ID in GraphQL.

## Data

```graphql

###### Begin: Extending existing types ######
type Cart {
    available_gift_wrappings: [GiftWrapping]! @doc(description: "The list of available gift wrapping options for the cart")
    gift_wrapping: GiftWrapping @doc(description: "The selected gift wrapping for the cart")
    printed_card_included: Boolean! @doc(description: "Wether customer requested printed card for the order")
    gift_receipt_included: Boolean! @doc(description: "Wether customer requested gift receipt for the order")
    gift_message: GiftMessage @doc(description: "The entered gift message for the cart")
}

type SimpleCartItem {
    available_gift_wrapping: [GiftWrapping]! @doc(description: "The list of available gift wrapping options for the cart item")
    gift_wrapping: GiftWrapping @doc(description: "The selected gift wrapping for the cart item")
    gift_message: GiftMessage @doc(description: "The entered gift message for the cart item")
}

type ConfigurableCartItem {
    available_gift_wrapping: [GiftWrapping]! @doc(description: "The list of available gift wrapping options for the cart item")
    gift_wrapping: GiftWrapping @doc(description: "The selected gift wrapping for the cart item")
    gift_message: GiftMessage @doc(description: "The entered gift message for the cart item")
}

type BundleCartItem {
    available_gift_wrapping: [GiftWrapping]! @doc(description: "The list of available gift wrapping options for the cart item")
    gift_wrapping: GiftWrapping @doc(description: "The selected gift wrapping for the cart item")
    gift_message: GiftMessage @doc(description: "The entered gift message for the cart item")
}

type GiftCardCartItem {
    gift_message: GiftMessage @doc(description: "The entered gift message for the cart item")
}

type SalesItemInterface {
    gift_message: GiftMessage @doc(description: "The entered gift message for the order item")
}

interface OrderItemInterface {
    gift_wrapping: GiftWrapping @resolver(class: "Magento\\GiftWrappingGraphQl\\Model\\Resolver\\Order\\Item\\GiftWrapping") @doc(description: "The selected gift wrapping for the order item")
}

type CustomerOrder {
    gift_wrapping: GiftWrapping @doc(description: "The selected gift wrapping for the order")
    printed_card_included: Boolean! @doc(description: "Whether customer requested printed card for the order")
    gift_receipt_included: Boolean! @doc(description: "Whether customer requested gift receipt for the order")
    gift_message: GiftMessage @doc(description: "The entered gift message for the order")
}

type CartPrices {
    gift_options: GiftOptionsPrices @doc(description: "The list of prices for the selected gift options")
}

###### End: Extending existing types ######


###### StoreConfig ######
type StoreConfig {
    allow_gift_wrapping_on_order: String @doc(description: "Allow Gift Wrapping on Order Level")
    allow_gift_wrapping_on_order_items: String @doc(description: "Allow Gift Wrapping for Order Items")
    allow_gift_receipt: String @doc(description: "Allow Gift Receipt")
    allow_printed_card: String @doc(description: "Allow Printed Card")
    printed_card_price: String @doc(description: "Default Price for Printed Card")
    cart_gift_wrapping: String @doc(description: "Display Gift Wrapping Prices")
    cart_printed_card: String @doc(description: "Display Printed Card Prices")
    sales_gift_wrapping: String @doc(description: "Display Gift Wrapping Prices")
    sales_printed_card: String @doc(description: "Display Printed Card Prices")
}
###### End ######




###### Begin: Defining new types ######

type GiftWrappingImage {
    label: String! @doc(description: "Gift wrapping preview image label")
    url: String! @doc(description: "Gift wrapping preview image URL")
}

type GiftWrapping {
    id: ID! @doc(description: "Gift wrapping unique identifier")
    design: String! @doc(description: "Gift wrapping design name")
    price: Money! @doc(description: "Gift wrapping price")
    image: GiftWrappingImage @doc(description: "Gift wrapping preview image")
}

type GiftOptionsPrices {
    gift_wrapping_for_order: Money @doc(description: "Price of the gift wrapping for the whole order")
    gift_wrapping_for_items: Money @doc(description: "Price of the gift wrapping for all individual order items")
    printed_card: Money @doc(description: "Price for the printed card")
}

type GiftMessage {
    to: String! @doc(description: "Recepient name")
    from: String! @doc(description: "Sender name")
    message: String! @doc(description: "Gift message text")
}
###### End: Defining new types ######

```

## Configuration

The following gift options need to be whitelisted in the `storeConfig` query. See [example](https://github.com/magento/magento2/blob/52b66acf17e049dc2c5c7d9e12bd6d29d6a1a16d/app/code/Magento/CatalogGraphQl/etc/graphql/di.xml#L96).

```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Config:etc/system_file.xsd">
    <system>
        <section id="sales">
            <group id="gift_options">
                <field id="allow_gift_wrapping_on_order" translate="label" type="select" sortOrder="10" showInDefault="1" showInWebsite="1" showInStore="0">
                    <label>Allow Gift Wrapping on Order Level</label>
                    <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
                </field>
                <field id="allow_gift_wrapping_on_order_items" translate="label" type="select" sortOrder="15" showInDefault="1" showInWebsite="1" showInStore="0">
                    <label>Allow Gift Wrapping for Order Items</label>
                    <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
                </field>
                <field id="allow_gift_receipt" translate="label" type="select" sortOrder="20" showInDefault="1" showInWebsite="1" showInStore="0">
                    <label>Allow Gift Receipt</label>
                    <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
                </field>
                <field id="allow_printed_card" translate="label" type="select" sortOrder="25" showInDefault="1" showInWebsite="1" showInStore="0">
                    <label>Allow Printed Card</label>
                    <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
                </field>
                <field id="printed_card_price" translate="label" type="text" sortOrder="30" showInDefault="1" showInWebsite="1" showInStore="0">
                    <label>Default Price for Printed Card</label>
                </field>
            </group>
        </section>
        <section id="tax">
            <group id="cart_display">
                <field id="gift_wrapping" translate="label" type="select" sortOrder="35" showInDefault="1" showInWebsite="1" showInStore="1" canRestore="1">
                    <label>Display Gift Wrapping Prices</label>
                    <source_model>Magento\GiftWrapping\Model\System\Config\Source\Display\Type</source_model>
                </field>
                <field id="printed_card" translate="label" type="select" sortOrder="40" showInDefault="1" showInWebsite="1" showInStore="1" canRestore="1">
                    <label>Display Printed Card Prices</label>
                    <source_model>Magento\GiftWrapping\Model\System\Config\Source\Display\Type</source_model>
                </field>
            </group>
            <group id="sales_display">
                <field id="gift_wrapping" translate="label" type="select" sortOrder="35" showInDefault="1" showInWebsite="1" showInStore="1" canRestore="1">
                    <label>Display Gift Wrapping Prices</label>
                    <source_model>Magento\GiftWrapping\Model\System\Config\Source\Display\Type</source_model>
                </field>
                <field id="printed_card" translate="label" type="select" sortOrder="40" showInDefault="1" showInWebsite="1" showInStore="1" canRestore="1">
                    <label>Display Printed Card Prices</label>
                    <source_model>Magento\GiftWrapping\Model\System\Config\Source\Display\Type</source_model>
                </field>
            </group>
        </section>
    </system>
</config>
```

```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Config:etc/system_file.xsd">
    <system>
        <section id="sales">
            <group id="gift_options" translate="label" type="text" sortOrder="100" showInDefault="1" showInWebsite="1">
                <label>Gift Options</label>
                <field id="allow_order" translate="label" type="select" sortOrder="1" showInDefault="1" showInWebsite="1" canRestore="1">
                    <label>Allow Gift Messages on Order Level</label>
                    <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
                </field>
                <field id="allow_items" translate="label" type="select" sortOrder="5" showInDefault="1" showInWebsite="1" canRestore="1">
                    <label>Allow Gift Messages for Order Items</label>
                    <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
                </field>
            </group>
        </section>
    </system>
</config>
```

# Mutations

Validation errors may be included to `SetGiftOptionsOnCartOutput` based on product requirements, like described [here](./customer-orders.md).

```graphql
###### Begin: Extending existing types ######
type CartItemUpdateInput {
    gift_wrapping_id: ID @doc(description: "The unique identifier of the gift wrapping to be used for the cart item")
    gift_message: GiftMessageInput @doc(description: "Gift message details for the cart item")
}

type Mutation {
    setGiftOptionsOnCart(input: SetGiftOptionsOnCartInput): SetGiftOptionsOnCartOutput @doc(description: "Set gift options like gift wrapping or gift message for the entire cart")
}

input SetGiftOptionsOnCartInput{
     cart_id: String! @doc(description:"The unique ID that identifies the shopper's cart")
     gift_message: GiftMessageInput @doc(description: "Gift message details for the cart")
     gift_wrapping_id: ID @doc(description: "The unique identifier of the gift wrapping to be used for the cart")
     printed_card_included: Boolean! @doc(description: "Whether customer requested printed card for the cart")
     gift_receipt_included: Boolean! @doc(description: "Whether customer requested gift receipt for the cart")
}

###### End: Extending existing types ######


###### Begin: Defining new types ######
type SetGiftOptionsOnCartOutput {
    cart: Cart! @doc(description: "The modified cart object")
}

input GiftMessageInput {
    to: String! @doc(description: "Recepient name")
    from: String! @doc(description: "Sender name")
    message: String! @doc(description: "Gift message text")
}
###### End: Defining new types ######
```



