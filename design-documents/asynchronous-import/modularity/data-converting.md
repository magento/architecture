# Data converting before import

## Modules

- `AsynchronousImportDataConvertingApi`
- `AsynchronousImportDataConverting`

## Implementation

### API

```php
namespace Magento\AsynchronousImportDataConvertingApi\Api;

use Magento\AsynchronousImportDataConvertingApi\Api\Data\ConvertingRuleInterface;
use Magento\Framework\Validation\ValidationException;

/**
 * Apply converting rules to import data operation. Uses differect strategies for rules applying
 * Responsible for data changing before import
 *
 * @api
 */
interface ApplyConvertingRulesInterface
{
    /**
     * Apply converting rules to import data operation. Uses differect strategies for rules applying
     *
     * @param array $importData
     * @param ConvertingRuleInterface[] $convertingRules
     * @return array
     * @throws ValidationException
     * @throws ApplyConvertingRulesException
     */
    public function execute(
        array $importData,
        array $convertingRules
    ): array;
}
```

```php
namespace Magento\AsynchronousImportDataConvertingApi\Api\Data;

/**
 * Describes how to change data before import
 *
 * @api
 */
interface ConvertingRuleInterface
{
    /**
     * Get rule identifier
     *
     * @return string
     */
    public function getIdentifier(): string;

    /**
     * Get rule parameters
     *
     * @return string[]|null Null value is needed fro SOAP parser
     */
    public function getParameters(): array;

    /**
     * Get sort
     *
     * @return int|null
     */
    public function getSort(): ?int;

    /**
     * Get apply to
     *
     * @return string[]|null Null value is needed fro SOAP parser
     */
    public function getApplyTo(): array;
}
```

### Extension points

```php
namespace Magento\AsynchronousImportDataConvertingApi\Model;

use Magento\AsynchronousImportDataConvertingApi\Api\Data\ConvertingRuleInterface;

/**
 * Extension point for adding converting rule applying algorithms
 * Represents concrete strategy
 *
 * @api
 */
interface ApplyConvertingRuleStrategyInterface
{
    /**
     * Converting rule applying strategy
     *
     * @param array $importData
     * @param ConvertingRuleInterface $convertingRule
     * @return array
     */
    public function execute(
        array $importData,
        ConvertingRuleInterface $convertingRule
    ): array;
}
```

```php
namespace Magento\AsynchronousImportDataConvertingApi\Model;

/**
 * Extension point for adding converting rule validators via DI configuration
 *
 * @api
 */
class ConvertingRuleValidatorChain implements ConvertingRuleValidatorInterface
{
    ...
}
```

```php
namespace Magento\AsynchronousImportDataConvertingApi\Model;

use Magento\AsynchronousImportDataConvertingApi\Api\Data\ConvertingRuleInterface;
use Magento\Framework\Validation\ValidationResult;

/**
 * Extension point for adding converting rule validators
 *
 * @api
 */
interface ConvertingRuleValidatorInterface
{
    /**
     * Validate converting rule
     *
     * @param ConvertingRuleInterface $convertingRule
     * @return ValidationResult
     */
    public function validate(ConvertingRuleInterface $convertingRule): ValidationResult;
}
```
