
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

If there is a necessity to execute a request with OR filter, it should be re-written as 2 requests instead.
For layered navigation (faceted search) where multi-select checkbox filtering supported, the request will look like 
Size IN (10, 10.5) instead of Size 10 OR 10.5 
Because in this case attribute values are always finite set, and their cardinality (number of unique values) is quite limited

### Sorting

Despite our API supports several fields for sorting here are some agreements that we will follow:
1. Sort by relevance is applied by default in case of full text search, if no other sort is provided
1. For mix sort by specified field and relevance you should explicitly set sort by relevance


### Aggregation

We agreed with the following:
1. Aggregations should be passed from outside in order to be retrieved
1. In case of empty list is provided no aggregations is returned
1. To simplify interface we are going to return all possible aggragation if **"\*"** is provided as aggregation name

Example:
```php
 ["category", "price"] // provide aggregations by category and price
 [] // do not provide aggregations
 ["*"] // provide all possible aggregations
 
```


### Scopes
1. Each scope consist of name and value, e.g. name: "store", value: "5".
1. API consumer pass all known scopes to API (store, customer_group)
1. API implementation utilize needed scpoes.
1. Exception is thrown in case of scope is missed


### API Details

Search (full text search) and Filtration of entities data is combined into one API.
1. ProductSearchInterface - Store Front API for product search by batch of search requests
1. ProductSearchRequestCriteria - Request criteria for product search
1. ProductResponseContainer - Response container for specified ProductSearchRequestCriteria

**Execution result is returned in the order of received requests**

We agreed, that the order of received result containers is equal to the order of sent Requests. Service implementation has to guarantee this sort order.

```php

interface ProductSearchInterface {
    /**
     * @param ProductSearchRequestCriteria[] $request
     * @return ProductResponseContainer[]
     */
    public function search(array $request): array;
}

interface ProductSearchRequestCriteria
{
    /**
     * @return string
     */
    public function getSearchTerm(): string;

    /**
     * @return \Magento\Framework\Api\Filter[] e.g. [["field", "value", "condition_type"], ...]
     */
    public function getFilters(): array;

    /**
     * @return \Magento\Framework\Api\SortOrder[] e.g. [["field", "direction"], ...]
     */
    public function getSort(): array;

    /**
     * @return string[] e.g. ["pageSize", "currentPage"]
     */
    public function getPage(): array;

    /**
     * @return string[] List of scopes in format: ["name" => "value"], e.g ['storeId' => 3]
     */
    public function getScopes(): array;

    /**
     * @return string[] List of requested fields. e.g. ['id', 'sku', 'name', 'url_key']
     */
    public function getFields(): array;

    /**
     * @return string[] Requested aggregations, e.g. ["attribute_name_1", "attribute_name_2", ...]
     */
    public function getAggregations(): array;
}


interface ProductResponseContainer extends \Magento\Framework\Api\ExtensionAttributesInterface
{
    /**
     * @return string[] Product item data
     */
    public function getItems(): array;
    
    /**
     * @return string[][] List of requested aggregations
     */
     public function getAggregations(): array;

    /**
     * @return array
     */
    public function getPageInfo(): array;
    
    /**
     * @return boolean Response status
     */
    public function getStatus(): bool;

    /**
     * @return string[]|null Error messages in case of failure
     */
    public function getError(): ?array;
}
```

