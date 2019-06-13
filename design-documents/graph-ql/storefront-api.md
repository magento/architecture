
# Storefront API (implementation details)

## Purpose
Working on [lightweight GraphQL resolvers](https://github.com/magento-performance/architecture/blob/graphql/design-documents/graph-ql/lightweight-resolver.md) we want to introduce new API
that is responsible for retrieving Storefront specific data

## Design 
New API should satisfy the following criteria:
1. Support batch requests
1. All services must be stateless
1. Be performance efficient
1. Keep service isolation in mind 

Here is a detailed design description which satisfies above criteria: [Batch query services](https://github.com/magento/architecture/pull/163/files?short_path=6bf9437#diff-6bf9437e365a3d978a3743fe86d815f5)

Technical vision of [StoreFront API](https://github.com/magento/architecture/blob/1c3bad3908bb90f45d020fd182881520057678a1/design-documents/storefront/storefront-api.md)

## Structure

* API module (**Magento\CatalogProductAPI**)
  * Contains API's for specific domain layer
* Implementations (**Magento\CatalogProduct**)
  * Basic implementation for specific API

### Technical notes

The main goal is performance. Here are some requirements that each new service should follow:
1. Be lazy: to return only requested data
1. Be greedy: to aggregate requests and execute query only once
1. Be lightweight: to return simple structures

## Product Search API

### Filtration

1. Only AND filters supported
1. OR filters are not supported

If there is a necessity to execute a request with OR filter, it should be re-writen as 2 requests instead.  

### Sorting

APIs support only **one** field for sorting:
1. Sort by relevance in case of full text search
1. Sort by specified field (e.g. price, name...) in other case

* Currently there are no valid scenarios on Magento Storefront where we need to apply sorting by more than one attribute
* The scenario of Quick Search (sorting by relevance) and then applying additional ordering by price not going to be supported 

### API Segregation

Let's consider the necessity of introducing 2 dedicated APIs for Search (full text search) and Filtration of entities data.
Taking into account that Full Text Search is both filtration (non zero relevance) and sorting (by relevance desc) operation, and the limitation to have only one field we sort by, there is no sense to provide a method which set sorting for Search API as ordering by relevance is always pre-defined.


| ProductSearchRequestCriteria         | ProductFilterRequestCriteria |
| ------------- | ----------------------- |
| getFilters(): array; | getFilters(): array; |
| getPage(): array; | getPage(): array; |
| getScopes(): array; | getScopes(): array; |
| getFields(): array; | getFields(): array; |
| getAggregations(): ?array; | getAggregations(): ?array; |
| *getSearchTerm(): string;* | *getSort(): string;* |

The difference between Search and Filter in "search term" and "sort" methods:
1. Search API provides "search term" method, but has a lack of "sort"
1. Filter API has "sort" but no need to have "search term"

Pros:
1. More semantically correct interfaces (OOP friendly)
1. Both interfaces could evolve independently

Cons:
1. Necessity to extend and customize both interfaces

As an alternative option we can combine both APIs which allow search, filtering or both for entities (i.e. product).

Option 1. Add "search term" as an optional field

```php
interface ProductSearchInterface
{
    public function getSearchTerm(): ?string;
    public function getFilters(): array;
    public function getPage(): array;
    public function getScopes(): array;
    public function getFields(): array;
    public function getSort(): array;
    public function getAggregations(): array;
}
```

Pros:
1. single extension point

Cons:
1. we ignore field "sort" in case of fulltext search
1. "search term" is optional, because it's no needed for filtering requests

Option 2. Set "search term" via filters OR sort field.

```php
interface ProductSearchInterface
{
    public function getFilters(): array;
    public function getPage(): array;
    public function getScopes(): array;
    public function getFields(): array;
    public function getSort(): array;
    public function getAggregations(): array;
}
```

There are different options for set search term:

1. Option 2.1 Set search term via sort field 
```php
[
    'sort' => [
        'field' => "super car", 'type' => 'relevance',  // always sort by relevance, DESC
    ,
    ]
]
```

As a drawback we provide search term in not intuitive way and sorting field is responsible for filtering.

2. Option 2.2 Set search term via filters field.
```php
[
    'filters' => [
        ['field' => '*', 'value' => 'super car', 'condition_type' => 'match'] // Full text search by all fields. Sort by relevance added automatically
    ],
]

```
As a drawback we will ignore field "sort" in case of fulltext search.

3. Option 2.3 Set search term via filter field and provide *additional* sort direction by relevance for another term
```php
[
    'filters' => [
        ['field' => '*', 'value' => 'super car', 'condition_type' => 'match'],  // Sort by relevance added automatically
    ],
    'sort' => [
        'field' => "relevance('modern vehicle')", 'type' => 'ASC',
     ]
]

```
As a drawback we can receive not expected data. Currently this feature is not supported in Magento.


#### Aggregation

We agreed with the following:
1. Aggregations should be passed from outside in order to be retrieved
1. In case of empty list is provided no aggregations is returned
1. To simplify interface we are going to return all possible aggragation if **null** is provided


#### API details

```php

/**
 * @api
 * Sort by relevance is automatically added
 */ 
interface ProductSearchInterface {
    /**
     * @param ProductSearchRequestCriteria[] $request
     * @return ProductResponse
     */
    public function search(array $request): ProductResponse;
}

interface ProductSearchRequestCriteria
{
    /**
     * @return string
     */
    public function getSearchTerm(): string;

    /**
     * @return \Magento\Framework\Api\Filter[] ["field", "value", "condition_type"]
     */
    public function getFilters(): array;

    /**
     * @return string[] ["pageSize", "currentPage"] || ["endCursor", "hasNextPage"]
     */
    public function getPage(): array;

    /**
     * @return string[] List of scopes in format: ["name" => "value"]
     */
    public function getScopes(): array;

    /**
     * @return string[] List of requested fields. See format below
     */
    public function getFields(): array;

    /**
     * @return string[] ["attribute_name_1", "attribute_name_2", ...]
     * OR add more options? ["attribute_name_1" => [...options...], ...] @see: https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-terms-aggregation.html
     */
    public function getAggregations(): array;
}


interface ProductResponse extends \Magento\Framework\Api\ExtensionAttributesInterface
{
    /**
     * @return string[] Product item data
     */
    public function getItems(): array;
    
    /**
     * @return string[][] see example below
     */
     public function getAggregations(): array;

    /**
     * @return array
     */
    public function getPageInfo(): array;
}
```

#### Request with response example
```php
// Request
[
  'searchTerm' => 'modern car',
  'filters' => [
     ['field' => 'price', 'value' => '12', 'condition_type' => 'from'],
     ['field' => 'color', 'value' => ['10', '11'], 'condition_type' => 'in'],
  ],
  'page' => [
     'pageSize' => 10,
     'currentPage' => 3
  ],
  'scopes' => [
     'store' => 'US',
     'customerGroupId' => 1
  ],
  'fields' => [
     'name',
     'price',
  ],
  'aggregations' => [
     'category',
     'price',
  ],
]

// Response
[
  [
     'items' => [
         [
           'name' => 'Car 1',
           'price' => '22',
         ],
         ...
     ],
     'aggregations' => [
         'category' => [
             [
                 'name' => 'Tesla',
                 'count' => 21,
             ],
             ...
         ],
         'price' => [
             [
                 'name' => '12..24',
                 'count' => 44,
             ],
             ...
         ]
     ],
     'pageInfo' => [
         'totalCount' => 100,
         'pages' => 10,
         'pageSize' => 10,
         'currentPage' => 3
     ]
     
  ]
]
```


## Product Price API

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
    public function getFilters() : ?string[][]; // list of filters in format: ["field", "value", "condition_type"]. Reffer to \Magento\Framework\Api\Filter. Can be empty
    public function getScopes() : string[]; // list of scopes in format: ["name" => "value"]
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


## Scopes
1. Each scope consist of name and value, e.g. name: "store", value: "5".
1. API consumer pass all known scopes to API (store, customer_group)
1. API implementation utilize needed scpoes.
1. Exception is thrown in case of scope is missed

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




