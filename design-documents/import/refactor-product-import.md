# Proposal to refactor product import in order to drop dependency on `AUTO_INCREMENT` and table status

While implementing support for MySQL 8, we found out that some places make use of the `AUTO_INCREMENT` to predict
the primary key of data rows that are being created. One of these places being the product import.

In general no application should rely on `AUTO_INCREMENT` when processing data for insertion, as generation of serial ids 
should be left to the RDBMS in order to be thread-safe. Unfortunately when importing relational data, such as products, 
we need to be able to predict identity of main records (`catalog_product_option` and `catalog_product_link` in our case).
Normally the entity models would take care of persisting the relational data involved but this is way too slow for big 
amounts of data, thus the importers resort to generating plain SQL `INSERT` or `UPDATE` queries.  
In order to improve the performance, as little SQL queries as possible should be made, thus inserting the main record 
and selecting it from database would slow down the process. Instead the importer predicts the next available serial id 
by fetching the table's `AUTO_INCREMENT` value one time and then incrementing it per new record.

This will eventually lead to collisions if another process inserts data into the same table shortly after the 
`AUTO_INCREMENT` was fetched but before the importer has persisted the new entity. Additionally this approach relies on
metadata that may not be present in RDBMS other than MySQL / MariaDB. Finally fetching the `AUTO_INCREMENT` may fail
on MySQL 8 due to the table statistics cache under the following circumstances:
1. If the table is empty and has never seen an `INSERT`, MySQL 8 returns `null` for the `AUTO_INCREMENT`
1. If the table's statistical data has not reached the statistics cache TTL, MySQL 8 will return an outdated value

## Approach 1: Sequences instead of `AUTO_INCREMENT`

RDBMS like Oracle and PostgreSQL use sequences in order to provide a thread-safe serial id. Though 
MySQL / MariaDB do not yet natively support sequences, they can be emulated by introducing a sequence table.

