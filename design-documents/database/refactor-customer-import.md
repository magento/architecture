# MC-21651 Proposal to refactor the customer and address import depending on AUTO_INCREMENT and showTableStatus

## Current usage in customer import (Open Source Edition)

* `\Magento\CustomerImportExport\Model\Import\Address::_getNextEntityId()` **@api**
    *  Only called in case no address id is specified for the address that is being imported. The result is then used to
        predict the id of the address that is being imported. The `AUTO_INCREMENT` is only fetched one, in case it has 
        not been fetched before. Every further call returns the cached value and increments it (post increment) for the 
        next call. This could lead to off-by-N issues.
* `\Magento\CustomerImportExport\Model\Import\Customer::_getNextEntityId()` **@api**
    * Only called in case no customer id is specified for the customer that is being imported. The result is then used
        to predict the id of the customer that is being imported.
        The `AUTO_INCREMENT` is only fetched one, in case it has not been fetched before. Every further call returns the
        cached value and increments it (post increment) for the next call. This could lead to off-by-N issues.
* `\Magento\CustomerImportExport\Model\Import\CustomerComposite::__construct()` **@api**
    * The result is used to predict the next available customer id in case it is not already specified in the
        optional constructor parameter `$data`.

## Ideas of Refactoring

### 1. Use `Sequence` implementation from Framework

Every `Sequence` follows the api `Magento\Framework\DB\Sequence\SequenceInterface`

### The benchmark consists of importing 1000 customers an median of 3 measurements.

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

**System setup:**

* NginX 1.14.0
* PHP-FPM 7.2.9 with Zend OPcache v7.3.9
* Magento in Developer Mode
* MySQL 5.6

#### Unmodified

* Time: 3,86 sec
* IO: 245ms
* CPU: 3.62 sec
* Network: 1.05 mb
* Sequel Queries: 200ms 87rq
  ​            

Concept 1 - Use `class CustomerSequence implements SequenceInterface` implementation
----------------------

```php
protected function _getNextEntityId()
{
    return $this->sequence->getNextValue();
}   
```
* Time: 13.3 sec
* IO: 4.64 sec
* CPU: 8.66sec
* Network: 1.08 mb
* Sequel Queries: 4.78sec 1,085rq

**Advantage:** 

- For import we only need to add optional parameter to `\Magento\CustomerImportExport\Model\Import\Customer::__construct`
- More flexibility 

**Disadvantage:**

- Slower because it triggers much more queries, this will be worse if we are using *AWS* database cluster for import
- We need to maintain the implementation. Maybe it is better to use MetadataPool implementation from the core

Concept 2 - `Magento\Framework\EntityManager\MetadataPool` implementation
----------------------

```php
protected function _getNextEntityId()
{
    return $this->metadataPool->getMetadata(CustomerInterface::class)->generateIdentifier();
}
```

**di.xml:**

 ```xml          
<type name="Magento\Framework\EntityManager\MetadataPool">
    <arguments>
        <argument name="metadata" xsi:type="array">
            <item name="Magento\Customer\Api\Data\CustomerInterface" xsi:type="array">
                <item name="sequenceTable" xsi:type="string">sequence_customer</item>
            </item>
        </argument>
    </arguments>
</type>     
 ```
* Time: 10.6 sec
* IO: 4.3 sec
* CPU: 6.3sec
* Network: 1.08 mb
* Sequel Queries: 4.4sec 1,085rq

**Advantage:** 

- For import we only need to add optional parameter to `\Magento\CustomerImportExport\Model\Import\Customer::__construct`

**Disadvantage:**

- Slower because it triggers much more queries, this will be worse if we are using *AWS* database cluster for import

### Summary Concept 1 and Concept 2

Both concepts add two queries per new customer.

If we now think about to import 200.000 new customers on **AWS**, it is much slower than before. For those reasons, we 
need a concept that helps us to optimize the queries that we send to MySQL.
  
  
Concept 3 - optimize queries 
----------

We need to split the import bunch by if already know the customer, this we can reach with query like.

To implement this we need to move  **magento2ce/app/code/Magento/CustomerImportExport/Model/Import/Customer.php**:537- 555 in a two service classes. 

