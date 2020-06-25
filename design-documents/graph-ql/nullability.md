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

Instead, an error in a resolver for a non-nullable field propagates up to the nearest nullable ancestor field. This can end up causing a client to lose more data farther up the response tree, even though that data may have been resolved without errors.

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
    related_products: [RelatedProduct!]!
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
          id
          name
          price
          url
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

## Recommendations

### Scalars

If the field's value is obtained from a backing data source, and that value is required in the backing data source, then the field in the GraphQL schema should be non-nullable

#### Example
```graphql
type Product {
   id: ID! # Products service would not allow a product without an ID, so id is non-nullable
   color: String # Backing products DB does not require color, so it's nullable
}
```

### Object Types

Object type fields are frequently used to create joins between types. Object type fields should be nullable when the backing data is likely to live in a different service from the parent object type. This rule will require more thought since it depends where we think domain boundaries will be drawn as we decompose to services.

#### Example
```graphql
type Product {
    price: Price! # non-nullable, because price is highly likely to be part of a products service
	recommended_products: ProductRecommendations # nullable, because product recommendations would likely be a function that lives outside of the products service
}
```

### List Types

List fields have [2 distinct forms of nullability](http://spec.graphql.org/draft/#sec-Combining-List-and-Non-Null):

1. Field nullability (whether field can return `null` instead of a list)
2. List nullability (whether a list can have `null` items inside it)

These forms can be composed together in various ways:
```graphql
type Example {
    foo1: [Foo]   # foo1 can be null or a list. If list, can have nulls in it
    foo2: [Foo!]! # foo2 must be a list, and every entry must be a 'Foo'
    foo3: [Foo!]  # foo3 can be a list or null. If list, every item must be a 'Foo'
    foo4: [Foo]!  # foo4 must be a list, and the list can have null values
}
```

There are 2 rules of thumb to follow. Note that these can be combined together depending on the use-case

#### Non-Nullable List _Field_

A list _field_ should be non-nullable when its absence would make the parent type fairly useless.

An example is a `Product` type with an `options` field. When rendering a product detail page and asking for `Product.options`, you typically wouldn't want to keep rendering the page if `options` fails, because a shopper can't do much with a configurable product without the options.

If a product has no options at all, an empty list can still be returned (`null` would only be used to represent an error state).

The syntax for a non-nullable list _field_ is `fieldname: [TypeName]!`

#### Non-Nullable List

A _list_ should be non-nullable when the absence of at least 1 item in the list would make the parent type useless.

Using the `Product.options` example again, it's extremely rare that you would want to render a product page with only _some_ of the configured options - what if you're missing a required one in the list?

The syntax for a non-nullable _list_ is `[TypeName!]`
