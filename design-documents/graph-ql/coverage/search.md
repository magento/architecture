# Queries

```graphql
type Query {
    multiSearch(phrase: String!, productSize: Int = 10): MultiSearchResponse!
    #Filter supports multiple clauses which will be wrapped in logical AND operator
    productSearch(phrase: String!, filter: [SearchClauseInput], sort: ProductSearchSortInput): ProductSearchResponse!
}

input ProductSearchSortInput
{
    relevance: SortEnum
    position: SortEnum
    name: SortEnum
}

# If from or to fields are omitted, $gte or $lte filter will be applied
type SearchRangeInput {
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
    facetsValues: [Aggregation]
}

type MultiSearchResponse {
    products: ProductSearchResponse
}

type ProductSearchItem {
    product: ProductInterface!
    highlights: [Highlight]
}

type Highlight {
    attribute: String!
    value: String!
    matchedWords: [String]!
}

interface Bucket {
    #Human readable bucket title
    title: String!
}

type StatsBucket implements Bucket {
    #Could be used for filtering and may contain non-human readable data
    id: ID!
    min: Int!
    max: Int!
}

type ScalarBucket implements Bucket {
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