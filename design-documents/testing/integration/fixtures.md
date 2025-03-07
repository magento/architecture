### What we would like to improve?
Recently, we introduced [parameterized fixture](https://devdocs.magento.com/guides/v2.4/test/integration/parameterized_data_fixture.html) for integration and api-functional test that accepts parameters directly from [@magentoDataFixture](https://devdocs.magento.com/guides/v2.4/test/integration/annotations/magento-data-fixture.html) annotation.

We enhanced `@magentoDataFixture` annotation format to support additional information that contains the parameters and the [alias](https://devdocs.magento.com/guides/v2.4/test/integration/annotations/magento-data-fixture.html#fixture-alias) of a fixture.
However, the current implementation has some imperfections:
- Extends `@magentoDataFixture` annotation format in order to pass additional information
- Does not use PHP native (built-in) syntax
- Cannot use constants. Even the fixture class name has to be passed in the form of String
- Cannot be split into multiple lines for readability
- Requires [@magentoDataFixtureDataProvider](https://devdocs.magento.com/guides/v2.4/test/integration/annotations/magento-data-fixture-data-provider.html) for more advanced configuration

#### Example

```php
class AddSimpleProductToCartSingleMutationTest extends GraphQlAbstract
{
    /**
     * @magentoApiDataFixture Magento\Catalog\Test\Fixture\Product as:product1
     * @magentoApiDataFixture Magento\Catalog\Test\Fixture\Product as:product2
     * @magentoApiDataFixture Magento\Catalog\Test\Fixture\Product as:product3
     * @magentoApiDataFixture Magento\Quote\Test\Fixture\GuestCart as:cart
     * @magentoApiDataFixture Magento\Quote\Test\Fixture\AddProductToCart as:cartItem1
     * @magentoApiDataFixture Magento\Quote\Test\Fixture\AddProductToCart as:cartItem2
     * @magentoDataFixtureDataProvider {"cartItem1":{"cart_id":"$cart.id$","product_id":"$product1.id$","qty":1}}
     * @magentoDataFixtureDataProvider {"cartItem2":{"cart_id":"$cart.id$","product_id":"$product2.id$","qty":1}}
     */
    public function testAddMultipleProductsToNotEmptyCart(): void
    {
        $product1 = $this->fixtures->get('product1');
        $product2 = $this->fixtures->get('product2');
        $product3 = $this->fixtures->get('product3');
        $cart = $this->fixtures->get('cart');
        //...
    }
}
```

### How we can achieve the goal?
PHP 8.0 introduced a new feature ([Attributes](https://www.php.net/manual/en/language.attributes.overview.php)) that we can utilize to generate fixtures instead of `@magentoDataFixture`.

#### Implementation
Create PHP attribute equivalent for all existing DocBlock annotations used in integration and functional tests

- `@magentoAppIsolation` -> `AppIsolation`
- `@magentoDbIsolation` -> `DbIsolation`
- `@magentoDataFixtureBeforeTransaction` ->`DataFixtureBeforeTransaction`
- `@magentoDataFixture` -> `DataFixture` (1)
- `@magentoApiDataFixture` -> `DataFixture` (1)
- `@magentoIndexerDimensionMode`-> `IndexerDimensionMode`
- `@magentoComponentsDir` -> `ComponentsDir`
- `@magentoAppArea` -> `AppArea`
- `@magentoCache` -> `Cache`
- `@magentoAdminConfigFixture` -> `Config` (2)
- `@magentoConfigFixture` -> `Config` (2)

(1)  Both `magentoDataFixture` and `magentoApiDataFixture` can be replaced with `DataFixture` attribute. Currently we use `magentoDataFixture` in integration tests and `magentoApiDataFixture` in functional api tests (WebAPI). Both executes data fixtures except that `magentoApiDataFixture` does not trigger db transaction (db isolation) by default (without explicitly enabling db isolation with @magentoDbIsolation) whereas `magentoDataFixture` does.

All it takes to make this work is to [replace](https://github.com/magento-commerce/magento2ce/blob/2.4-develop/dev/tests/integration/framework/Magento/TestFramework/Bootstrap/DocBlock.php#L67-L74) `\Magento\TestFramework\Annotation\DataFixture` with `\Magento\TestFramework\Annotation\ApiDataFixture` in [WebapiDocBlock](https://github.com/magento-commerce/magento2ce/blob/2.4-develop/dev/tests/api-functional/framework/Magento/TestFramework/Bootstrap/WebapiDocBlock.php#L16) and override `\Magento\TestFramework\Annotation\AbstractDataFixture::getDbIsolationState` in `\Magento\TestFramework\Annotation\ApiDataFixture` (ApiDataFixture extends DataFixture which also extends AbstractDataFixture)  to return `[disabled]` instead of `NULL` in case the db isolation is not explicitly defined. This will prevent the transaction from auto starting in [\Magento\TestFramework\Annotation\DataFixture::startTestTransactionRequest](https://github.com/magento-commerce/magento2ce/blob/2.4-develop/dev/tests/integration/framework/Magento/TestFramework/Annotation/DataFixture.php#L32) unless explicitly enabled with `@magentoDbIsolation`.

<img width="1073" alt="Screen Shot 2022-05-25 at 9 41 37 AM" src="https://user-images.githubusercontent.com/8973208/170289644-f0d661f0-04df-49d3-acd9-88c00f6c23c2.png">
<img width="1072" alt="Screen Shot 2022-05-25 at 9 41 13 AM" src="https://user-images.githubusercontent.com/8973208/170289671-c6c9da54-b447-431e-9a88-144b25d0976e.png">


(2) Both `magentoAdminConfigFixture` and `magentoConfigFixture` can be replaced with `Config` attribute.

**Example**:

```php
namespace Magento\TestFramework\Fixture;

use Attribute;

#[Attribute(Attribute::TARGET_METHOD | Attribute::TARGET_CLASS | Attribute::IS_REPEATABLE)]
class DataFixture
{
    /**
     * @param string $type
     * @param array $data
     * @param string|null $as
     */
    public function __construct(
        public string $type,
        public array $data = [],
        public ?string $as = null
    ) {
    }
}
```

```php
#[Attribute(Attribute::TARGET_METHOD | Attribute::TARGET_CLASS | Attribute::IS_REPEATABLE)]
class Config
{
    /**
     * @param string $path
     * @param mixed $value
     * @param string $scopeType
     * @param string|null $scopeValue
     */
    public function __construct(
        public string $path,
        public mixed $value,
        public string $scopeType = ScopeConfigInterface::SCOPE_TYPE_DEFAULT,
        public ?string $scopeValue = null
    ) {
    }
}
```

```php
class AddSimpleProductToCartSingleMutationTest extends GraphQlAbstract
{
    #[
        Config(Configuration::XML_PATH_SHOW_OUT_OF_STOCK, 1, ScopeInterface::SCOPE_STORE, 'default'),
        DataFixture(ProductFixture::class, as: 'product1'),
        DataFixture(ProductFixture::class, as: 'product2'),
        DataFixture(ProductFixture::class, as: 'product3'),
        DataFixture(GuestCartFixture::class, as: 'cart'),
        DataFixture(AddProductToCartFixture::class, ['cart_id' => '$cart.id$', 'product_id' => '$product1.id$']),
        DataFixture(AddProductToCartFixture::class, ['cart_id' => '$cart.id$', 'product_id' => '$product2.id$']),
    ]
    public function testAddMultipleProductsToNotEmptyCart(): void
    {
        $product1 = $this->fixtures->get('product1');
        $product2 = $this->fixtures->get('product2');
        $product3 = $this->fixtures->get('product3');
        $cart = $this->fixtures->get('cart');
        //...
    }
}
```

#### Pros
- Best time to make such changes as Parameterized Fixture is not released yet
- Fixture classes can be imported and referenced using constant (Fixture::class)
- PHP Native syntax
- Easy to maintain and extend
#### Cons
- Require PHP >= 8.0 to run tests

### Justification
Less custom code and more PHP built-in syntax
