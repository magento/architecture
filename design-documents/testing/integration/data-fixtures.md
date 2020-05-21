## Problem
When it comes to write a data fixture for your test case, the first thing you would do is to search for existing data fixture you can reuse in your test case. Most of the time you will find such data fixture that almost meets the requirements of your test case except that it's missing something very important to your test case.
Therefore, you end up writing a new data fixture for your test case that is 99% a copy of existing one.
Here is a real example:

- dev/tests/integration/testsuite/Magento/Catalog/_files/product_virtual.php
- dev/tests/integration/testsuite/Magento/Catalog/_files/product_virtual_in_stock.php
- dev/tests/integration/testsuite/Magento/Catalog/_files/product_virtual_out_of_stock.php

All these 3 files create a virtual product except one marks the product as out of stock and the other assigns different quantity to the product.
So if you need a virtual product with available quantity 1, you will probably have to:

- Write a new data fixture (copy-past)
- Reuse one of these data fixtures in a new data fixture file and edit the quantity there.
- Reuse one of these data fixtures in your test case and edit the quantity in the test directly.
 
## Solution

Extend the format of data fixture annotation as following:

```php
class QuoteTest extends \PHPUnit\Framework\TestCase
{
    /**
     * @magentoDataFixture Magento/Catalog/_files/product_virtual.php as virtualProduct
     * @magentoDataFixture Magento/Customer/_files/customer.php as customer
     * @magentoDataFixture Magento/Customer/_files/customer_address.php as customerAddress require customer
     * @magentoDataFixture Magento/Checkout/_files/quote.php as quote require customer, virtualProduct
     */
    public function testCollectTotals(): void
    {
    }

    public function collectTotalsDataFixture(): array
    {
        return [
            'virtualProduct' => [
                'special_price' => 15
            ],
            'customerAddress' => [
                'telephone' => '+15129000201',
                '@customer' => 'customer',
            ],
            'quote' => [
                'is_multi_shipping' => 1,
                '@customer' => 'customer',
                '@products[]' => [
                    'virtualProduct' => [
                        'qty' => 2
                    ]
                ],
            ],
        ];
    }
}
```
With the `as` and `require` keywords, we'll be able to inject data and resolve dependencies between fixtures.
The `as` keyword is used for aliasing a fixture and the `require` keyword is used to declare fixtures dependencies on each other using fixtures aliases.

The method `collectTotalsDataFixture` deducted from the test name with suffix `DataFixture` is the data provider for all fixtures defined for test case. Test class fixture data provider method would be `dataFixture`. The data fixture provider should return an associative array with fixtures aliases as key and data to overwrite the corresponding fixture data.
The data fixture provider uses `@` before attribute name to resolve dependencies. The `@` prefixed attribute value must be an associative array which keys are dependent fixtures aliases and values are an array containing the relationship data.

In the example above, `Magento/Catalog/_files/product_virtual.php`, `Magento/Customer/_files/customer.php`, `Magento/Customer/_files/customer_address.php` and `Magento/Checkout/_files/quote.php` are respectively aliased as _virtualProduct_, _customer_, _customerAddress_ and _quote_.
`Magento/Customer/_files/customer_address.php` requires _customer_ (`Magento/Customer/_files/customer.php`) and `Magento/Checkout/_files/quote.php` requires _virtualProduct_ (`Magento/Catalog/_files/product_virtual.php`) and _customer_ (`Magento/Customer/_files/customer.php`) fixtures.
The data fixture provider overwrites the fixtures _virtualProduct_, _customerAddress_ and _quote_ data as well as resolves fixtures dependencies using the `@` keyword.

```php
#Magento/Catalog/_files/product_virtual.php
use Magento\Catalog\Fixtures\ProductFixture;
use Magento\TestFramework\Helper\Bootstrap;

$objectManager = Bootstrap::getObjectManager();
$productFixture = $objectManager->get(ProductFixture::class);
$product = $productFixture->create($data ?? []);
return $product;
```

```php
#Magento/Customer/_files/customer.php
use Magento\Customer\Fixtures\CustomerFixture;
use Magento\TestFramework\Helper\Bootstrap;

$objectManager = Bootstrap::getObjectManager();
$customerFixture = $objectManager->get(CustomerFixture::class);
$customer = $customerFixture->create($data ?? []);
return $customer;
```

```php
#Magento/Customer/_files/customer_address.php
use Magento\Customer\Fixtures\CustomerAddressFixture;
use Magento\TestFramework\Helper\Bootstrap;

$objectManager = Bootstrap::getObjectManager();
$customerAddressFixture = $objectManager->get(CustomerAddressFixture::class);
$customerAddress = $customerAddressFixture->create(['customer' => $customer, 'data' => $data ?? []]);
return $customerAddress;
```

```php
#Magento/Checkout/_files/quote.php
use Magento\Checkout\Fixtures\QuoteFixture;
use Magento\TestFramework\Helper\Bootstrap;

$objectManager = Bootstrap::getObjectManager();
$quoteFixture = $objectManager->get(QuoteFixture::class);
$quote = $quoteFixture->create(['customer' => $customer, 'products' => $products, 'data' => $data ?? []]);
return $quote;
```

