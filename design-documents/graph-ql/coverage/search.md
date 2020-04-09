# Queries

```graphql
type Query {
    multiSearch(phrase: String!, productSize: Int = 10): MultiSearchResponse!
    #Filter supports multiple clauses which will be wrapped in logical AND operator
    productSearch(phrase: String!, filter: [Clause], sort: ProductSearchSortInput): ProductSearchResponse!
}

input ProductSearchSortInput
{
    relevance: SortEnum
    position: SortEnum
    name: SortEnum
}

input Clause {
    attribute: String!
    in: [String]
    eq: String
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

interface Aggregation {
    name: String!
    buckets: [Bucket]!
}

interface Bucket {
    count: Int!
}

type ScalarBucket implements Bucket {
    title: String!
}

type RangeBucket implements Bucket {
    from: Float!
    to: Float!
}
```