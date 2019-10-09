# Declarative schema database compatibility

## About

Declarative schema relies on querying metadata from the DB.
The query itself and format of the result may differ for different RDBMS systems.
For example, here is the query from `Magento\Framework\Setup\Declaration\Schema\Db\MySQL\DbSchemaReader::readColumns`

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

For `COLUMN_DEFAULT`, PHP PDO connector returns `NULL` for MySQL and `"NULL"` (string) for MariaDB 10.2.7+.

## Design

Introduce new interfaces under Db namespace that would allow to get information about table schema, refactor implementations of `\Magento\Framework\Setup\Declaration\Schema\Db\DbSchemaReaderInterface` in declarative schema to use interfaces under Db namespace.

Revise implementation of these interfaces to see if anything need to be changed here
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DDLTriggerInterface`
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DbDefinitionProcessorInterface` (see usages of `\Magento\Framework\Setup\Declaration\Schema\Db\DefinitionAggregator`)

New interfaces and types for managing table schema.

```
class InformationSchema\Table
{
    public function getSchema(): string {}
    public function getName(): string {}
    public function getEngine(): string {}
    public function getComment(): string {}
    public function getCollation(): string {}
    public function getCharset(): string {}
}
```
Note: auto increment is omitted intentionally. Auto increment should not be used for the application logic.

```
class InformationSchema\Extra
{
    public function getOnUpdate(): string {}
}
```

```
abstract class InformationSchema\Table\Column
{
    public function getTableName(): string {}
    public function getName(): string {}
    public function getIsNullable(): bool {}
    public function getExtra(): Extra {}
    public function getComment() {}
    public function getDefaultValue(): ?string {}
}
```

```
class InformationSchema\Table\VarcharColumn extends Column
{
    public const TYPE = 'VARCHAR';

    public function getCharLength(): int {}
    public function getCharsetName(): string {}
    public function getCollationName(): string {}
}
```

```
class InformationSchema\Table\IntColumn extends Column
{
    public const TYPE = 'INT';

    public function isUnsigned(): bool {}
    public function getPadding(): int {}
    public function getType(): IntEnum {}
}
```

```
class InformationSchema\Table\FloatColumn extends Column
{
    public const TYPE = 'FLOAT';

    public function getDefaultValue() {}
    public function getScale(): int {}
    public function getPrecision(): int {}
    public function getNumericScale(): int {}
}
```

```
class InformationSchema\Table\DateColumn extends Column
{
    public const TYPE = 'DATE';

    public function getDefaultValue(): string {}
    public function getDatetimePrecision(): int {}
}
```

```
interface InformationSchema\Table\ColumnProviderInterface
{
    /**
     * @return Column[]
     */
    public function getColumns(string $tableName, string $connectionName);
}
```

// Consider removing $connectionName from the interface (move to constructor via resolver that returns by table name)
```
interface InformationSchema\TableProviderInterface
{
    public function getTableByName(string $tableName, string $connectionName): Table;
    public function getAllTables(string $connectionName): Table; // check if we use it anywhere
}
```

### Profiles

The above interfaces will have multiple implementations, and the framework should be switching between them depending on the RDBMS it works with.
The framework decides which implementations to use based on declared profiles.
The profiles are declared in `di.xml` or as a separate configuration, with the following structure:

```
information_schema_profiles:
  mysql-5.6:
    table_provider: InformationSchema\TableProviderMysql56
    column_provider: InformationSchema\Table\ColumnProviderMysql56
  mysql-8:
    table_provider: InformationSchema\TableProviderMysql8
    column_provider: InformationSchema\Table\ColumnProviderMysql8
  mariadb-10.2.0:
    table_provider: InformationSchema\TableProviderMariaDb1020
    column_provider: InformationSchema\Table\ColumnProviderMariaDb1020
  mariadb-10.2.3:
    table_provider: InformationSchema\TableProviderMariaDb1020 // note that it's the same as for 'mariadb-10.2.0' - example of nochanges in TableProvider interface for this version
    column_provider: InformationSchema\Table\ColumnProviderMariaDb1023
  mariadb-10.2.7:
    table_provider: InformationSchema\TableProviderMariaDb1020 // note that it's the same as for 'mariadb-10.2.0' - example of nochanges in TableProvider interface for this version
    column_provider: InformationSchema\Table\ColumnProviderMariaDb1027
information_schema_default_profile: mysql-8
```

During application installation and upgrade, the application tries to determine profile based on RDBMS used.
For example, if Magento application is being installed on MariaDB 10.2.5, 'mariadb-10.2.3' profile is selected.
If the application can't determine which RDBMS is used, `information_schema_default_profile` is selected.
This is repeated for each connection.

The profile is recorded in `env.php` as part of installation/upgrade process.

A user (SI) also can specify the profile manually. This can be helpful in case a custom distributive of RDBMS is used and Magento application can't identify correctly which profile to use.
The following tools should provide an ability for the user to specify the profile:

* CLI installation command
* Web installation UI
* CLI command to update the profile (or manually update `env.php`)

When Magento application is running and needs to read DB Information Schema, necessary implementation is selected based on the profile in `env.php`.

### Alternative approach

Alternative approach is to reuse existing interfaces and add replaceability on the level of declarative schema.

`\Magento\Framework\Setup\Declaration\Schema\Db\SchemaBuilder` should accept `\Magento\Framework\Setup\Declaration\Schema\Db\DbSchemaReaderFactory` that would allow to create database specific implementations of `DbSchemaReaderInterface` for given connection. `\Magento\Framework\Setup\Declaration\Schema\Db\DbSchemaReaderFactory` will depend on `\Magento\Framework\App\DeploymentConfig` to get configuration for connection, `model` will contain name of database adapter.

Similar approach need to be taken for clients of the following interfaces
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DDLTriggerInterface`
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DbSchemaWriterInterface`
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DbDefinitionProcessorInterface` (see usages of `\Magento\Framework\Setup\Declaration\Schema\Db\DefinitionAggregator`)

### Replaceability of `\Magento\Framework\DB\Adapter\AdapterInterface`

Not needed for declarative schema, but with this refactoring would also make sense to add ability to have different adapters for different databases.

`\Magento\Framework\Model\ResourceModel\Type\Db\ConnectionFactoryInterface` currently returns instance of `Magento\Framework\App\ResourceConnection\ConnectionAdapterInterface` that is defined by preference, similarly to approach above it should resolve database specific adapter based on configuration.

`model` is the name of parameter that supposed to be used to configure database specific adapters. Somehow functionality behind it is absent.

It has default value of `mysql4`. I propose to add 2 values: `mysql` and `mariadb` and have `mysql4` be alias for `mysql`.

## Considerations

`\Magento\Framework\DB\Adapter\AdapterInterface` has methods like `describeTable` which return raw values from database engine. This is leaky interface. We should have interfaces that return normalized data and are compatible with all databases. Methods that don't return normalized data need to be deprecated on `\Magento\Framework\DB\Adapter\AdapterInterface`.

## Resources

* https://en.wikipedia.org/wiki/Information_schema
* [MariaDB Information Schema COLUMNS](https://mariadb.com/kb/en/library/information-schema-columns-table/)
* [MySQL Information Schema COLUMNS](https://dev.mysql.com/doc/refman/5.7/en/columns-table.html)
* [MDEV-13132](https://jira.mariadb.org/browse/MDEV-13132) - ticket in MariaDB issue tracker, where they changes behavior of `COLUMN_DEFAULT`
