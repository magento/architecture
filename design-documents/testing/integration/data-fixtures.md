## Problem
When it comes to write a data fixture for your test case, the first thing you would do is to search for existing data fixture you can reuse in your test case. Most of the time you will find such data fixture that almost meets the requirements of your test case except that it's missing something very important to your test case.
Therefore you end up writing a new data fixture for your test case that is 99% a copy of existing one.
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

We could certainly redesign data fixtures to be object oriented. But if we just want to tackle this issue as simpler as we can, we could consist we could extend the format of data fixture annotation
to support a second parameter which will be injected to the data fixture file as following.
```php
/**
 * @magentoDataFixture Magento/Catalog/_files/product_virtual.php, {"productData":{"stock_data": {"qty":1}}}
 */

```
The string after comma following the fixture file name is indeed the data that needs to be injected into the fixture file for customization. The format is well known JSON format that gives a flexible way to pass any type of data to the fixture (string, int, float and array).

```php
use Magento\Catalog\Api\Data\ProductInterface;
use Magento\Catalog\Api\Data\ProductInterfaceFactory;
use Magento\Catalog\Api\ProductRepositoryInterface;
use Magento\Catalog\Model\Product\Attribute\Source\Status;
use Magento\Catalog\Model\Product\Visibility;
use Magento\Framework\DataObject;
use Magento\TestFramework\Helper\Bootstrap;

$productData = $productData ?? [];
$defaultProductData = [
    'id' => 21,
    'sku' => 'virtual-product',
    'name' => 'Virtual Product',
    'attribute_set_id' => 'sku',
    'website_ids' => 'sku',
    'price' => 'sku',
    'visibility' => Visibility::VISIBILITY_BOTH,
    'status' => Status::STATUS_ENABLED,
    'stock_data' => Status::STATUS_ENABLED,
];

$productData = array_merge($productData, $defaultProductData);
$objectManager = Bootstrap::getObjectManager();
$productFactory = $objectManager->get(ProductInterfaceFactory::class);
/** @var ProductRepositoryInterface $productResource */
$productRepository = $objectManager->get(ProductRepositoryInterface::class);
/** @var ProductInterface $product */
$product = $productFactory->create();
$product = data_object_hydrate_from_array($product, $productData);
$productResource->save($product);

// this part of the code should be moved to a service.
// something similar to \Magento\Framework\Webapi\ServiceInputProcessor::convertValue but less strict.

if (!function_exists('upper_came_case')) {
    function upper_came_case(string $value): string
    {
        return str_replace('_', '', ucwords($value, '_'));
    }
}

if (!function_exists('data_object_hydrate_from_array')) {
    function data_object_hydrate_from_array(DataObject $object, array $data): DataObject
    {
        foreach ($data as $key => $value) {
            $camelCaseProperty = upper_came_case($key);
            $setterName = 'set' . $camelCaseProperty;
            $boolSetterName = 'setIs' . $camelCaseProperty;
            if (method_exists($object, $setterName)) {
                $object->$setterName($value);
                unset($data[$key]);
            } elseif (method_exists($object, $boolSetterName)) {
                $object->$boolSetterName($value);
                unset($data[$key]);
            }
        }
        $object->addData($data);
        return $object;
    }
}
```
**Pros**
- Reduces duplicate codes in data fixtures files
- Reduces the number of data fixtures files

**Cons**
- Increases dock block size
