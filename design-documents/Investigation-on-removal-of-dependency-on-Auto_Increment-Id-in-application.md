## Investigate removing dependency on autoincrement id in application

### Overview

When trying to upgrade Magento to MySQL v8 an issue was encountered that relates to relying on the `AUTO_INCREMENT` of
a table in several places. This document investigates on where this dependency is being used, why it is used and how
difficult it will be to get rid of this dependency.
Note that this document does not discuss the whether or not the dependency shall be removed but rather helps in making
the decision.

### List of places where `AUTO_INCREMENT` is used and for which purpose

Currently the `AUTO_INCREMENT` is read from database in the following two methods:
* `\Magento\ImportExport\Model\ResourceModel\Helper::getNextAutoincrement()`: Attempts to extract the `AUTO_INCREMENT`
    from table status and returns it so that it can be used for predictive purposes.
* `\Magento\Framework\Mview\View\Changelog::getVersion()`: Attempts to extract the `AUTO_INCREMENT` from table status,
    decreases it by 1 and returns that. Actually returns the maximal version number that is currently in use.
    This can probably be replaced by a query of the likes of `SELECT MAX(id) FROM table`.

Places that make use of `\Magento\ImportExport\Model\ResourceModel\Helper::getNextAutoincrement()` were identified as:

* CE
    * `\Magento\CatalogImportExport\Model\Import\Product::_saveLinks()`:
        Uses the result to predict the id of the next product link that is being imported.
    * `\Magento\CatalogImportExport\Model\Import\Product\Option::_importData()`:
        Uses the result to predict the id of the next option and its value(s) being imported.
    * `\Magento\ConfigurableImportExport\Model\Import\Product\Type\Configurable::_getNextAttrId()`:
        Uses the result to collect the labels of of the next super attribute of a configurable that is being imported.
        Since the `AUTO_INCREMENT` is only fetched once (in case it has not been fetched before) and is then incremented
        with each further call, this could lead to serious off-by-N issues.
    * `\Magento\CustomerImportExport\Model\Import\Address::_getNextEntityId()`:
        Only called in case no address id is specified for the address that is being imported. The result is then used
        to predict the id of the address that is being imported.
        The `AUTO_INCREMENT` is only fetched one, in case it has not been fetched before. Every further call returns the
        cached value and increments it (post increment) for the next call. This could lead to off-by-N issues.
    * `\Magento\CustomerImportExport\Model\Import\Customer::_getNextEntityId()`
        Only called in case no customer id is specified for the customer that is being imported. The result is then used
        to predict the id of the customer that is being imported.
        The `AUTO_INCREMENT` is only fetched one, in case it has not been fetched before. Every further call returns the
        cached value and increments it (post increment) for the next call. This could lead to off-by-N issues.
    * `\Magento\CustomerImportExport\Model\Import\CustomerComposite::__construct()`
        The result is used to predict the next available customer id in case it is not already specified in the
        optional constructor parameter `$data`.
* EE
    * Not used
* B2B
    * Not used

Places that make use of `\Magento\Framework\Mview\View\Changelog::getVersion()` were identified as:

* CE
    * `\Magento\Catalog\Model\Indexer\Category\Product\Plugin\MviewState::afterSetStatus()`
        The plugin is executed after `\Magento\Framework\Mview\View\StateInterface::setStatus()` has run. In case the
        status was set to `\Magento\Framework\Mview\View\StateInterface::STATUS_SUSPENDED`, uses the result to set the
        version id of the related materialized view.
    * `\Magento\Indexer\Console\Command\IndexerStatusCommand::getPendingCount()`
        Uses the result to fetch the amount of pending records for an index.
    * `\Magento\Framework\Mview\View::update()`
        Uses the result to perform an action on a specific version of a materialized view.
    * `\Magento\Framework\Mview\View::suspend()`
        The result is used to set the version id of the materialized view to the end of the changelog.
