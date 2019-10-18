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

Introduce new interfaces under Db namespace that would allow to get information about table schema, refactor 
implementations of `\Magento\Framework\Setup\Declaration\Schema\Db\DbSchemaReaderInterface` in declarative schema to use 
interfaces under Db namespace.

Revise implementation of these interfaces to see if anything need to be changed here
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DDLTriggerInterface`
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DbDefinitionProcessorInterface` (see usages of `\Magento\Framework\Setup\Declaration\Schema\Db\DefinitionAggregator`)

### New interfaces and types for managing table schema.

Designing a database abstraction layer is no easy task as one has to have a vast knowledge on various RDBMS and their
interpretation of ANSI-SQL. Especially when it comes to metadata most common RDBMS used in web projects do make use of 
the `information_schema`, yet some big players (e.g. Oracle) do not implement this feature. Since metadata is closely 
coupled with supported data types, one also faces the problem that not all data types defined by ANSI-SQL are supported
by all RDMBS or have very special implementations for the latter, e.g. MySQL / MariaDB does not have a distinct `boolean`
data type but rather map it to `tinyint(1)` which is not the same since it stores values from `0` to `9` (as opposed to
`true` and `false` for an actual `boolean`) but behaves similarly.

Taking into account that Magento makes use of the Zend Framework, I strongly suggest to make use of 
`\Zend\Db\Metadata\MetadataInterface` instead of trying to reinvent the wheel.

Nevertheless, here is a potential draft for implementing a metadata engine.

#### Tables

```
class InformationSchema\Table
{
    /**
     * Returns The name of the schema (database) to which the table belongs.
     *
     * @return string 
     */
    public function getSchema(): string;

    /**
     * Returns the name of the table.
     *
     * @return string 
     */
    public function getName(): string;

    /**
     * Returns the storage engine for the table.
     *
     * Note that not all RDBMS support this, returning `null` indicates whether this is supported.
     *
     * @return EngineEnum
     */
    public function getEngine(): EngineEnum;

    /**
     * Returns the comment used when creating the table.
     *
     * Note that not all RDBMS support this, returning `null` indicates whether this is supported.
     *
     * @return string|null
     */
    public function getComment(): ?string;

    /**
     * Returns the table's default collation.
     *
     * Note that not all RDBMS support this, returning `null` indicates whether this is supported. 
     *
     * @return string|null
     */
    public function getCollation(): ?string;

    /**
     * Returns the table's default character set.
     *
     * Note that not all RDBMS support this, returning `null` indicates whether this is supported.
     *
     * @return string|null
     */
    public function getCharset(): ?string;
}
```
Note: `AUTO_INCREMENT` is omitted intentionally. Auto increment must not be used for the application logic.

```
class InformationSchema\EngineEnum extends \SplEnum
{
    const __default = self::TRANSACTIONAL;

    const IN_MEMORY = 'in memory'
    const NON_TRANSACTIONAL = 'non tranactional';
    const TRANSACTIONAL = 'transactional'
}
```
Note: `SplEnum` is part of the SPLTypes PECL package, it might not be available on all systems, thus we may have to 
create a userland enum class!

#### Columns

```
class InformationSchema\Extra
{
    /**
     * Returns the on-update clause of a column.
     *
     * @return string|null
     */
    public function getOnUpdate(): ?string {}
}
```

```
abstract class InformationSchema\Table\Column
{
    /**
     * Retuns the name of the table to which colum belongs.
     *
     * @return string
     */
    public function getTableName(): string {}

    /**
     * Returns the name of the column.
     *
     * @return string
     */
    public function getName(): string {}

    /**
     * Returns whether column is nullable.
     *
     * @return bool
     */
    public function getIsNullable(): bool {}

    /**
     * Returns the data object for the extra information of a column.
     *
     * Note that not all RDBMS support this, returning `null` indicates whether this is supported.
     *
     * @return ColumnExtraInterface|null
     */
    public function getExtra(): ?ColumnExtraInterface {}

    /**
     * Returns the comment included in the column definition.
     *
     * Note that not all RDBMS support this, returning `null` indicates whether this is supported.
     *
     * @return string|null
     */
    public function getComment(): ?string {}

    /**
     * Returns the default value of the column.
     *
     * @return mixed
     */
    public function getDefaultValue() {}
}
```

##### Boolean types

```
class InformationSchema\Table\BoolColumn extends Column
{
    /**
     * Returns the default value of the column.
     *
     * @return int|null
     */
    public function getDefaultValue(): ?int {}
}
```
Note that MySQL / MariaDB maps this type to `tinyint(1)` thus resulting in an integer type.

