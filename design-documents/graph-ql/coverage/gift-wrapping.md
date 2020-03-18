# Queries

## Data

```graphql

###### Begin: Extending existing types ######
type Cart {
    available_gift_wrappings: [GiftWrapping]!
    gift_wrapping: GiftWrapping
    include_printed_card: Boolean! @doc(description: "Wether customer requested printed card for the order")
    include_gift_receipt: Boolean! @doc(description: "Wether customer requested gift receipt for the order")
}

type SimpleCartItem {
    available_gift_wrapping: [GiftWrapping]!
    gift_wrapping: GiftWrapping
}

type ConfigurableCartItem {
    available_gift_wrapping: [GiftWrapping]!
    gift_wrapping: GiftWrapping
}

type BundleCartItem {
    available_gift_wrapping: [GiftWrapping]!
    gift_wrapping: GiftWrapping
}

type SalesItemInterface {
    gift_wrapping: GiftWrapping
}

type CustomerOrder {
    gift_wrapping: GiftWrapping
    include_printed_card: Boolean! @doc(description: "Wether customer requested printed card for the order")
    include_gift_receipt: Boolean! @doc(description: "Wether customer requested gift receipt for the order")
}

type CartPrices {
    gift_options: giftOptionsPrices
}
###### End: Extending existing types ######


###### Begin: Defining new types ######
type GiftWrapping {
    id: ID!
    design: String!
    price: Money!
    image: GiftWrappingImage!
}

type GiftWrappingImage {
    label: String!
    url: String!
}

type giftOptionsPrices {
    gift_wrapping_for_order: Money
    gift_wrapping_for_items: Money
    printed_card: Money
}
###### End: Defining new types ######

```

## Configuration

The following gift options need to be whitelisted in the `storeConfig` query.

```xml
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Config:etc/system_file.xsd">
    <system>
        <section id="sales">
            <group id="gift_options">
                <field id="wrapping_allow_order" translate="label" type="select" sortOrder="10" showInDefault="1" showInWebsite="1" showInStore="0">
                    <label>Allow Gift Wrapping on Order Level</label>
                    <source_model>Magento\Config\Model\Config\Source\Yesno</source_model>
                </field>
                <field id="wrapping_allow_items" translate="label" type="select" sortOrder="15" showInDefault="1" showInWebsite="1" showInStore="0">
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


# Mutations

```graphql
###### Begin: Extending existing types ######
type CartItemUpdateInput {
    gift_wrapping_id: ID
}

type Mutation {
    setGiftOptionsOnCart(cart_id: String!, gift_wrapping_id: ID, include_gift_receipt: Boolean, include_printed_card: Boolean): setGiftOptionsOnCartOutput
}
###### End: Extending existing types ######


###### Begin: Defining new types ######
type setGiftOptionsOnCartOutput {
    cart: Cart!
}
###### End: Defining new types ######
```