```mysql
SELECT entity_id, email
  FROM customer_entity
 WHERE CONCAT(email,website_id) IN (
     'malcolm85@gmail.com1',
     'khuel@yahoo.com1',
     'newcustomer@yahoo.com1'
);
```

**Result:**

| entity\_id | email               |
| :--------- | :------------------ |
| 4290       | khuel@yahoo.com     |
| 4288       | malcolm85@gmail.com |

This helps to find all customers that we already have created in Magento. With this Information we can refactor the `protected function _importData()` 

```php
while ($bunch = $this->_dataSourceModel->getNextBunch()) {
    $this->prepareCustomerData($bunch);
    $entitiesToCreate = [];
    $entitiesToUpdate = [];
    $entitiesToDelete = [];
    $attributesToSave = [];
    
    $bunchDataByMail = [];
    $customerAddresses = [];
    foreach ($bunch as $rowNumber => $rowData) {
        if (!$this->validateRow($rowData, $rowNumber)) {
            continue;
        }
        if ($this->getErrorAggregator()->hasToBeTerminated()) {
            $this->getErrorAggregator()->addRowToSkip($rowNumber);
            continue;
        }
        $email = $rowData[self::COLUMN_EMAIL];
        $customerAddresses[] = $email.$rowData['website_id'];
        $bunchDataByMail[$email] = $rowData;
    }
    
    $query = $this->_connection->select()->from(
        'customer_entity',
        ['entity_id', 'email']
    )->where('CONCAT(email,website_id) in (?)', $customerAddresses);
    
    if ($this->getBehavior($rowData) == Import::BEHAVIOR_DELETE) {
        $entitiesToDelete = $this->_connection->fetchCol($query, 'entity_id');
    } elseif ($this->getBehavior($rowData) == Import::BEHAVIOR_ADD_UPDATE) {
        $entitiesToUpdate = $this->_connection->fetchAll($query, 'email');
        
        /* should filter $validBunchData[$email] by $entitiesToUpdate and split them in two arrays $entitiesToUpdate and $entitiesToCreate*/
    }
```

With these two arrays we can use the row by email to create the data to import. 

To generate new ids I recommend to add functions:

`Sequence`

```php
public function getNextValueBunch(int $count): array
{
    $currentValue = $this->getCurrentValue();
    $nextvalue = $currentValue + $count;
    $this->resource->getConnection($this->connectionName)
        ->insert($this->resource->getTableName($this->sequenceTable), ['sequence_value' => $nextValue]);
    return range($currentValue, $nextValue);
}
```

 `EntityMetadata` 

```php
public function generateIdentifierBunch(int $count) : array
{
    $nextIdentifiers = [];
    if ($this->sequence) {
        $nextIdentifiers = $this->sequence->getNextValueBunch($count);
    }
    return $nextIdentifiers;
}
```

**Advantage:** 

* We can apply this approach to Address and Customer and import should improve performance and stability  


# Concept 4 -  Stored Function to generate Sequences

I use as base the following technical Article  https://www.percona.com/blog/2008/04/02/stored-function-to-generate-sequences/

Query that I run before I started performance testing: 

```mysql
CREATE TABLE `sequence` (
  `type` varchar(20) NOT NULL,
  `value` int(10) unsigned NOT NULL,
  PRIMARY KEY  (`type`)
) ENGINE=MyISAM DEFAULT CHARSET=latin1;

INSERT INTO sequence VALUES ('customer', 1);

DELIMITER //
CREATE FUNCTION sequence(seq_type char (20)) RETURNS int
BEGIN
 UPDATE sequence SET value=last_insert_id(value+1) WHERE type=seq_type;
 RETURN last_insert_id();
END
//
DELIMITER ;
```

Implementation of ` _getNextEntityId()` 

```php
protected function _getNextEntityId()
{
  return $this->_connection->query("select sequence('customer')")->fetchColumn();
}
```

* Time: 3.92 sec
* IO: 511ms
* CPU: 3.4sec
* Network: 1.15 mb
* Sequel Queries: 538.5ms 1,084rq

# Summary

For all 4 concepts we need to have a migration from `customer_entity` to `sequence` table idea like 
`migrateSequneceColumnData(customer_entity,entity_id)`. I think we will get the most benefit with the implementation of 
Concept 3 because we are sending less queries to the database.  
All concepts can be implemented in a backward compatible way because we only touch constructors and protected functions 
for import.  
