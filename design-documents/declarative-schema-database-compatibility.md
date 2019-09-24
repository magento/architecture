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

Introduce new interfaces under Db namespace that allow to manage table schema, refactor implementations of the following interfaces in declarative schema to use interfaces under Db namespace.

Similar approach need to be taken for clients of the following interfaces
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DbSchemaReaderInterface`
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DbSchemaWriterInterface`

Revise implementation of these interfaces to see if anything need to be changed here
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DDLTriggerInterface`
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DbDefinitionProcessorInterface` (see usages of `\Magento\Framework\Setup\Declaration\Schema\Db\DefinitionAggregator`)

New interfaces and types for managing table schema.

```
class ConstraintType extends \SplEnum
{
    const primary = 'primary';
    const unique = 'unique';
    const auto_increment = 'auto_increment';
}
```

```
class IndexType extends \SplEnum
{
    // For MySQL, Oracle, MS SQL and Postgres will use btree
    const normal = 'normal';

    // For MySQL and MS SQL fulltext, one of text indexes for Oracle and text for Postgres
    const text = 'text';
}
```

```
class OnDelete extends \SplEnum
{
    const cascade = 'cascade';
}
```

```
class CollationType extends \SplEnum
{
    const utf8 = 'utf8';
}
```

```
class Constraint
{
    public function getName(): string {}

    /**
     * @return string[]
     */
    public function getColumns() {}

    public function getType(): ConstraintType {}
}
```

```
class Index
{
    public function getName(): string {}

    /**
     * @return string[]
     */
    public function getColumns() {}

    public function getType(): IndexType {}
}
```

```
class ForeignKey
{
    public function getType(): string {}
    public function getName(): string {}
    public function getColumn(): string {}
    public function getReferenceTable(): string {}
    public function getReferenceColumn(): string {}
    public function getOnDelete(): OnDelete {}
}
```

```
/**
 * There is no option for engine. Do we want to support different engines, if so how to abstract them, normal/memory?
 */
class TableOptions
{
    public function getName(): string {}
    public function getComment(): string {}

    /**
     * Potentially can be removed as well?
     */
    public function getCollation(): CollationType {}
}
```

```
class Column
{
    public function getName(): string {}
    public function getType() {}
    public function getIsNullable() {}
    public function getExtra() {}
    public function getComment() {}
}
```

```
class IntegerColumn extends Column
{
    public function getDefaultValue() {}
    public function isUnsigned(): bool {}
    public function getPadding(): int {}
}
```

```
class DecimalColumn extends Column
{
    public function getDefaultValue() {}
    public function getScale(): int {}
    public function getPrecision(): int {}
}
```

```
class DateTimeColumn extends Column
{
    public function getDefaultValue(): string {}
}
```

```
interface TableInformationInterface
{
    /**
     * @return Constraint[]
     */
    public function getConstraints($tableName, $resource);

    /**
     * @return ForeignKey[]
     */
    public function getReferences($tableName, $resource);

    /**
     * @return Index[]
     */
    public function getIndexes($tableName, $resource);

    /**
     * @return Column
     */
    public function getColumns($tableName, $resource);

    public function getTableOptions($tableName, $resource): TableOptions;
}
```

```
interface TableManagementInterface
{
    public function createTable(TableOptions $tableOptions, $resource);
    public function dropTable($tableName, $resource);

    /**
     * @param string $tableName
     * @param Column|IntegerColumn|DecimalColumn|DateTimeColumn $column
     * @param $resource
     */
    public function addColumn($tableName, Column $column, $resource);
    public function dropColumn($tableName, $columnName, $resource);
    public function addIndex($tableName, Index $index, $resource);
    public function dropIndex($tableName, $indexName, $resource);
    public function addConstraint($tableName, Constraint $constraint, $resource);
    public function dropConstraint($tableName, $constraintName, $resource);
    public function addForeignKey($tableName, ForeignKey $foreignKey, $resource);
    public function dropForeignKey($tableName, $foreignKeyName, $resource);
    public function updateTableOptions($tableName, TableOptions $tableOptions, $resource);
}
```

```
class TableDiff
{
    public function getName(): string {}

    /**
     * Column[]|IntegerColumn[]|DecimalColumn[]|DateTimeColumn[]
     */
    public function getAddedColumns() {}

    /**
     * Column[]|IntegerColumn[]|DecimalColumn[]|DateTimeColumn[]
     */
    public function getChangedColumns() {}

    /**
     * Column[]|IntegerColumn[]|DecimalColumn[]|DateTimeColumn[]
     */
    public function getRemovedColumns() {}

    /**
     * Index[]
     */
    public function getAddedIndexes() {}

    /**
     * Index[]
     */
    public function getChangedIndexes() {}

    /**
     * Index[]
     */
    public function getRemovedIndexes() {}

    /**
     * Constraint[]
     */
    public function getAddedConstraints() {}

    /**
     * Constraint[]
     */
    public function getChangedConstraints() {}

    /**
     * Constraint[]
     */
    public function getRemovedConstraints() {}

    /**
     * ForeignKey[]
     */
    public function getAddedForeignKeys() {}

    /**
     * ForeignKey[]
     */
    public function getChangedForeignKeys() {}

    /**
     * ForeignKey[]
     */
    public function getRemovedForeignKeys() {}

    public function getFromTable(): string {}
}
```

```
/**
 * Alternative interface
 */
interface TableManagementInterface
{
    public function createTable(TableOptions $tableOptions, $resource);
    public function dropTable($tableName, $resource);
    public function alterTable($tableName, $resource, TableDiff $tableDiff);
}
```

### Alternative approach

Alternative approach is to reuse existing interfaces and add replaceability on the level of declarative schema.

`\Magento\Framework\Setup\Declaration\Schema\Db\SchemaBuilder` should accept `\Magento\Framework\Setup\Declaration\Schema\Db\DbSchemaReaderFactory` that would allow to create database specific implementations of `DbSchemaReaderInterface` for given connection. `\Magento\Framework\Setup\Declaration\Schema\Db\DbSchemaReaderFactory` will depend on `\Magento\Framework\App\DeploymentConfig` to get configuration for connection, `model` will contain name of database adapter.

Similar approach need to be taken for clients of the following interfaces
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DDLTriggerInterface`
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DbSchemaWriterInterface`
1. `\Magento\Framework\Setup\Declaration\Schema\Db\DbDefinitionProcessorInterface` (see usages of `\Magento\Framework\Setup\Declaration\Schema\Db\DefinitionAggregator`)

### Replaceability of `\Magento\Framework\DB\Adapter\AdapterInterface`

Not needed for declarative schema, but with this refactoring would also make sense to add ability have different adapters for different databases.

`\Magento\Framework\Model\ResourceModel\Type\Db\ConnectionFactoryInterface` currently returns instance of `Magento\Framework\App\ResourceConnection\ConnectionAdapterInterface` that is defined by preference, similarly to approach above it should resolve database specific adapter based on configuration.

`model` is the name of parameter that supposed to be used to configure database specific adapters. Somehow functionality behind it is absent.

It has default value of `mysql4`. I propose to add 2 values: `mysql` and `mariadb` and have `mysql4` be alias for `mysql`.

## Considerations

`\Magento\Framework\DB\Adapter\AdapterInterface` has methods like `describeTable` which return raw values from database engine. This is leaky interface. We should have interfaces that return normalized data and are compatible with all databases. Methods that don't return normalized data need to be deprecated on `\Magento\Framework\DB\Adapter\AdapterInterface`.

`engine` field in declarative schema need to be deprecated.
