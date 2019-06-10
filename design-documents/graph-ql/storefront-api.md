
# Storefront API (implementation details)

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

## Product Search API

```php
 /**
 * @api
 * @fields: [
 *  "sku": string,
 *  "productId": int,
 *  "facet.name": string,
 *  "facet.items.label": string,
 * ]
 */
interface Products
{
  /**
   * @param ProductRequestCriteria[] $requests List of requests
   * @return array
   */
  public function search(array $requests) : array
}

/**
* Request DTO
*/
interface ProductRequestCriteria
{
    public function getQuery() : ?string; // query string used for full text search within products. Can be empty
    public function getFilters() : ?string[][]; // list of filters if format: ["field", "value", "condition_type"]. Reffer to \Magento\Framework\Api\Filter. Can be empty
    public function getDimensions() : string[]; // list of dimensions if format: ["name" => "value"]
    public function getFields() : string[]; // list of requested fields.
}

$prices = Products::search([
    new ProductPriceSearchCriteria(
       null, // no full text search
       [
           ["field" => "name", "value" => "Tesla X*", "condition_type" => "like"],
           ["field" => "price", "value" => "55000", "condition_type" => "from"],
       ]
       ['store' => 1, 'customer_group_id' => 2],
       ['sku', 'name', 'productId']
    ),
    new ProductPriceSearchCriteria(
       "tesla car",
       null, // no filtering
       ['store' => 1, 'customer_group_id' => 2],
       ['sku', 'name', 'productId']
    ),
   ]);

```


## Product Price API

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
    public function getFilters() : ?string[][]; // list of filters if format: ["field", "value", "condition_type"]. Reffer to \Magento\Framework\Api\Filter. Can be empty
    public function getDimensions() : string[]; // list of dimenstions if format: ["name" => "value"]
    public function getFields() : string[]; // list of requested fields. Must be declared with API
}

$prices = ProductPrice::getPrices([
    new ProductPriceSearchCriteria(
       [
           ["field" => "productId", "value" => [1,2,4], "condition_type" => "in"],
       ],
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


// return prices in the same order as requested.
 [
     [
         [
             'minimalPrice' => '10.22',
             'maximalPrice' => '15',
         ],
         
         // 42 product  is missed and not returned. Client must handle this if needed (e.g. request productId field)
         [
             'minimalPrice' => '24',
             'maximalPrice' => '44',
         ]
     ],
     [
         [
             'productId' => 4,
             'maximalPrice' => '12'
         ]
     ]
 ];
  
```


## Dimensions
1. Each dimension consist of name and value, e.g. name: "store", value: "5".
1. API consumer pass all known dimensions to API (store, customer_group)
1. API implementation utilize needed dimensions.
1. *Exception is thrown in case of dimension is missed* (need to be discussed)

## Requested fields

1. Each API should expose the list of fields that could be requested.
1. List of fields should be declarative and extendable.
1. Nested fields declared with dot notation (facet.items.label)

For the first iteration it can be hard-coded as a part of API documentation, e.g. (mixed example):

```php
/** 
 * @api
 * @fields: [
 *  "sku": string,
 *  "productId": int,
 *  "minimalPrice": float,
 *  "facet.name": string,
 *  "facet.items.label": string,
 *  "facet.items.count": int,
 * ]
**/ 

// Response
[
  'sku' => 'sku-777',
  'productId' => 3,
  'minimalPrice' => 4.5,
  'facet' => [
   [
     'name' => 'category', 
     'items' => [
        [
           'label' => 'Cat 1',
           'count' => 5,
         ],
        ...
     ],
   ...
   ],
]  

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




