**Cart Discount Itemization**

Overview

The individual itemized break down of the discounts applied at cart and cart item level are to be computed and persisted on every order.

**Existing Implementation**

The itemized discounts are computed and stored with extension attributes on Magento\Quote\Api\Data\CartItemInterface Magento\Quote\Api\Data\AddressInterface. These are served via GraphQL cart query. The eventual end goal is to persist this data on every order.

```
/**
 * Rule discount Interface
 */
interface RuleDiscountInterface
{
    /**
     * Get Discount Data
     *
     * @return \Magento\SalesRule\Api\Data\DiscountDataInterface
     */
    public function getDiscountData();

    /**
     * Get Rule Label
     *
     * @return string
     */
    public function getRuleLabel();

    /**
     * Get Rule ID
     *
     * @return int
     */
    public function getRuleID();
}
```

```
 <extension_attributes for="Magento\Quote\Api\Data\CartItemInterface">
        <attribute code="discounts" type="Magento\SalesRule\Api\Data\RuleDiscountInterface[]" />
    </extension_attributes>
    <extension_attributes for="Magento\Quote\Api\Data\AddressInterface">
        <attribute code="discounts" type="Magento\SalesRule\Api\Data\RuleDiscountInterface[]" />
    </extension_attributes>
```

**Schema changes**

The existing discount related metadata are being stored in quote_address, quote_item, sales_order, sales_order_item. But these tables tend to get large and have millions of records, and is decided its not ideal to alter heavyweight tables. So new tables have to be created to hold discounts data.

Also have to be cognizant about avoiding referencing between sales and quote tables.

The data will be stored as serialized json to support extensibility.

quote_totals
```
 <table name="quote_attributes" resource="checkout" engine="innodb" comment="Quote Attributes Document">
        <column xsi:type="int" name="quote_id" padding="10" unsigned="true" nullable="false" identity="false"
                comment="Quote ID"/>
        <column xsi:type="text" name="value" nullable="true" comment="Serialized quote attributes"/>
        <constraint xsi:type="primary" referenceId="PRIMARY">
            <column name="quote_id"/>
        </constraint>
        <constraint xsi:type="foreign" referenceId="QUOTE_ATTRIBUTES_QUOTE_ID_QUOTE_ENTITY_ID"    table="quote_attributes" column="quote_id" referenceTable="quote" referenceColumn="entity_id" onDelete="CASCADE"/>
        <index referenceId="QUOTE_ATTRIBUTES_QUOTE_ID" indexType="btree">
            <column name="quote_id"/>
        </index>
 </table>
```
sales_order_totals
```
 <table name="sales_order_attributes" resource="checkout" engine="innodb" comment="Sales Order Attributes Document">
        <column xsi:type="int" name="order_id" padding="10" unsigned="true" nullable="false" identity="false"
                comment="Order ID"/>
        <column xsi:type="text" name="value" nullable="true" comment="Serialized order attributes"/>
        <constraint xsi:type="primary" referenceId="PRIMARY">
            <column name=order_id"/>
        </constraint>
        <constraint xsi:type="foreign" referenceId="SALES_ORDER_ATTRIBUTES_ORDER_ID_SALES_ORDER_ENTITY_ID"   table="sales_order_attributes" column="order_id" referenceTable="sales_order" referenceColumn="entity_id" onDelete="CASCADE"/>
        <index referenceId="SALES_ORDER_ATTRIBUTES_ORDER_ID" indexType="btree">
            <column name="order_id"/>
        </index>
 </table>
```
```
/**
 * Quote Attribute Interface
 */
interface QuoteAttributeInterface
{
    /**
     * Get Quote Id
     *
     * @return int
     */
    public function getQuoteId();

    /**
     * Get Value
     *
     * @return string
     */
    public function getValue();
}
```
**Benefits**

Need to modify current table structure to incorporate this feature. Will support extensibility in the future to store additional data pertaining to totals, without modifying the table.


