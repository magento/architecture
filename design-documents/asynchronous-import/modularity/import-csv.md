# Start Import based on CSV data (main entry point)

## Modules

- `AsynchronousImportCsvApi`
- `AsynchronousImportCsv`

## Implementation

### API

```php
namespace Magento\AsynchronousImportCsvApi\Api;

use Magento\AsynchronousImportDataConvertingApi\Api\ApplyConvertingRulesException;
use Magento\AsynchronousImportCsvApi\Api\Data\CsvFormatInterface;
use Magento\AsynchronousImportDataConvertingApi\Api\Data\ConvertingRuleInterface;
use Magento\AsynchronousImportDataExchangingApi\Api\Data\ImportInterface;
use Magento\AsynchronousImportDataExchangingApi\Api\ImportDataExchangeException;
use Magento\AsynchronousImportSourceDataRetrievingApi\Api\Data\SourceInterface;
use Magento\AsynchronousImportSourceDataRetrievingApi\Api\SourceDataRetrievingException;
use Magento\Framework\Validation\ValidationException;

/**
 * Start import operation
 *
 * @api
 */
interface StartImportInterface
{
    /**
     * Start import operation
     *
     * @param SourceInterface $source Describes how to retrieve data from data source
     * @param ImportInterface $import Describes how to import data
     * @param CsvFormatInterface|null $format Describes how to parse data
     * @param ConvertingRuleInterface[] $convertingRules Describes how to change data before import
     * @return string
     * @throws ValidationException
     * @throws SourceDataRetrievingException
     * @throws ApplyConvertingRulesException
     * @throws ImportDataExchangeException
     */
    public function execute(
        SourceInterface $source,
        ImportInterface $import,
        CsvFormatInterface $format = null,
        array $convertingRules = []
    ): string;
}
```

```php
namespace Magento\AsynchronousImportCsvApi\Api\Data;

use Magento\Framework\Api\ExtensibleDataInterface;

/**
 * Describes how to parse data
 *
 * @api
 */
interface CsvFormatInterface extends ExtensibleDataInterface
{
    /**
     * Get CSV Escape
     *
     * @return string|null
     */
    public function getEscape(): ?string;

    /**
     * Get CSV Enclosure
     *
     * @return string|null
     */
    public function getEnclosure(): ?string;

    /**
     * Get CSV Delimiter
     *
     * @return string|null
     */
    public function getDelimiter(): ?string;

    /**
     * Get Multiple Value Separator
     *
     * @return string|null
     */
    public function getMultipleValueSeparator(): ?string;

    /**
     * Get existing extension attributes object
     *
     * Used fully qualified namespaces in annotations for proper work of extension interface/class code generation
     *
     * @return \Magento\AsynchronousImportCsvApi\Api\Data\CsvFormatExtensionInterface|null
     */
    public function getExtensionAttributes(): ?CsvFormatExtensionInterface;
}
```

### Extension points

```php
namespace Magento\AsynchronousImportCsvApi\Model;

use Magento\AsynchronousImportCsvApi\Api\Data\CsvFormatInterface;

/**
 * Extension point for data parsing (based on passed CSV format)
 *
 * @api
 */
interface DataParserInterface
{
    /**
     * Extension point for data parsing (based on passed CSV format)
     *
     * @param array $data
     * @param CsvFormatInterface|null $csvFormat
     * @return array
     */
    public function execute(array $data, CsvFormatInterface $csvFormat = null): array;
}
```