```php
namespace Magento\Catalog\Fixtures;

use Magento\Catalog\Api\Data\ProductInterface;
use Magento\Catalog\Api\Data\ProductInterfaceFactory;
use Magento\Catalog\Api\ProductRepositoryInterface;
use Magento\Catalog\Model\Product\Attribute\Source\Status;
use Magento\Catalog\Model\Product\Type;
use Magento\Catalog\Model\Product\Visibility;
use Magento\TestFramework\DataObjectHydrator;
use Magento\TestFramework\FixtureInterface;

class ProductFixture implements FixtureInterface
{
    /**
     * @var ProductInterfaceFactory
     */
    private $factory;
    /**
     * @var ProductRepositoryInterface
     */
    private $repository;
    /**
     * @var DataObjectHydrator
     */
    private $dataObjectHydrator;

    /**
     * @param ProductInterfaceFactory $factory
     * @param ProductRepositoryInterface $repository
     * @param DataObjectHydrator $dataObjectHydrator
     */
    public function __construct(
        ProductInterfaceFactory $factory,
        ProductRepositoryInterface $repository,
        DataObjectHydrator $dataObjectHydrator
    ) {
        $this->factory = $factory;
        $this->repository = $repository;
        $this->dataObjectHydrator = $dataObjectHydrator;
    }

    /**
     * @param array $data
     * @return ProductInterface
     */
    public function create(array $data)
    {
        $product = $this->factory->create();
        $product = $this->dataObjectHydrator->hydrate($product, array_merge($this->defaultData(), $data));
        $this->repository->save($product);
        return $product;
    }

    /**
     * @return array
     */
    private function defaultData(): array
    {
        return [
            'id' => 21,
            'type_id' => Type::TYPE_VIRTUAL,
            'sku' => 'virtual-product',
            'name' => 'Virtual Product',
            'attribute_set_id' => 4,
            'tax_class_id' => 0,
            'website_ids' => [1],
            'price' => 10,
            'visibility' => Visibility::VISIBILITY_BOTH,
            'status' => Status::STATUS_ENABLED,
            'stock_data' => [
                'qty' => 100,
                'is_in_stock' => 1,
                'manage_stock' => 1,
            ],
        ];
    }
}
```
With the examples above, one can notice that there is absolutely no need for fixtures file anymore. Instead, we can simply use the fixture class in the data fixture annotation as following:

```php
class QuoteTest extends \PHPUnit\Framework\TestCase
{
    /**
     * @magentoDataFixture \Magento\Catalog\Fixtures\ProductFixture as virtualProduct
     * @magentoDataFixture \Magento\Customer\Fixtures\CustomerFixture as customer
     * @magentoDataFixture \Magento\Customer\Fixtures\CustomerAddressFixture as customerAddress require customer
     * @magentoDataFixture \Magento\Checkout\Fixtures\QuoteFixture as quote require customer, virtualProduct
     */
    public function testCollectTotals(): void
    {
    }

    public function collectTotalsDataFixture(): array
    {
        return [];
    }
}
```
You can create as many instances of a fixture as you wish for your test case:

```php
class ProductsList extends \PHPUnit\Framework\TestCase
{
    /**
     * @magentoDataFixture \Magento\Catalog\Fixtures\ProductFixture as product1
     * @magentoDataFixture \Magento\Catalog\Fixtures\ProductFixture as product2
     * @magentoDataFixture \Magento\Catalog\Fixtures\ProductFixture as product3
     */
    public function testGetProductsCount(): void
    {
    }

    public function getProductsCountDataFixture(): array
    {
        return [
            'product1' => [
                'sku' => 'simple1'
            ],
            'product2' => [
                'sku' => 'simple2'
            ],
            'product3' => [
                'sku' => 'simple3',
                'status' => Status::STATUS_DISABLED,
            ],
        ];
    }
}
```

**Integration test extensibility**

```xml
<test class="Magento\Quote\Model\Quote">
    <method name="testCollectTotals">
        <!-- Add new fixture -->
        <magentoDataFixture path="Magento/Catalog/_files/product_simple.php" before="Magento/Checkout/_files/quote.php" as="simpleProduct"/>
        <fixtureDataProvider>
            <fixtures>
                <fixture name="virtualProduct">
                    <attributes>
                        <!-- Overwrite attribute -->
                        <attribute name="special_price" xsi:type="number">15.0</attribute>
                        <!-- Add new attribute -->
                        <attribute name="stock_data" xsi:type="array">
                            <attribute name="qty">1</attribute>
                        </attribute>
                    </attributes>
                </fixture>
                <fixture name="quote">
                    <attributes>
                        <attribute name="@products[]" xsi:type="array">
                            <!-- Overwrite dependency -->
                            <attribute name="virtualProduct" xsi:type="array">
                                <attribute name="qty">1</attribute>
                            </attribute>
                            <!-- Add new dependency -->
                            <attribute name="simpleProduct" xsi:type="array">
                                <attribute name="qty">2</attribute>
                            </attribute>
                        </attribute>
                    </attributes>
                </fixture>
            </fixtures>
        </fixtureDataProvider>
    </method>
</test>
```


**Pros**
- Reduces duplicate codes in data fixtures files
- Reduces the number of data fixtures files

**Cons**
- Increases dock block size
