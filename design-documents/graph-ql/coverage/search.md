# Queries

```graphql
type Query {
    multiSearch(phrase: String!, productSize: Int = 10): MultiSearchResponse!
    #Filter supports multiple clauses which will be wrapped in logical AND operator
    productSearch(phrase: String!, filter: [SearchClauseInput], sort: [ProductSearchSortInput]): ProductSearchResponse!
}

input ProductSearchSortInput
{
    attribute: String!
    direction: SortEnum!
}

# If from or to fields are omitted, $gte or $lte filter will be applied
input SearchRangeInput {
    from: Int
    to: Int
}

input SearchClauseInput {
    attribute: String!
    in: [String]
    eq: String
    range: SearchRangeInput
}

type ProductSearchResponse {
    items: [ProductSearchItem]
    hits: Int!
    facets: [Aggregation]
    facets_values: [Aggregation]
}

type MultiSearchResponse {
    products: ProductSearchResponse
    suggestions: [String]
}

type ProductSearchItem {
    product: ProductInterface!
    highlights: [Highlight]
}

type Highlight {
    attribute: String!
    value: String!
    matched_words: [String]!
}

interface Bucket {
    #Human readable bucket title
    title: String!
    attribute: String!
}

type StatsBucket implements Bucket {
    min: Int!
    max: Int!
}

type ScalarBucket implements Bucket {
    #Could be used for filtering and may contain non-human readable data
    id: ID!
    count: Int!
}

type RangeBucket implements Bucket {
    from: Float!
    to: Float!
    count: Int!
}

interface Aggregation {
    name: String!
    buckets: [Bucket]!
}
```