**Cart Discount Itemization**

Overview
The individual itemized break down of the discounts applied at cart and cart item level are to be computed and persisted on every order.

Existing Implementation
The itemized discounts are computed and stored with extension attributes on 
Magento\Quote\Api\Data\CartItemInterface 
Magento\Quote\Api\Data\AddressInterface. 
These are served via GraphQL cart query. The eventual end goal is to persist this data with every order.

Schema changes
#The following is a draft for the schema changes #

The existing discount related metadata are being stored in quote_address, quote_item, sales_order, sales_order_item. But these tables tend to have millions of rows,
and is decided it is not wise to alter heavyweight tables. So new tables have to be created to hold discounts data.

Also have to be congizant about avoiding referencing between sales and quote tables.

 <table name="quote_item_totals" resource="checkout" engine="innodb" comment="Quote Item Totals">
        <column xsi:type="int" name="item_id" padding="10" unsigned="true" nullable="false" identity="false"
                comment="Item ID"/>
        <column xsi:type="text" name="totals" nullable="true" comment="Collected item totals"/>
        <constraint xsi:type="primary" referenceId="PRIMARY">
            <column name="item_id"/>
        </constraint>
        <constraint xsi:type="foreign" referenceId="QUOTE_ITEM_TOTALS_ITEM_ID_QUOTE_ITEM_ITEM_ID" table="quote_item_totals"
                    column="item_id" referenceTable="quote_item" referenceColumn="item_id" onDelete="CASCADE"/>
        <index referenceId="QUOTE_ITEM_OPTION_ITEM_ID" indexType="btree">
            <column name="item_id"/>
        </index>
  </table>
