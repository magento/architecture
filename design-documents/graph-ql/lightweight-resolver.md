# Lightweight GrahpQL resolvers

## Why?

For provide significantly better performance.
 
## How?
**Aggregate** request data during iterating over resolvers (solving "N+1 select" problem) and execute **direct queries**(SQL/Elasticsearch) instead of using existing API.


Lets see example

```graphql
query products_with_categories {
  productDetail: products(search: "iphone" ) {
    items {
      id
      sku
      name
      categories {
        id
        name
      }
    }
  }
}
```

The old approach will do the following for executing query:
1. Load product collection
2. Iterate over each product and load categories

As a result it will perform 1 + N actions, where N amount of products

In the new approach product ids will be aggregated and later used for retrieve categories.
As a result it will perform 1 + 1 actions

## Known limitations
Current implementation of Product Resolver for GraphQL support "search" and "filter" operations. "filter" support all possible operations (eq, like, finset, ...) for hard-coded list of product attributes. Some of the attributes are not "filterable", that not allow using index for filter by those attributes.

This leads us to necessarily to execute "search" without filters with huge size limit by (by default 10000) and then additionally execute "filter" operation in MySql.

We must execute "search" and "filter" as a single operation from performance reasons. Some numbers: 
* Response from elasticsearch for 10k on 120Pro instance: **600-800ms**.
* Additional filtration in MySql took: 120-2000 ms

For achieving the better performance we **must accept limitations**:
1. Use current Search API for simultaneously make search and filter by attributes
1.1. Support only "and" condition
1.2. Support limited conditions per each attribute type (e.g. only "eq"/"in" for drop-down attribute, "match" for "text" attribute)
2. New implementation will not be functionally compatible with previous


## Design
Use plain arrays for transfer data between resolvers. Inner resolvers can depends only in fields that present in schema. 
Use indexers for retrieve data (prices, search, ...)


## Drawbacks

1. Maintaining: separate set of modules, that must replace current GraphQL* modules
2. Potentially issue with sql query complexity caused by aggregation


## POC

https://github.com/magento/graphql-ce/tree/567-category-query

## Additional changes
1. Visibility must be set explicitly. Currently product visibility depends on request: if we use "search" or "filter" + "filter", then "visible in search" condition is applied. If we use "filter" only then "visible in catalog" condition is applied.