##### Numeric types

```
class InformationSchema\Table\IntColumn extends Column
{
    /**
     * Returns the default value of the column.
     *
     * @return int|null
     */
    public function getDefaultValue(): ?int {}
}
```

```
class InformationSchema\Table\SmallIntColumn extends Column
{
    /**
     * Returns the default value of the column.
     *
     * @return int|null
     */
    public function getDefaultValue(): ?int {}
}
```

```
class InformationSchema\Table\BigIntColumn extends Column
{
    /**
     * Returns the default value of the column.
     *
     * @return int|null
     */
    public function getDefaultValue(): ?int {}
}
```

```
class InformationSchema\Table\DecimalColumn extends Column
{
    /**
     * Returns the precision (amount of digits) the column can store.
     *
     * @return int
     */
    public function getPrecision(): int {}

    /**
     * Returns the scale (amount of digits right of the decimal point) the column can store.
     *
     * In case the column has not been defined to have a numeric scale (aka dynamic scale), `null` is returned.
     *
     * @return int|null
     */
    public function getScale(): ?int {}

    /**
     * Returns the default value of the column.
     *
     * @return float|null
     */
    public function getDefaultValue(): ?float {}
}
```
Note that ANSI-SQL defines the exact numerical data types `DECIMAL` and `NUMERIC` as being equivalent. MySQL / MariaDB
sticks to this definitions and maps `NUMERIC` to `DECIMAL`. PostgreSQL behaves the other way around, mapping `DECIMAL`
to `NUMERIC`. This needs to be kept in mind should support for other RDBMS become a thing in Magento.