* EE
    * Not used
* B2B
    * Not used

### Refactoring approaches

#### `\Magento\ImportExport\Model\ResourceModel\Helper::getNextAutoincrement()`

Relying on the proprietary features of specific databases is a bad idea as it prevents us from changing the underlying
database. In this specific case, relying on the reported `AUTO_INCREMENT` value is even worse as it makes a potentially
wrong assumption about the state of the database and actually circumvents the transaction isolation capability of
databases.

All cases listed above use the `AUTO_INCREMENT` value to determine / predict the primary keys of rows that are being
imported. This is a task that databases are a lot better at and thus should be performed by the database.

I would propose to rely on transaction isolation in order to determine the id of an entity that is being imported.
If we take the customer import as an example, we would get the following steps:

1. Start a transaction
1. `INSERT` row in `customer_entity`
1. Get the id of the customer that was just created and store it in a variable like `$customerId`.
1. `INSERT` all customer attribute values contained in current row using `$customerId` as foreign key to associate it to
   the record that was just created in `customer_entity`
1. `COMMIT` the changes
1. Import customer addresses using the same approach

Of course this would mean to rewrite the import from ground up, which is no easy task (due to the complexity of the
import process and lack of documentation in the existing code). Analyzing how individual imports of the entities listed
above is way too much work for the scope of a single spike, thus this document will focus on two examples.

##### Refactoring of `\Magento\CustomerImportExport\Model\Import\Address`

###### Current flow

1. Preload customer data (preloads customers and addresses in order to be able to distinguish between updated and added
   records)
1. Consume data bunches performing the following steps for every bunch
1. Report and skip invalid rows
1. Decide whether row shall be deleted or added / updated
1. If row is to be added / updated, prepare its data:
    1. If no address id is specified in row, predict the id that it will have after import (this is the dangerous part)
    1. Extract non-static attributes and gather them in an array using the computed address id as an index in order to
       be able to associate the attributes with the address entity when persisting the data
    1. Decide whether current row is a default address and if so use the computed address id to indicate which type of
       default address (shipping / billing) the row defines
1. Persist the whole bunch using the prepared data
1. Move on to next bunch

###### Proposed flow

1. Preload existing customer data
1. Consume data bunches performing the following steps for every bunch
1. Start a database transaction
1. Report and skip invalid rows
1. Decide whether row shall be deleted or added / updated
1. Delete address if row indicates that it shall be deleted and move on to next row
1. If row is to be added / updated, extract address entity data and persist the address entity
    1. Get hold of address id if address was added
1. Extract / normalize non-static attributes and persist them using the actual address id to associate them with the
   address entity
1. Persist the attributes
1. Decide whether current row is a default address and if so persist that information on the customer entity using the
   actual address id
1. Commit the database transaction after all rows have been consumed
1. Move on to next bunch

###### Performance impact

This approach will have a minor impact on performance since it will issue one `INSERT INTO` statement per address entity
that is being added (as opposed to one `INSERT INTO` statement per bunch that contains added addresses in the current
implementation). When persisting the address attributes and defaults, the only performance impact will be on PHP side
when repeatedly calling the methods `\Magento\CustomerImportExport\Model\Import\Address::_saveAddressAttributes()`
and `\Magento\CustomerImportExport\Model\Import\Address::_saveCustomerDefaults()` which are currently called once per
bunch.

