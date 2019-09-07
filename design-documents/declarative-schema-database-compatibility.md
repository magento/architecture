# Declarative schema database compatibility

## About

Declarative schema relies on querying data that may have different format depending on database. For example, here is the query from `Magento\Framework\Setup\Declaration\Schema\Db\MySQL\DbSchemaReader::readColumns`

```
SELECT 
    COLUMN_NAME as 'name',
    COLUMN_DEFAULT as 'default',
    DATA_TYPE as 'type',
    IS_NULLABLE as 'nullable',
    COLUMN_TYPE as 'definition',
    EXTRA as 'extra',
    COLUMN_COMMENT as 'comment'
FROM
    information_schema.COLUMNS
WHERE
    TABLE_SCHEMA = 'magento'
AND
    TABLE_NAME = 'store_website'
ORDER BY
    ORDINAL_POSITION ASC;
```

MySQL returns NULL for COLUMN_DEFAULT and MariaDB returns NULL as a string.

## Design

Introduce concept of database adapters to allow normalize values in declarative schema for different databases.

`\Magento\Framework\Setup\Declaration\Schema\Db\SchemaBuilder` should accept `\Schema\Db\MySQL\DbSchemaReaderFactory` that would allow to create database specific implementations of `DbSchemaReaderInterface` for given connection. `\Schema\Db\MySQL\DbSchemaReaderFactory` will depend on `\Magento\Framework\App\DeploymentConfig` to get configuration for connection, `model` will contain name of database adapter.

Similar approach need to be taken for clients of the following interfaces
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DDLTriggerInterface`
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DbSchemaWriterInterface`
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DbDefinitionProcessorInterface` (see usages of `\Magento\Framework\Setup\Declaration\Schema\Db\DefinitionAggregator`)

Not needed for declarative schema, but with this refactoring would also make sense to add ability have different adapters for different databases.

`\Magento\Framework\Model\ResourceModel\Type\Db\ConnectionFactoryInterface` currently returns instance of `Magento\Framework\App\ResourceConnection\ConnectionAdapterInterface` that is defined by preference, similarly to approach above it should resolve database specific adapter based on configuration.

`model` is the name of parameter that supposed to be used to configure database specific adapters. Somehow functionality behind it is absent.

It has default value of `mysql4`. I propose to add 2 values: `mysql` and `mariadb` and have `mysql4` be alias for `mysql`.

## Considerations

`\Magento\Framework\DB\Adapter\AdapterInterface` has methods like `describeTable` which return raw values from database engine. This is leaky interface. Ideally we should have an interface that would return normalized data and be compatible with all databases. This is a much larger change and might be considered for minor release.