```
class InformationSchema\Table\RealColumn extends Column
{
    /**
     * Returns the precision (amount of digits) the column can store.
     *
     * @return int
     */
    public function getPrecision(): int {}

    /**
     * Returns the default value of the column.
     *
     * @return float|null
     */
    public function getDefaultValue(): ?float {}
}
```
Note that by default MySQL / MariaDB treats `REAL` as an alias for `DOUBLE PRECISION`. This behaviour can be changed 
by setting the [REAL_AS_FLOAT](https://dev.mysql.com/doc/refman/5.7/en/sql-mode.html#sqlmode_real_as_float) mode.

```
class InformationSchema\Table\DoublePrecisionColumn extends Column
{
    /**
     * Returns the precision (amount of digits) the column can store.
     *
     * @return int
     */
    public function getPrecision(): int {}

    /**
     * Returns the default value of the column.
     *
     * @return float|null
     */
    public function getDefaultValue(): ?float {}
}
```

```
class InformationSchema\Table\FloatColumn extends Column
{
    /**
     * Returns the precision (amount of digits) the column can store.
     *
     * @return int
     */
    public function getPrecision(): int {}

    /**
     * Returns the default value of the column.
     *
     * @return float|null
     */
    public function getDefaultValue(): ?float {}
}
```

##### String types

```
class InformationSchema\Table\CharColumn extends Column
{
    /**
     * Retturns the maximum length in characters.
     */
    public function getCharLength(): int {}

    /**
     * Returns the default value of the column.
     *
     * @return string|null
     */
    public function getDefaultValue(): ?string {}
}
```

```
class InformationSchema\Table\VarcharColumn extends Column
{
    /**
     * Returns the maximum length in characters.
     *
     * @return int
     */
    public function getCharLength(): int {}

    /**
     * Returns the default value of the column.
     *
     * @return string|null
     */
    public function getDefaultValue(): ?string {}
}
```

```
class InformationSchema\Table\ClobColumn extends Column
{
    /**
     * Returns the default value of the column.
     *
     * @return string|null
     */
    public function getDefaultValue(): ?string {}
}
```
Note that MySQL / MariaDB and PostgreSQL use the type `text` for this purpose.

##### Date/Time types

```
class InformationSchema\Table\DateColumn extends Column
{
    /**
     * Returns the default value of the column.
     *
     * @return string|null
     */
    public function getDefaultValue(): ?string {}
}
```

```
class InformationSchema\Table\TimestampColumn extends Column
{
    /**
     * Returns the fractional seconds precision (the amount of digits maitained after the decimal dot of the seconds 
     * value).
     *
     * @return int|null 
     */
    public function getPrecision(): ?int {}

    /**
     * Returns the default value of the column.
     *
     * @return string|null
     */
    public function getDefaultValue(): ?string {}
}
```

##### Binary types

```
class InformationSchema\Table\BinaryColumn extends Column
{
    /**
     * Returns the maximum length in bytes.
     *
     * @return int
     */
    public function getByteLength(): int {}

    /**
     * Returns the default value of the column as binary string.
     *
     * @return string|null
     */
    public function getDefaultValue(): ?string {}
}
```

```
class InformationSchema\Table\BlobColumn extends Column
{
    /**
     * Returns the default value of the column as binary string.
     *
     * @return string|null
     */
    public function getDefaultValue(): ?string {}
}
```

#### Providers

Providers are responsible for mapping RDBMS specific types to the ANSI-SQL column types (e.g. `text` in MySQL is mapped 
to `InformationSchema\Table\ClobColumn`). 

```
interface InformationSchema\Table\ColumnProviderInterface
{
    /**
     * @return Column[]
     */
    public function getColumns(string $tableName): array;
}
```

```
interface InformationSchema\TableProviderInterface
{
    public function getTableByName(string $tableName): Table;
}
```

```
use Magento\Framework\DB\Adapter\AdapterInterface;

class InformationSchema\ConnectionResolver
{
    public function getConnectionForTable(string $tableName): AdapterInterface {}
}
```

```
abstract class InformationSchema\AbstractTableProvider implements TableProviderInterface
{
    public function __construct(ConnectionResolver $connectionResolver)
    {
        $this->connectionResolver = $connectionResolver;
    }
}
```

```
abstract class InformationSchema\AbstractColumnProvider implements ColumnProviderInterface
{
    public function __construct(ConnectionResolver $connectionResolver)
    {
        $this->connectionResolver = $connectionResolver;
    }
}
```

### Profiles

The above interfaces will have multiple implementations, and the framework should be switching between them depending on
the RDBMS it works with.
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

A user (SI) also can specify the profile manually. This can be helpful in case a custom distributive of RDBMS is used 
and Magento application can't identify correctly which profile to use.
The following tools should provide an ability for the user to specify the profile:

* CLI installation command
* Web installation UI
* CLI command to update the profile (or manually update `env.php`)

When Magento application is running and needs to read DB Information Schema, necessary implementation is selected based 
on the profile in `env.php`.

### Alternative approach

Alternative approach is to reuse existing interfaces and add replaceability on the level of declarative schema.

`\Magento\Framework\Setup\Declaration\Schema\Db\SchemaBuilder` should accept 
`\Magento\Framework\Setup\Declaration\Schema\Db\DbSchemaReaderFactory` that would allow to create database specific 
implementations of `DbSchemaReaderInterface` for given connection. 
`\Magento\Framework\Setup\Declaration\Schema\Db\DbSchemaReaderFactory` will depend on 
`\Magento\Framework\App\DeploymentConfig` to get configuration for connection, `model` will contain name of database 
adapter.

Similar approach needs to be taken for clients of the following interfaces
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DDLTriggerInterface`
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DbSchemaWriterInterface`
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DbDefinitionProcessorInterface` (see usages of `\Magento\Framework\Setup\Declaration\Schema\Db\DefinitionAggregator`)

### Replaceability of `\Magento\Framework\DB\Adapter\AdapterInterface`

Not needed for declarative schema, but with this refactoring would also make sense to add ability to have different 
adapters for different databases.

`\Magento\Framework\Model\ResourceModel\Type\Db\ConnectionFactoryInterface` currently returns instance of 
`Magento\Framework\App\ResourceConnection\ConnectionAdapterInterface` that is defined by preference, similarly to 
approach above it should resolve database specific adapter based on configuration.

`model` is the name of the parameter that is supposed to be used to configure database specific adapters. Somehow 
functionality behind it is absent.

It has te default value of `mysql4`. I propose to add 2 values: `mysql` and `mariadb` and have `mysql4` be an alias for 
`mysql`.

## Considerations

`\Magento\Framework\DB\Adapter\AdapterInterface` has methods like `describeTable` which return raw values from database 
engine. This is a leaky interface. We should have interfaces that return normalized data and are compatible with all 
databases. Methods that don't return normalized data need to be deprecated on `\Magento\Framework\DB\Adapter\AdapterInterface`.

## Resources

* https://en.wikipedia.org/wiki/Information_schema
* [MariaDB Information Schema COLUMNS](https://mariadb.com/kb/en/library/information-schema-columns-table/)
* [MySQL Information Schema COLUMNS](https://dev.mysql.com/doc/refman/5.7/en/columns-table.html)
* [MDEV-13132](https://jira.mariadb.org/browse/MDEV-13132) - ticket in MariaDB issue tracker, where they changed behavior of `COLUMN_DEFAULT`
