# MySQL 8 - Table Status Support

## Background

Magento DB adapter interface provides method for obtaining table status: `\Magento\Framework\DB\Adapter\AdapterInterface::showTableStatus()`.
For MySQL adapter it's implemented as `SHOW TABLE STATUS FROM <db> LIKE <table>`.
Note: the interface doesn't prescribe what data and in which format can be returned.

## Problem Statement

MySQL 8 added caching layer on top of table status, which is refreshed from time to time (default `86400` = `1 day`).
This leads to the behavior where returned `auto_increment` value is `null` even though it shouldn't be (there is an auto_increment column in the table).
The table status information is not calculated until the table is accessed, and (probably) the expiration time is not passed.

### Real Use Cases

1. Magento native import relies on auto_increment for entity relations. See `\Magento\ImportExport\Model\ResourceModel\Helper::getNextAutoincrement()`.
2. Mview. See `\Magento\Framework\Mview\View\Changelog::getVersion()`.
3. Extensions may rely on the adapter interface for their scenarios.

## Possible Solutions

### Remove auto_increment from the interface

Reasoning: application should not deal with auto_increment, nor rely on it.

This decision will require reworking of import. This may introduce performance degradation for import.

This is a breaking change. Because we don't have clear table status interface, docblock and devdocs documentation are the places for communication to developers.

Note: the table status interface looks too loose and does not guarantee anything. May make no sense for different adapters as results will be different (at least in different format). 

### Disable table status cache

`SET GLOBAL INFORMATION_SCHEMA_STATS_EXPIRY = 0;`
The above directive disables status cache completely. The setting was added in MySQL 8.

To guarantee Magento application functioning, the directive should be set from the application during connection to DB.
MySQL < 8 would not accept the setting, so there should be distinction in MySQL versions.

Drawbacks: table status query is never cached. This should not be a big issue taking into account:
- it was not cached in previous MySQL versions
- it is not frequently used. Though import should be performance tested

This solution doesn't solve all the issues with auto_increment.
Table status still returns `null` in case a table is a newly created one with no queries to it.
This case has to be handled on the adapter level. 

Note: MySQL behavior looks like a bug and it may be fixed in the future, which would make the behaviour consistent for different versions.
Though it is not expected that they remove caching layer for the table status. 

## Sources

* https://stackoverflow.com/questions/50749547/how-to-find-next-auto-increment-value-of-a-table-in-mysql-8
* https://bugs.mysql.com/bug.php?id=91038
* https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_information_schema_stats_expiry
 