#### Request with response example
```php
// Request
{
    "searchTerm": "modern car",
    "filters": [
        {
            "field": "price",
            "value": "12",
            "condition_type": "from"
        },
        {
            "field": "color",
            "value": [
                "10",
                "11"
            ],
            "condition_type": "in"
        }
    ],
    "sort": [
        {
            "field": "price",
            "direction": "ASC"
        },
        {
            "field": "_relevance",
            "direction": "DESC"
        }
    ],
    "page": {
        "pageSize": 10,
        "currentPage": 3
    },
    "scopes": {
        "storeId": "US",
        "customerGroupId": 1
    },
    "fields": [
        "name",
        "price"
    ],
    "aggregations": [
        "category",
        "price"
    ]
}

// Response for specific request

{
    "items": [
        {
            "name": "Car 1",
            "price": "22"
        }
    ],
    "aggregations": {
        "category": [
            {
                "name": "Tesla",
                "count": 21
            }
        ],
        "price": [
            {
                "name": "12..24",
                "count": 44
            }
        ]
    },
    "pageInfo": {
        "totalCount": 100,
        "pages": 10,
        "pageSize": 10,
        "currentPage": 3
    },
    "status": true,
    "error": null
}

```


### Open questions

#### Requested fields

From the performance perspective, we must return only requested data.
There are two options how it can be achieved:


#### Option 1

We consider everything that API can request as "data". 
Use dot.annotation for describing requested fields without segregation between "entity data" and "response aggregation", "paging"...

We just need to follow a simple rule: 

**All requested fields are returned in corresponding containers**, e.g. "items.name" returned in "items" container

API expose XML (that should be autogenerated) with all possible fields and field types that can be requested

```php
/** 
 * @api
 * @fields: [
 *  "items.sku": string,
 *  "items.productId": int,
 *  "items.minimalPrice": float,
 *  "pageInfo.totalCount",
 *  "pageInfo.totalPages",
 *  "aggregations.category": string,
 *  "aggregations.price": string,
 * ]
**/ 

// Request

  "fields": [
      "items.name",
      "items.price",
      "aggregations.catagory",
      "pageInfo.totalCount"
  ],


// Response
{
    "items": [
     {
        "sku": "sku-777",
        "price": 45
     },
     {
        "sku": "sku-42",
        "price": 5
     }
    ],
    "pageInfo": {
        "totalCount": 42
    },
    "aggregation": [
        {
            "name": "category",
            "items": [
                {
                    "label": "Cat 1",
                    "count": 5
                }
            ]
        }
    ]
}
```

List of requested fields can be dynamically generated (e.g. for retrieve list of product attributes, aggregations,...)

**Proc**

1. Help to solve the "greedy" requirement
1. Delcarative, intutive interface, follow WYSIWYG approach
1. No need to itnroduce "hack" like \* (see Aggregations)
1. We do not need "aggregation" field in API Request - by the fact this field just request additional data

**Cons**

1. Need to follow agreements

After accepting this field **getAggregations** will be removed from ProductSearchRequestCriteria


#### Option2

We consider everything that API can request as different type of "data". 

Requested fields are splitted into 3 groups:
- entity data
- aggregations
- meta info

Due to we do not want to return data, that was not requested we add the following rules
1. All "data specific" arguments are optional (NULL)
1. If argument was not passed NULL is returned in Response object for this argument


```php
interface ProductSearchInterface
{
// set filtrations 
    public function getSearchTerm(): ?string;
    public function getFilters(): array;
    public function getPage(): array;
    public function getScopes(): array;
    public function getSort(): ?array;

// request data...
    public function getFields(): ?array;
    public function getAggregations(): ?array;
    public function getMetaInfo(): ?array;
}
```


```php
// Request

  "fields": [
      "sku",
      "price",
  "aggregations": [
      "catagory"
  ],
  "metaInfo": [
     "totalCount",
     "currentPage"
  ]


// Response
{
    "items": [
     {
        "sku": "sku-777",
        "price": 45
     },
     {
        "sku": "sku-42",
        "price": 5
     }
    ],
    "aggregation": [
        {
            "name": "category",
            "items": [
                {
                    "label": "Cat 1",
                    "count": 5
                }
            ]
        }
    ],
    "metaInfo": {
        "totalCount": 42,
        "currentPage": 1
    }
}
```


<details>
<summary>

## Product Price API (TBD)
</summary>


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
</details>
 
 
<details>
<summary>

### API Segregation
</summary>
Here you can find some thoughts that led us to the accepted solution 
  

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

</details>


