**Cart Discount Itemization**

Overview

The individual itemized break down of the discounts applied at cart and cart item level are to be computed and persisted on every order.

**Existing Implementation**

The itemized discounts are computed and stored with extension attributes on 
Magento\Quote\Api\Data\CartItemInterface 
Magento\Quote\Api\Data\AddressInterface. 
These are served via GraphQL cart query. The eventual end goal is to persist this data on every order.

**Schema changes**

The existing discount related metadata are being stored in quote_address, quote_item, sales_order, sales_order_item. But these tables tend to get large and have millions of records,
and is decided it is not wise to alter heavyweight tables. So new tables have to be created to hold discounts data.

Also have to be congizant about avoiding referencing between sales and quote tables.

The data will be stored as serialized json to support extensibility.

*quote_totals* 
```
 <table name="quote_totals" resource="checkout" engine="innodb" comment="Quote Totals">
        <column xsi:type="int" name="entity_id" padding="10" unsigned="true" nullable="false" identity="false"
                comment="Quote ID"/>
        <column xsi:type="text" name="value" nullable="true" comment="Serialized quote totals"/>
        <constraint xsi:type="primary" referenceId="PRIMARY">
            <column name="entity_id"/>
        </constraint>
        <constraint xsi:type="foreign" referenceId="QUOTE_TOTALS_ENITITY_ID_QUOTE_ENTITY_ID"    table="quote_totals" column="entity_id" referenceTable="quote" referenceColumn="entity_id" onDelete="CASCADE"/>
        <index referenceId="QUOTE_TOTALS_ENTITY_ID" indexType="btree">
            <column name="entity_id"/>
        </index>
 </table>
```
*sales_order_totals* 
```
 <table name="sales_order_totals" resource="checkout" engine="innodb" comment="Sales Order Totals">
        <column xsi:type="int" name="entity_id" padding="10" unsigned="true" nullable="false" identity="false"
                comment="Order ID"/>
        <column xsi:type="text" name="value" nullable="true" comment="Serialized order totals"/>
        <constraint xsi:type="primary" referenceId="PRIMARY">
            <column name="entity_id"/>
        </constraint>
        <constraint xsi:type="foreign" referenceId="SALES_ORDER_TOTALS_ENTITY_ID_SALES_ORDER_ENTITY_ID"   table="sales_order_totals" column="order_id" referenceTable="sales_order" referenceColumn="entity_id" onDelete="CASCADE"/>
        <index referenceId="SALES_ORDER_TOTALS_ENTITY_ID" indexType="btree">
            <column name="entity_id"/>
        </index>
 </table>
```

**Benefits**

Need to modify current table structure to incorporate this feature.
Will support extensibility in the future to store additional data pertaining to totals, without modifying the table.

