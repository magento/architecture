
# Storefront API

## Purpose
Working on [lightweight GrpaphQL resolvers](https://github.com/magento-performance/architecture/blob/graphql/design-documents/graph-ql/lightweight-resolver.md) we want to introduce new API
that will be responsible for retrieving Storefront specific data

## Design 
New API should satisfy the following criteria:
1. Support batch requests
1. All services must be stateless
1. Be performance friendly
1. Keep in mind about service isolation

Here is a detailed design description that satisfies these criteria: [Batch query services](https://github.com/magento/architecture/pull/163/files?short_path=6bf9437#diff-6bf9437e365a3d978a3743fe86d815f5)

## Structure

* API module (**Magento\CatalogProductAPI**)
  * Contains API's for specific domain layer
* Implementations (**Magento\CatalogProduct**)
  * Basic implementation for specific API

### Technical notes

The main keystone is performance. Here are some requirements that each new service should follow:
1. Be lazy: return only requested data
1. Be greedy: aggregate requests and execute query only once
1. Be lightweight:  return simple structures


## Example
```php
interface ProductPrice
{
  /**
   * @param ProductPriceSearchCriteria[] $requests
   * @return array
   */
  public function getPrices(array $requests) : array
}

/**
* Request DTO
*/
class ProductPriceSearchCriteria
{
    public function getProductIds() : int[];
    public function getCustomerGroupId() : int;
    public function getWebsiteId() : int;
    public function getFields() : string[];
}

class ResultContainer implements ContainerInterface
{
    public function getResult() : array
}

$prices = ProductPrice::getPrices([
    new ProductPriceSearchCriteria([1,2], 4, 1, ['minimalPrice', 'maximalPrice']),
    new ProductPriceSearchCriteria([3,4], 4, 1, ['maximalPrice']),
]);

/**
* return prices in the same order as requested
* [
*     [
*         1 => [
*             'productId' => 1,
*             'minimalPrice' => '10.22',
*             'maximalPrice' => '15',
*         ],
*         2 => [
*             'productId' => 2,
*             'minimalPrice' => '24',
*             'maximalPrice' => '42',
*         ]
*     ],
*     [
*         3 => [
*             'productId' => 1,
*             'minimalPrice' => '8.56'
*         ]
*     ]
* ];
*/
$prices->getResult();

 
```