Magento already provides the possibility to make use of a sequence, but the implementation relies on the existence of a 
specific sequence table per table that will be fed by the sequence. This tends to be slow and creates more data than 
actually needed. In order to reduce the amount of data and increase performance, we could use a generic sequence table
and implement a stored procedure (actually a stored function) in the database that increments the sequence value and
returns the new value, as proposed by [Percona](https://www.percona.com/blog/2008/04/02/stored-function-to-generate-sequences/)

### Database

The following `db_schema.xml` is needed:
```
<schema xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:framework:Setup/Declaration/Schema/etc/schema.xsd">
    <table name="sequences" resource="default" engine="innodb">
        <column xsi:type="varchar" name="name" length="64" nullable="false"/>
        <column xsi:type="int" name="val" padding="10" unsigned="true" nullable="false"/>
        <constraint xsi:type="primary" referenceId="PRIMARY">
            <column name="name"/>
        </constraint>
    </table>
</schema>
```

Additionally the following stored functions must be created (currently not possible via `db_schema.xml`):
```
DELIMITER //
CREATE FUNCTION sequence_next_value(tableName VARCHAR(64)) RETURNS INT
BEGIN
    UPDATE seqeuences SET val=LAST_INSERT_ID(val+1) WHERE name=tableName;
    RETURN LAST_INSERT_ID();
END
//
CREATE FUNCTION sequence_current_value(tableName VARCHAR(64)) RETURNS INT
BEGIN
    SET @currentValue = NULL;
    SELECT val FROM sequences WHERE name=tableName INTO @currentValue;
    RETURN @currentvalue;
END
//
DELIMITER ;
```
This can be achieved in a schema patch, which will also make sure that the optional table prefix is prepended to the
name of the sequence table inside the stored functions.

Also the sequence entries for the aforementioned tables must be initialized, which can be done in a data patch that
takes existing entries into consideration.  

### Classes

```
class \Magento\Framework\EntityManager\Sequence\GenericSequence implements \Magento\Framework\DB\Sequence\SequenceInterface
{
    const SEQUENCE_NEXT_VALUE_TEMPLATE = "SELECT sequence_next_value('%s')";
    const SEQUENCE_CURRENT_VALUE_TEMPLATE = "SELECT sequence_current_value('%s')"

    private $resource;
    private $connectionName;
    private $entityTable;

    public function __construct(
        \Magento\Framework\App\ResourceConnection $resource,
        string $connectionName,
        string $entityTable
    ) {
        $this->resource       = $resource;
        $this->connectionName = $connectionName;
        $this->entityTable    = $entityTable
    }

    public function getNextValue()
    {
        $connection = $this->resource->getConnection();
        return $connection->query(sprintf(self::SEQUENCE_NEXT_VALUE_TEMPLATE), $this->entityTableName)->fetchCol();
    }

    public function getCurrentValue()
    {
        $connection = $this->resource->getConnection();
        return $connection->query(sprintf(self::SEQUENCE_CURRENT_VALUE_TEMPLATE, $this->entityTableName))->fetchCol();
    }
}
```

### di.xml

In the `di.xml` of the module `Magento_Catalog` the following lines need to be inserted:
```
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <virtualType name="productLinkSequence" type="\Magento\Framework\EntityManager\Sequence\GenericSequence">
        <arguments>
            <argument name="connectionName" xsi:type="string">default</argument>
            <argument name="entityTable" xsi:type="string">catalog_product_link</argument>
        </arguments>
    </virtualType>
    <virtualType name="productOptionSequence" type="\Magento\Framework\EntityManager\Sequence\GenericSequence">
        <arguments>
            <argument name="connectionName" xsi:type="string">default</argument>
            <argument name="entityTable" xsi:type="string">catalog_product_option</argument>
        </arguments>
    </virtualType>
    <type name="Magento\Framework\EntityManager\MetadataPool">
        <arguments>
            <argument name="metadata" xsi:type="array">
                <item name="Magento\Catalog\Api\Data\ProductLinkInterface" xsi:type="array">
                    <item name="sequence" xsi:type="object">productLinkSequence</item>
                </item>
                <item name="Magento\Catalog\Api\Data\ProductOptionInterface" xsi:type="array">
                    <item name="sequence" xsi:type="object">productOptionSequence</item>
                </item>
            </argument>
        </arguments>
    </type>
</config>
```

### Classes

The class `\Magento\Staging\Setup\Declaration\Schema\Db\MySQL\DDL\Triggers\MigrateSequenceColumnData` needs to be moved
to `\Magento\Framework\Setup\Declaration\Schema\Db\MySQL\DDL\Triggers\MigrateSequenceColumnData` in order to avoid code 
duplication (currently the class is available in Magento 2 EE only).

The following classes need to be changed in order to make use of sequences:
* `\Magento\Catalog\Model\ResourceModel\Product\Link` when saving links the class currently performs `INSERT` and `UPDATE`
    via the connection, this must be changed to make use of the `\Magento\Framework\EntityManager\EntityManager` which
    will automatically use the sequence if available.
* `\Magento\Catalog\Model\ResourceModel\Product\Option` currently relies on `\Magento\Framework\Model\ResourceModel\Db\AbstractDb::save()`
    to persist data, this must be changed to make use of the `\Magento\Framework\EntityManager\EntityManager` which
    will automatically use the sequence if available.
* `\Magento\CatalogImportExport\Model\Import\Product` when processing link bunches
* `\Magento\CatalogImportExport\Model\Import\Product\Option` when importing options and their values

### Considerations

The product import will slow down a little bit as every new product link and new product option will add one query to 
the overall process. The performance impact should be minimal though since we use a stored function which is a lot 
faster than initiating an `INSERT` like `\Magento\Framework\EntityManager\Sequence\Sequence` does (also updating an 
existing row takes less time than inserting a new one). If we try to import 500 products with 5 new links 
each, this will sum up to 2500 additional queries as compared to the current implementation.  
Profiling of this has been performed by Lars Röttig in the scope of creating a proposal for the refactoring of the 
customer import (see [MC-21651](https://jira.corp.magento.com/browse/MC-21651)).

### Estimation

As detailed above, the implementation and refactoring is quite straight forward. The most difficult part will be creating
the needed stored functions as this is not yet supported by the declarative schema. This must be done in a schema patch,
which is no big issue though.

I'd estimate 3 to 4 man-days for the job.

## Approach 2: UUID

Usage of a serial primary key has been around for a very long time, yet it has always been known to be suboptimal.
Either the software has to make sure to generate a key that is unique, which is not an easy feat, or generation is left
to the RDBMS using a sequence-ish construct (e.g. `AUTO_INCREMENT` in MySQL / MariaDB).
In the first case race conditions can occur as multiple threads may generate the same key, in the last case the RDBMS
has to be queried in order to retrieve the generated key.

Additionally both options will eventually lead to the case where all keys are used up (depending on the size of the 
serial field). On systems that perform a lot of `INSERT`s this may happen sooner than one might think.

This is where UUID come in handy. As per definition UUID are supposed to be universally unique (which is not entirely 
true with UUID4 but the [probability for a collision](https://en.wikipedia.org/wiki/Universally_unique_identifier#Collisions)
is so small that it can be ignored) thus they can be generated by the application no matter how many application copies 
use the same database to store data. Depending on the RDBMS, UUID can be stored and indexed natively (e.g. PostgreSQL 
has a custom type `uuid`), on MySQL / MariaDB though we would have to use a `char(36)` (since UUID are always 36 
characters long and `char` performs better than `varchar`).

### Database

The following tables need change:
* Column `catalog_product_option.option_id` must be renamed to `catalog_product_option.old_option_id` such that external 
    services that query for product options by id can still be fed with the expected data. Also we will need the values
    of this column to transpose referential integrity of existing records to the new UUIDs.
* Column `catalog_product_option.option_id` needs to be created as `char(36)` and filled with new UUIDs.
* Column `catalog_product_option_price.option_id` must be changed to datatype `char(36)` and referential integrity must
    be recreated to make use of the new values from `catalog_product_option.option_id`.
* Foreign Key Constraint `CAT_PRD_OPT_PRICE_OPT_ID_CAT_PRD_OPT_OPT_ID` must be rebuilt.
* Unique Constraint `CATALOG_PRODUCT_OPTION_PRICE_OPTION_ID_STORE_ID` must be rebuilt.
* Column `catalog_product_option_title.option_id` must be changed to datatype `char(36)` and referential integrity must
    be recreated to make use of the new values from `catalog_product_option.option_id`.
* Foreign Key Constraint `CAT_PRD_OPT_TTL_OPT_ID_CAT_PRD_OPT_OPT_ID` must be rebuilt.
* Column `catalog_product_option_type_value.option_type_id` must be changed to datatype `char(36)`.
* Column `catalog_product_option_type_value.option_id` must be changed to datatype `char(36)` and referential integrity 
    must be recreated to make use of the new values from `catalog_product_option.option_id`.
* Foreign Key Constraint `CAT_PRD_OPT_TYPE_VAL_OPT_ID_CAT_PRD_OPT_OPT_ID` mut be rebuilt.
* Column `catalog_product_option_type_price.option_type_id` must be changed to datatype `char(36)` and referential integrity 
    must be recreated to make use of the new values from `catalog_product_option_type_value.option_type_id`.
* Foreign Key Constraint `FK_B523E3378E8602F376CC415825576B7F` must be rebuilt.
* Column `catalog_product_option_type_title.option_type_id` must be changed to datatype `char(36)`  and referential integrity 
    must be recreated to make use of the new values from `catalog_product_option_type_value.option_type_id`.
* Foreign Key Constraint `FK_C085B9CF2C2A302E8043FDEA1937D6A2` must be rebuilt.
* Column `catalog_product_link.link_id` must be renamed to `catalog_product_link.old_link_id` such that external 
    services that query for product links by id can still be fed with the expected data. Also we will need the values
    of this column to transpose referential integrity of existing records to the new UUIDs.
* Column `catalog_product_link.link_id` needs to be created as `char(36)` and filled with new UUIDs.
* Column `catalog_product_link_attribute_decimal.link_id` must be changed to datatype `char(36)` and referential integrity 
    must be recreated to make use of the new values from `catalog_product_link.link_id`.
* Foreign Key Constraint `CAT_PRD_LNK_ATTR_DEC_LNK_ID_CAT_PRD_LNK_LNK_ID` must be rebuilt.
* Column `catalog_product_link_attribute_int.link_id` must be changed to datatype `char(36)` and referential integrity 
    must be recreated to make use of the new values from `catalog_product_link.link_id`.
* Foreign Key Constraint `CAT_PRD_LNK_ATTR_INT_LNK_ID_CAT_PRD_LNK_LNK_ID` must be rebuilt.
* Column `catalog_product_link_attribute_varchar.link_id` must be changed to datatype `char(36)` and referential integrity 
    must be recreated to make use of the new values from `catalog_product_link.link_id`.
* Foreign Key Constraint `CAT_PRD_LNK_ATTR_VCHR_LNK_ID_CAT_PRD_LNK_LNK_ID` must be rebuilt.

### Classes

First off we need a service that returns UUIDs and can be used by entity and/or resource classes to create an identifier
in case there is none. Luckily Magento already has such a service built in, `\Magento\Framework\DataObject\IdentityService`.

The following classes need change:
* In `\Magento\Catalog\Model\ResourceModel\Product\Option` the following property and method must be added:
    ```
        /** @var \Magento\Framework\DataObject\IdentityService */
        private $identityService;
    
        protected function _beforeSave(\Magento\Framework\Model\AbstractModel $object)
        {
            if ($this->isObjectNew($object)) {
                $object->setId($this->identityService->generateId());
            }
  
            return parent::_beforeSave($object);
        }
    ```
* In the class `\Magento\Catalog\Model\ResourceModel\Product\Option\Value` the following property and method must be added:
    ```
        /** @var \Magento\Framework\DataObject\IdentityService */
        private $identityService;
        
        protected function _beforeSave(\Magento\Framework\Model\AbstractModel $object)
        {
          if ($this->isObjectNew($object)) {
              $object->setId($this->identityService->generateId());
          }
        
          return parent::_beforeSave($object);
        }
    ```
* In `\Magento\Catalog\Model\ResourceModel\Product\Link` the following property and method must be added:
    ```
        /** @var \Magento\Framework\DataObject\IdentityService */
        private $identityService;
    
        protected function _beforeSave(\Magento\Framework\Model\AbstractModel $object)
        {
            if ($this->isObjectNew($object)) {
                $object->setId($this->identityService->generateId());
            }
  
            return parent::_beforeSave($object);
        }
    ```
* In the method `\Magento\Catalog\Model\ResourceModel\Product\Link::prepareProductLinksData` the lines
    ```
        $bind = [
            'product_id' => $parentId,
            'linked_product_id' => $linkedProductId,
            'link_type_id' => $typeId,
        ];
        $connection->insert($this->getMainTable(), $bind);
        $linkId = $connection->lastInsertId($this->getMainTable());
    ```
    must be changed to
    ```
        $linkId = $this->identityService->generateId();
        $bind = [
            'link_id' = $linkId,
            'product_id' => $parentId,
            'linked_product_id' => $linkedProductId,
            'link_type_id' => $typeId,
        ];
        $connection->insert($this->getMainTable(), $bind);
    ```
* In `\Magento\CatalogImportExport\Model\Import\Product` the following property must be added:
    ```
        /** @var \Magento\Framework\DataObject\IdentityService */
        private $identityService;
    ```
* In the method `\Magento\CatalogImportExport\Model\Import\Product::_saveLinks` the following line must be removed
    ```
        $nextLinkId = $this->_resourceHelper->getNextAutoincrement($mainTable);
    ```
* The signature of the method `\Magento\CatalogImportExport\Model\Import\Product::processLinkBunches` must be changed to
    ```
        private function processLinkBunches(array $bunch, Link $resource, array $positionAttrId)
    ```
* In the method `\Magento\CatalogImportExport\Model\Import\Product::processLinkBunches` the line
    ```
        $productLinkKeys[$linkKey] = $productLinkKeys[$linkKey] ?? $nextLinkId;
    ``` 
    must be changed to
    ```
        $productLinkKeys[$linkKey] = $productLinkKeys[$linkKey] ?? $this->identityService->generateId();
    ```
* In the method `\Magento\CatalogImportExport\Model\Import\Product::processLinkBunches` the line
    ```
        $nextLinkId++;
    ```
    must be removed.
* In the class `\Magento\CatalogImportExport\Model\Import\Product\Option` the following property must be added:
    ```
        /** @var \Magento\Framework\DataObject\IdentityService */
        private $identityService;
    ``` 
* In the method `\Magento\CatalogImportExport\Model\Import\Product\Option::_importData` the lines
    ```
        $nextOptionId = $this->_resourceHelper->getNextAutoincrement($this->_tables['catalog_product_option']);
        $nextValueId = $this->_resourceHelper->getNextAutoincrement(
            $this->_tables['catalog_product_option_type_value']
        );
    ```
    must be changed to
    ```
        $nextOptionId = $this->identityService->generateId();
        $nextValueId = $this->identityService->generateId();
    ```

### Considerations

Traditionally Magento uses `int(10)` to store serial primary keys, which evaluates to 4 bytes of data per row. 
On MySQL / MariaDB UUID are stored as `char(36)` which evaluates to 36 bytes of data per row.
This means that we'd add 32 bytes of storage per row if we switched to UUID.

For the same reason, querying the database for an entity that uses UUID primary keys by its id could be a bit slower 
as the index lookup may take a little longer (depending on the type of index that is being used).

Since a process can create new entities without the need to talk to the database, the performance of the import will not
degrade in any way, which is definitely worth the trade-off. 

### Estimation

As displayed above, refactoring the classes is rather easy as we simply have to replace fetching the `AUTO_INCREMENT`
by generation of a UUID in the importers and give control over identity generation to the resource
models. Altering the database and migrating from serial id to UUID without losing referential integrity on the other 
hand will be a complex task that requires good planning and lots of testing.

I'd estimate roughly 5 to 8 man-days for the job. 

## Conclusion

Although using UUIDs instead of serial ids would be a great option, migrating existing data could become troublesome and
the overall performance of using a `char` field for primary key needs to be benchmarked before even attempting to use
UUIDs. Additionally this is probably a BIC, thus it is not an option for a patch release.

Switching to sequences on the other hand brings stability and thread-safety at minimal cost. The changes that need to be 
done are straight forward and will definitely not raise a BIC.

Considering all aspects, I'd suggest switching to generic sequences. In the long run I'd suggest to support sequences in
the declarative schema and let the database abstraction layer decide how these are implemented for arbitrary RDBMS (e.g. 
native sequences in Oracle, PostgreSQL and MSSQL, generic sequence table in MySQL / MariaDB etc.). The `identity` 
attribute of columns in the declarative schema could then be used to indicate which column is fed by the sequence and 
thus initiate the sequence for the table in case is does not exist. This would give us the opportunity to get rid of 
`AUTO_INCREMENT` (and its MSSQL equivalent `IDENTITY`) altogether.