Benchmarks will have to show how big the impact on performance actually is. Should performance become an issue, it could
be improved by generating a [prepared statement](https://dev.mysql.com/doc/refman/5.5/en/sql-syntax-prepared-statements.html)
for inserts at the very beginning of the import and executing it for every address entity that is being added.

###### How to refactor

Refactoring the importer should not be a big deal as most of the existing code can be reused. The methods
`\Magento\CustomerImportExport\Model\Import\Address::_importData()` and
`\Magento\CustomerImportExport\Model\Import\Address::_prepareDataForUpdate()` need will see the main refactoring effort.
While `\Magento\CustomerImportExport\Model\Import\Address::_importData()` needs to implement the flow defined above,
`\Magento\CustomerImportExport\Model\Import\Address::_prepareDataForUpdate()` will have to be split into three methods:
* A method that extracts and returns the address entity data (aka static attributes)
* A method that extracts and returns the address attributes data
* A method that persists the defaults for the current row

The tricky part here will be data extraction since we don't want to iterate the attributes multiple times per row. This
can be avoided by writing a method that extracts the basic address entity data (i.e. the fields `entity_id`,
`parent_id` and `updated_at`) and second method that iterates the attributes and adds static attributes to the entity
data while returning the non-static attributes, e.g.:
```php
private function getBaseEntityData(array $rowData)
{
    return [
        'entity_id'  => $rowData['address_id'] ?? null,
        'parent_id'  => $this->_getCustomerId(strtolower($rowData[self::COLUMN_EMAIL]), $rowData[self::COLUMN_WEBSITE]),
        'updated_at' => (new \DateTime())->format(\Magento\Framework\Stdlib\DateTime::DATETIME_PHP_FORMAT),
    ];
}

private function getAttributesData(array $rowData, array &$entityData)
{
    $newAddress = !isset($entityData['address_id']);

    foreach ($this->_attributes as $attributeAlias => $attributeParams) {
        if (array_key_exists($attributeAlias, $rowData)) {
            $attributeParams = $this->adjustAttributeDataForWebsite($attributeParams, $websiteId);
            $value           = $rowData[$attributeAlias];

            if (!strlen($value)) {
                if (!$newAddress) {
                    continue;
                }

                $value = null;
            } elseif (in_array($attributeParams['type'], ['select', 'boolean', 'datetime', 'multiselect'])) {
                $value = $this->getValueByAttributeType($rowData[$attributeAlias], $attributeParams);
            }

            if ($attributeParams['is_static']) {
                $entityData[$attributeAlias] = $value;
            } else {
                $attributes[$attributeParams['table']][$attributeParams['id']]= $value;
            }
        }
    }

    return $attributes;
}
```

The method `\Magento\CustomerImportExport\Model\Import\Address::_saveAddressAttributes()` will have to changed to expect
the address id as a parameter instead of expecting it in the attributes data.
Accordingly `\Magento\CustomerImportExport\Model\Import\Address::_saveCustomerDefaults()` will have to be changed to
expect the parameters address id and row data. It should then contain the logic to decide whether the address is a
default address and if so persist this information.
Finally the method `\Magento\CustomerImportExport\Model\Import\Address::_mergeEntityAttributes()` can be removed
altogether as it won't be used anymore.

##### Refactoring of `\Magento\CatalogImportExport\Model\Import\Product`

The product import works differently than the address importer, i.e. it decides whether products are to be deleted or
added / updated long before it actually processes the data.
Since only a small part of the product import relies on the `AUTO_INCREMENT`, namely saving product links, this chapter
focuses on the that import phase, which is executed after all products have been saved.

###### Current flow

1. Fetch next available `AUTO_INCREMENT` (next link id) from product link table
1. Preload information about link attribute `position` for each type of link
1. Consume data bunches performing the following steps for every bunch
1. Determine the link types that shall be imported for current row
1. Determine the products (link targets) that shall be linked for current link type in current row
1. Report and skip non existing link targets
1. Gather link data using the next link id if the link does not exist yet
1. Gather link position data for product link
1. Increment next link id and move on to next link target
1. Persist gathered link data and link position data
1. Move on to next bunch

###### Proposed flow

1. Preload information about link attribute `position` for each type of link
1. Consume data bunches performing the following steps for every bunch
1. Start a database transaction
1. Determine the link types that shall be imported for current row
1. Determine the products (link targets) that shall be linked for current link type in current row
1. Report and skip non existing link targets
1. Persist link taking into account that link id might have been specified (`INSERT INTO ... ON DUPLICATE`)
1. Get hold of the link id
1. Persist link position data for persisted link using the actual link id
1. Move on to next link target
1. Commit the database transaction
1. Move on to next bunch

###### Performance impact

This approach will have a minor impact on performance since it will issue one `INSERT INTO` statement per link data and
link position data (as opposed to two `INSERT INTO` statements per bunch that contains link data in the current
implementation).

Benchmarks will have to show how big the impact on performance actually is. Should performance become an issue, it could
be improved by generating two [prepared statements](https://dev.mysql.com/doc/refman/5.5/en/sql-syntax-prepared-statements.html)
for inserts of link data and link poisition data at the very beginning of the import and executing it for every link
data / link position data that is being persisted.

###### How to refactor

The method `\Magento\CatalogImportExport\Model\Import\Product::_saveLinks()` needs to be changed, the line where
it fetches the `AUTO_INCREMENT` must be dropped. This is the easy part.

Refactoring the method `\Magento\CatalogImportExport\Model\Import\Product::processLinkBunches()` requires to change the
innermost loop. Here instead of gathering links, the link data must be persisted.
Additionally this is the method where the database transaction is started and committed.

Finally the method `\Magento\CatalogImportExport\Model\Import\Product::saveLinksData()` must be changed in such a way,
that it persists a link and its position. Its parameter `$productIds` must be changed to accept one product id only,
thus changing the signature from
```php
private function saveLinksData(Link $resource, array $productIds, array $linkRows, array $positionRows)
```
to
```php
private function saveLinksData(Link $resource, int $productId, array $linkRow, array $positionRow)
```
The main problem with this method is, that in case the import behaviour was set to something different than `append` (
i.e. `replace`), it will take care to delete all existing links from the specified products (`$productIds`) before
persisting the new data. The consequence is that we have to make sure to delete these links somewhere else, otherwise
we'll end up having a single link per product when replacing links. An idea would be to track which links have been
imported, compare this to the links found in the database and selectively delete all links of a product that have not
been imported when in `replace` behaviour. This may affect performance so there might be a better way.

#### `\Magento\Framework\Mview\View\Changelog::getVersion()`

Due to the nature of the call, the method can be changed to something like:
```php
    /**
     * Get maximum version_id from changelog
     *
     * @return int
     * @throws ChangelogTableNotExistsException
     * @throws RuntimeException
     */
    public function getVersion()
    {
        $changelogTableName = $this->resource->getTableName($this->getName());
        if (!$this->connection->isTableExists($changelogTableName)) {
            throw new ChangelogTableNotExistsException(new Phrase("Table %1 does not exist", [$changelogTableName]));
        }
        $row = $this->connection->fetchRow('SELECT MAX(version_id) AS max_version_id FROM ' . $changelogTableName);
        if (isset($row['max_version_id'])) {
            return (int)$row['max_version_id'];
        } else {
            throw new RuntimeException(
                new Phrase("Table status for %1 is incorrect. Can`t fetch version id.", [$changelogTableName])
            );
        }
    }
