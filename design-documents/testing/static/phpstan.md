# Overview
[PHPStan](https://github.com/phpstan/phpstan) is a code analysis tool which focuses on finding bugs without running the code.
PHPStan uses [nikic/PHP-Parser](https://github.com/nikic/PHP-Parser) under the hood to build AST (abstract syntax tree) and perform sophisticated static code analysis.

### Prerequisites
- PHP >= 7.1.
- Generated code (factories, proxies, and so on).

### What it checks for?
- Class existence.
- Existence and accessibility of called methods and functions.
- Property existence and visibility.
- The number of passed arguments.
- Correctness of typehints.
- Whether a method returns the same type it declares to return.
- Number of parameters passed to `sprintf`/`printf` calls based on format strings.
- Duplicate array keys.
- Dead code.

This is not the full list of it's features. More details can be found in the [PHPStan GitHub repository](https://github.com/phpstan/phpstan) .

## Rule levels
PHPStan offers different levels of checks - from 0 (loosest) to 7 (strictest)

It may help in the incremental adoption of PHPStan checks.

Default level is 0.

### Supported output formats
- text(raw or table)
- xml(checkstyle),
- json(can be prettified)
- custom format

## Magento 2.3.0 Findings Examples

In total found >150 errors on level 0 check. Here is the list with some examples from Magento 2.3.0 codebase:

```
  ------ ---------------------------------------------------------------------------------------------------------------------
   Line   app/code/Magento/Catalog/Model/ResourceModel/Product/Action.php
  ------ ---------------------------------------------------------------------------------------------------------------------
   91     Method Magento\Catalog\Model\ResourceModel\Product\Action::resolveEntityId() invoked with 2 parameters, 1 required.
  ------ ---------------------------------------------------------------------------------------------------------------------

  ------ -------------------------------------------------------------------------------------------------------------------------------------------
   Line   app/code/Magento/Catalog/Model/Rss/Product/NotifyStock.php
  ------ -------------------------------------------------------------------------------------------------------------------------------------------
   42     Magento\Catalog\Model\Rss\Product\NotifyStock::__construct() does not call parent constructor from Magento\Framework\Model\AbstractModel.
  ------ -------------------------------------------------------------------------------------------------------------------------------------------

  ------ --------------------------------------------------------------------------------
   Line   app/code/Magento/Catalog/Setup/CategorySetup.php
  ------ --------------------------------------------------------------------------------
   596    Class Magento\Catalog\Block\Adminhtml\Product\Helper\Form\BaseImage not found.
   629    Class Magento\Catalog\Setup\Media not found.
  ------ --------------------------------------------------------------------------------

  ------ -------------------------------------------------------------------
   Line   app/code/Magento/Checkout/Model/Type/Onepage.php
  ------ -------------------------------------------------------------------
   408    Class customer_address not found.
  ------ -------------------------------------------------------------------

  ------ -----------------------------------------------------------------------------------------------------------------------------
   Line   app/code/Magento/MysqlMq/Model/QueueManagement.php
  ------ -----------------------------------------------------------------------------------------------------------------------------
   278    Return typehint of method Magento\MysqlMq\Model\QueueManagement::readMessages() has invalid type Magento\MysqlMq\Model\pre.
  ------ -----------------------------------------------------------------------------------------------------------------------------

  ------ -------------------------------------------------------------------------------------------------------------------------------
   Line   app/code/Magento/Newsletter/Model/ResourceModel/Subscriber/Grid/Collection.php
  ------ -------------------------------------------------------------------------------------------------------------------------------
   20     Method Magento\Newsletter\Model\ResourceModel\Subscriber\Collection::showCustomerInfo() invoked with 1 parameter, 0 required.
  ------ -------------------------------------------------------------------------------------------------------------------------------

  ------ --------------------------------------------------------------------------------------------------------------------
   Line   app/code/Magento/OfflineShipping/Model/ResourceModel/Carrier/Tablerate/CSV/ColumnResolver.php
  ------ --------------------------------------------------------------------------------------------------------------------
   25     Array has 2 duplicate keys with value 'Weight (and above)' (self::COLUMN_WEIGHT, self::COLUMN_WEIGHT_DESTINATION).
  ------ --------------------------------------------------------------------------------------------------------------------

```

### Time to Analyse Magento in Comparison with PHP Mess Detector)
|  | PHPStan | PHPMD |
| ------------- | ------------- |------------- |
| app/code/Magento/AdminNotification  | 0m1.711s  | 0m36.907s |
| app/code  | 8m12.334s | 132m58.415s |

# Proposal
Add `phpstan/phpstan` to Magento `require-dev` section, update static build by adding level 0 check and fix all corresponding findings.

## Long Term Plan
1. Eliminate the use PHP Mess Detector. Replace existing checks with PHPStan.
2. Iteratively increase the level of check.

### Useful Links
[PHPStan GitHub Repository](https://github.com/phpstan/phpstan)

[PHPStan: Find Bugs In Your Code Without Writing Tests!](https://medium.com/@ondrejmirtes/phpstan-2939cd0ad0e3)

[PHPStan 0.11](https://medium.com/@ondrejmirtes/phpstan-0-11-5aba0e4108c8)
