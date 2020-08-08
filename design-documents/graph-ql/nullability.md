# Nullability in GraphQL

By default, all field values in GraphQL are _nullable_

```graphql
type Query {
    product(id: ID): Product # Server can also return "null"
}
```

GraphQL provides a wrapper/modifier that can be used to disallow null values. It's opt-in on a per-field basis

```graphql
type Query {
    product(id: ID): Product! # Server cannot return "null"
}
```

## Why non-nullable?

Non-nullable fields help form a contract with clients that makes it safe for them to work with a field's value without having to inspect its shape for possible `null` values.

For clients that are written in a language with a static type system, the compiler will force them to wrap their code in conditionals anytime they're accessing a possibly null field. These checks can become tedious if every single field is nullable.

For clients that are written in more dynamic languages (JavaScript), developers need to be very aware of nullable fields in the schema to prevent trying to accessing data on a `null` value. This leads to lots of defensive coding (or bugs).

Making fields non-nullable alleviates some of these issues for clients.

## Why nullable?

When a field's resolver has an error, GraphQL requires that the value of the field be set to `null`. But, if a field is non-nullable, a `null` value would break that contract.

Instead, an error in a resolver for a non-nullable field propagates up to the nearest parent field. It that field is also non-nullable, the error propagates up to the next parent field. This propagation continues until either:

- A nullable parent field is found
- We reach the root `Query` object

 A non-nullable type in the wrong place can end up causing a client to lose more data farther up the response tree, even though that data may have been resolved without errors.

_[Further reading on the relationship between non-null types and errors](http://spec.graphql.org/draft/#sec-Errors-and-Non-Nullability)_

### Example: Bad use of non-nullable field

In this example, a client wants to query for data to render a product details page. But, because the `related_products` field is non-nullable, the client will lose access to data like `name` and `price` if `related_products` throws any errors.

Ideally, a UI would get the critical product data, render the page, and just not render the "Related Products" display.

**Schema**:
```graphql
type Query {
   product(id: ID): Product
}

type Product {
    id: ID!
    name: String!
    price: Money!
    description: String!
    url: String!
    related_products: RelatedProducts!
}
```
**Query**:
```graphql
query ProductDetailsPage($id: ID) {
    product(id: $id) {
       id
       name
       price
       description
       # related_products can't be null. When the backing service is
       # down/unreachable at the time this query runs, we can't fulfill
       # the contract
       related_products {
           items {
               id
               name
               price
               url
           }
       }
    }
}
```

**Response**

Note that no data was obtained by the client for `product`
```js
{
   "data": {
      "product": null
   },
   "errors": [
      // details about failure in related_products field populated here
   ]
}
```

## Backwards Compatibility

- It is a safe, non-breaking change to move a field from nullable to non-nullable
- It is an unsafe, breaking change to move a field from non-nullable to nullable

With this in mind, when in doubt, the safer route is to make a field nullable, with the agreement that we'll iterate and improve schemas as we get feedback on usage from clients.

**Note**: The [GraphQL reference implementation](https://github.com/graphql/graphql-js) has a `findBreakingChanges` feature that can be inspected to see a [list of all known schema changes that cause breaks for clients](https://github.com/graphql/graphql-js/blob/2232ebdef828566f3add3ed2a31709d3c1710c0e/src/utilities/findBreakingChanges.js#L41-L67). It [explicitly notes](https://github.com/graphql/graphql-js/blob/2232ebdef828566f3add3ed2a31709d3c1710c0e/src/utilities/findBreakingChanges.js#L461) that moving from a nullable value to a non-nullable value of the same type is safe.

## Recommendations

### Top-level Query fields should always be nullable

Because of error propagation, if an error occurs in a non-null field on the `Query` root, the client will lose data from all other top-level queries.

```graphql
type Query {
    # If a client queries for both fields, and one of them fails,
    # the client will receive no data
    product(id: ID): Product!
    category(id: ID): Category!
}
```

### Primary keys (id field) should always be non-nullable

It very rarely makes sense to have a resource that _can_ have an ID but might not.

IDs are extremely important for caching in most GraphQL clients, so it's worthwhile to be safe here.

```graphql
type Product {
    id: ID! # Rarely makes sense for this to be nullable
}
```

### Scalars should be non-nullable when required in backing data source

Scalars form the leaves of a response in GraphQL, and are the bits of data the client really cares about. Because of this, we should strive to make scalars non-nullable when possible.

The exceptions to this rule are:

- A scalar field _should_ be nullable if it can be empty in the application/backing data source
- A scalar field _should_ be nullable if it's likely to come from a different service than its parent object (fairly rare)


### Consider the Parent Type

If you're not dealing with an `id` field or a top-level `Query` field, the most important question to ask is:

**Is the parent object still usable if this field has an error?**

#### Example: Parent still usable with field error

```graphql
type Product {
    # Recommended products are not critical data on a product page, and a UI can represent
    # a product safely without related products, so we keep the field nullable
    recommended_products: ProductRecommendations
}
```

#### Example: Parent not usable with field error

```graphql
type Product {
    # A user would not be able to add a product to the cart from the Product
    # details page if this field fails, because it may have required options.
    # We make the field's type non-nullable
    options: ProductOptions!
}
```

### List Types

List fields have [2 distinct forms of nullability](http://spec.graphql.org/draft/#sec-Combining-List-and-Non-Null):

1. Field nullability (same as all other fields)
2. List item nullability (whether a list can have `null` items inside it)

These forms can be composed together in various ways:
```graphql
type Example {
    foo1: [Foo]   # foo1 can be null or a list. If list, can have nulls in it
    foo2: [Foo!]! # foo2 must be a list, and every entry must be a 'Foo'
    foo3: [Foo!]  # foo3 can be a list or null. If list, every item must be a 'Foo'
    foo4: [Foo]!  # foo4 must be a list, and the list can have null values
}
```

To decide whether the _field_ should be nullable, use the other recommendations provided in this document.

When deciding whether _List items_ should be nullable, the most important question to ask is:

**"Is the parent object still usable if at least one item in this list has an error?"**

#### Example: Parent not usable if an item in List has an error

```graphql
type Product {
   # Note: The "!" inside of the List ([]) means the list items are non-nullable
   # Making the list items non-nullable guarantees to the client that, if they receive
   # a list of product options, it will be complete/without errors
   options: [ProductOption!]!
}
```

#### Example: Parent still usable if an item in List has an error

```graphql
type Product {
    # The absence of a "!" inside the list means that we could fail
    # to fetch a nested field in a related product, and it won't
    # impact our ability to render the rest of the product page
    related_products: [RelatedProduct]
}
```