```

Note that instead of using the proprietary `SHOW TABLE STATUS` and then decrease the returned value of the column
`Auto_increment`, we can switch to using the standard ANSI SQL aggregate function `MAX()`. Of course this should remain
a short term fix to the issue as it is equally prone to race conditions. In the long run we should overhaul the
architecture to make use of a data model which knows the maximum version id.

### Performance of workarounds

In order to measure whether any of the proposed workarounds affect performance in a negative way, both proposals were
benchmarked on a MySQL 8.
The benchmark consists of importing 250,000 customers, 10 times in a row.

The benchmarking process is as follows:
1. Delete all customers
1. In the admin section navigate to _System > Import_
1. In the field _Entity Type_ select "Customers Main File"
1. In the field _Import Behavior_ select "Add/Update Complex Data"
1. In the field _Select File to Import_ upload the file containing customer data
1. Hit button _Check Data_
1. Start measuring time
1. Hit button _Import_
1. Stop measuring time
1. Start over

System setup:
* NginX 1.14.0
* PHP-FPM 7.3.9 with Zend OPcache v7.3.9
* Magento in Default mode
* MySQL 8.0.17

#### Unmodified

Modification: none

Import times:
* Run #1: 1.7 min
* Run #2: 1.7 min
* Run #3: 1.7 min
* Run #4: 1.7 min
* Run #5: 1.7 min
* Run #6: 1.7 min
* Run #7: 1.6 min
* Run #8: 1.7 min
* Run #9: 1.7 min
* Run #10: 1.7 min
* Average: 1.69 min
* Average per record: 0.00000676 seconds


#### Using `ANALYZE TABLE` before fetching the table status

Modification:
```php
    public function getNextAutoincrement($tableName)
    {
        $connection = $this->getConnection();
        //MODIFICATION STARTS
        $connection->query("ANALYZE TABLE `$tableName`");
        //MODIFICATION ENDS
        $entityStatus = $connection->showTableStatus($tableName);

        if (empty($entityStatus['Auto_increment'])) {
            throw new \Magento\Framework\Exception\LocalizedException(__('Cannot get autoincrement value'));
        }
        return $entityStatus['Auto_increment'];
    }
