
# Storefront API

## Purpose
Working on [lightweight GraphQL resolvers](https://github.com/magento-performance/architecture/blob/graphql/design-documents/graph-ql/lightweight-resolver.md) we want to introduce new API
that is responsible for retrieving Storefront specific data

## Design 
New API should satisfy the following criteria:
1. Support batch requests
1. All services must be stateless
1. Be performance friendly
1. Keep in mind about service isolation

Here is a detailed design description that satisfies these criteria: [Batch query services](https://github.com/magento/architecture/pull/163/files?short_path=6bf9437#diff-6bf9437e365a3d978a3743fe86d815f5)

Technical vision of [StoreFront API](https://github.com/magento/architecture/blob/1c3bad3908bb90f45d020fd182881520057678a1/design-documents/storefront/storefront-api.md)

## Structure

* API module (**Magento\CatalogProductAPI**)
  * Contains API's for specific domain layer
* Implementations (**Magento\CatalogProduct**)
  * Basic implementation for specific API

### Technical notes

The main goal is performance. Here are some requirements that each new service should follow:
1. Be lazy: return only requested data
1. Be greedy: aggregate requests and execute query only once
1. Be lightweight:  return simple structures

## Example of API

Lets builds Store front API for Product Price, that will satisfyour criteria

```php
// @api
// List of fields that could be requested: [productId, minimalPrice, maximalPrice, price]
interface ProductPrice
{
  /**
   * @param ProductPriceRequest[] $requests List of requests
   * @return array
   */
  public function getPrices(array $requests) : array
}

/**
* Request DTO
*/
class ProductPriceRequest
{
    public function getProductIds() : int[]; // product ids
    public function getDimensions() : string[]; // list of dimenstions if format: ["name" => "value"]
    public function getFields() : string[]; // list of requested fields. Must be declared with API
}

$prices = ProductPrice::getPrices([
    new ProductPriceSearchCriteria(
       [1, 42, 2],
       ['store' => 1, 'customer_group_id' => 2],
       ['minimalPrice', 'maximalPrice']
    ),
    // additionally return "productId"
    new ProductPriceSearchCriteria(
       [4],
       ['store' => 1, 'customer_group_id' => 2],
       ['productId', 'maximalPrice']
    ),
   ]);


// return prices in the same order as requested. Returning data in associative array
 [
     [
         1 => [
             'minimalPrice' => '10.22',
             'maximalPrice' => '15',
         ],
         
         // 42 product  is missed and not returned
         2 => [
             'minimalPrice' => '24',
             'maximalPrice' => '44',
         ]
     ],
     [
         4 => [
             'productId' => 4,
             'maximalPrice' => '12'
         ]
     ]
 ];
 
```

**Return data as associative array** will solve problem with missed data for requested product (42 in example)

## Dimensions
Each dimension consist of name and value, e.g. name: "store", value: "5"
API implementation utilize needed dimensions. 
Open questions:
1. How to pass dimension to API in consumer? Pass all possible scopes (store, website, customer group)?
1. Behaviour if part of required for implementation dimensions is not passed

## Requested fields

Each API should expose the list of fields that could be requested. List of fields should be declarative and extendable. 
For the first iteration it can be hard-coded as a part of API documentation, e.g.:

```php
/** 
 * @api
 * @fields: [
 *  "productId": int,
 *  "minimalPrice": float,
 *  "maximalPrice": float,
 *  "price": float, 
 * ]
**/ 
interface ProductPrice
{
  /**
   * @param ProductPriceRequest[] $requests List of requests
   * @return array
   */
  public function getPrices(array $requests) : array
}
```

List of requested fields can be dynamically generated (e.g. for retrieve list of product attributes)




