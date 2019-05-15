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
1. Use current Search API for simultaneously make search and filter by attributes (actually this is a mix of **Advansed Search + Quick Search functionality**) 
1. New implementation will not be functionally compatible with previous 

Here are the list of changes that will be made for new GraphQl Resolver

#### Product Filtering
Current Product Filter will be eliminated and replaced with new one, which will follow the following rules:
(actually this is behaviour of Advanced Search)

1. Product Attribute will be available for filtering if:
1.1. Attribute be "searchable" and have option "Visible in Advanced Search" is set to "Yes"
1.1. Product Attribute of type "Select" must have options
1. Only the following filter conditions are available
1.1. **From..To** filter: Price and all attributes with type "Date" 
1.1. **Like** filter: All text attributes like name, sku, desciption ...
1.1. **Equal/In** filter: All attributes with type "Select" like category, color, size ...
1. "Or" filter is not supported

#### Schema changes
On global level schemal will be the same, but some changes will be introduced:
1. **input ProductFilterInput** will be deprecated and replaced with new FilterInput
1. Some "inner" resolvers, that not needed anymore will be removed, e.g.\Magento\CatalogGraphQl\Model\Resolver\Product\EntityIdToId for ProductInterface

#### Functional incompatible
Due to we stop using existing "APIs" and start use new Data Providers any extensions, that used events fired with those APIs will stop working 


## Design
Use plain arrays for transfer data between resolvers. Inner resolvers can depends only in fields that present in schema. 
Use indexers for retrieve data (prices, search, ...)


## Backward Compatibility Policy

New resolvers will be delivered as a separate set of modules for do not introduce incompatible changes in the patch release. 
In a further minor (major?) release, those modules will replace the existing ones.


## New behaviour

Here is a table of visibility mapping between different search areas. 


Area/Visibility  | not visible | catalog | search| catalog+search 
---------------- |-------------|---------|-------|---------------
 category                |           |    1    |       |    1           
 search                  |           |         |    1  |    1            
 advanced search         |           |    1    |    1  |    1            
 grahpql search          |           |         |    1  |    1            
 grahpql filter          |           |    1    |       |    1            
 grahpql search + filter |           |    ?    |    ?  |    ?            


## POC

https://github.com/magento/graphql-ce/tree/567-category-query
