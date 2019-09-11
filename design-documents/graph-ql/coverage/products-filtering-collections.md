**Problem description**

The attributes that can be filtered on are being generated/added dynamically into the schema. The schemas in GraphQL are cached and the cache needs to be cleaned every-time when a new filterable attribute is added. This is an expensive operation and it also adds overhead during integration testing.

Also based on this proposal https://github.com/magento/architecture/blob/c389fc4a6c145283d90e73e2e4c79a8f2a7dbf52/design-documents/graph-ql/coverage/product_filter_and_search_changes.md  which proposes to add an additional field (input_type), that will explain which UI type should be used for the attribute. When the client is going to cache that request, they need to be notified when a new attribute is added so they can clear the cache. This might cause data inconsistencies between client and server.



**Current Implementation (With new Layered Navigation changes):**

The attributes that can be filtered on are generated dynamically. 3 filter input types are added
```
input FilterEqualTypeInput @doc(description: "Specifies which action will be performed in a query ") {
    in: [String]
    eq: String
}

input FilterRangeTypeInput @doc(description: "Specifies which action will be performed in a query ") {
    from: String
    to: String
}

input FilterMatchTypeInput @doc(description: "Specifies which action will be performed in a query ") {
    like: String
    eq: String
}
```
And the attributes belong to any of the 3 types. A dynamically generated schema will look like this
```
input ProductAttributeFilterInput @doc(description: "ProductAttributeFilterInput defines the filters to be used in the search. A filter contains at least one attribute, a comparison operator, and the value that is being searched for.") {
    category_id: FilterEqualTypeInput  @doc(description: "Filter product by category id")
    description: FilterMatchTypeInput
    price: FilterRangeTypeInput
}
```
These attribute codes will be returned by the new response type Aggregation on every search
```
type Aggregation {
    count: Int @doc(description: "The number of filter items in the filter group.")
    label: String @doc(description: "The filter named displayed in layered navigation.")
    attribute_code: String! @doc(description: "Attribute code of the filter item.")
    options: [AggregationOption] @doc(description: "Describes each aggregated filter option.")
}
```
To get the input type for the attribute code, clients have to make a query to customAttributeMetadata which will return the input_type for each attribute_code. This needs to be cached by the clients, but still clients needs to be notified for eviction if new attributes are added.

**Proposal**

Filtering is an output driven approach. The Output Aggregations help in conveying to the clients wha attributes can be filtered on, rather than the schema.

To solve this, instead of modifying the schema dynamically, have a generic filtering input (or Collections) that can consume all possible filtering types. As of now there are only 3 possible filtering types

FilterEqualTypeInput
FilterMatchTypeInput
FilterRangeTypeInput
The filter input query can be modified to accommodate filtering of possible attributes based on above 3 filtering inputs. This model is extensible when new filter type needs to be added.

******
```
type Query {
products (
    search: String @doc(description: "Performs a full-text search using the specified key words."),
    filter: [ProductAttributeFilterInput] @doc(description: "Identifies which product attributes to search for and return."),
    pageSize: Int = 20 @doc(description: "Specifies the maximum number of results to return at once. This attribute is optional."),
    currentPage: Int = 1 @doc(description: "Specifies which page of results to return. The default value is 1."),
    sort: ProductAttributeSortInput @doc(description: "Specifies which attributes to sort on, and whether to return the results in ascending or descending order.")
): Products

}

input ProductAttributeFilterInput {
    attribute_code: String!
    filter_equal: FilterEqualTypeInput
    filter_match: FilterLikeTypeInput
    filter_range: FilterRangeTypeInput
}
```
The output aggregation should also convey what type a filter item belongs to. In the aggregation output, input_type can be added to signify this.
```
aggregations {
      count
      label
      attribute_code
      input_type
      input_code
      options {
        count
        label
        value
      }
    }
```
**Search Before**
```
{

products (
   filter  {

     category_id {

        eq: "category-1"

      }

      price {

         from: "100"

         to: "500"

      }

  }
)

}
```
**Search After**
```
{

products (
   filter:  [{

    attribute_code: "category_id"
    filter_equal: {
        eq: "category-1"  
    }

 },

{

   attribute_code: "category_id"

    filter_range: {
        from: "100"  
        to: "500"
    }

 }

  }]
)

}
```
**Positives**
Schema need not be generated dynamically
When an attribute is added the client need not wait for the cache to be cleaned to filter on
Avoids Data inconsistency between clients and server

**Negatives**

Schema is less rigid