```

Import times:
* Run #1: 1.7 min
* Run #2: 1.8 min
* Run #3: 1.7 min
* Run #4: 1.7 min
* Run #5: 1.7 min
* Run #6: 1.8 min
* Run #7: 1.7 min
* Run #8: 1.8 min
* Run #9: 1.7 min
* Run #10: 1.7 min
* Average: 1.73 min
* Average per record: 0.00000692 seconds
* Performance loss: ~ 2,36%

#### Disabling table status cache

Modification:
```php
    public function getNextAutoincrement($tableName)
    {
        $connection = $this->getConnection();
        //MODIFICATION STARTS
        $connection->query("SET @@SESSION.`information_schema_stats_expiry` = 0");
        //MODIFICATION ENDS
        $entityStatus = $connection->showTableStatus($tableName);

        if (empty($entityStatus['Auto_increment'])) {
            throw new \Magento\Framework\Exception\LocalizedException(__('Cannot get autoincrement value'));
        }
        return $entityStatus['Auto_increment'];
    }
```

Import times:
* Run #1: 1.8 min
* Run #2: 1.7 min
* Run #3: 1.7 min
* Run #4: 1.6 min
* Run #5: 1.7 min
* Run #6: 1.7 min
* Run #7: 1.7 min
* Run #8: 1.6 min
* Run #9: 1.7 min
* Run #10: 1.8 min
* Average: 1.7 min
* Average per record: 0.0000068 seconds
* Performance loss: ~ 0.59%

#### Conclusion

The performance loss for both workarounds is minimal and can be neglected. While the first workaround will affect the
import only, the second workaround will affect all subsequent queries to the database in the same session.
Hence I'd opt for adding an `ANALYZE TABLE` if a workaround shall be used.

### Summary

Implementing the suggested workaround sounds tempting as it is a solution that will be available quickly. It will not
solve all issues though (fresh tables will return `null` for their `AUTO_INCREMENT` in MySQL 8, which will be cast to
`0` by the helper), hence I'd opt for refactoring the importers.

Refactoring will probably take round about 3-4 weeks of work including adapting / creating tests to make sure that
everything woks as expected.
I'd even propose to go one step further and rewrite the importers from ground up as the existing code is barely
maintainable (classes are too big, methods are too long, almost no documentation / comments). This would increase the
amount of work that needs to be done, but Magento would greatly benefit from that step in that the architecture of the
importers could be built to be extensible and replaceable.

If we decide to walk the refactoring path, I'd propose to do this for Magento 2.4 at the earliest.
