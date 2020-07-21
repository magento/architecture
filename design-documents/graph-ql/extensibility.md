**Extensibility**

## GraphQL Vocabulary
- _Root Query_ - [Spec](http://spec.graphql.org/June2018/#sec-Query) | [Docs](https://graphql.org/learn/execution/#root-fields-resolvers)
- _Mutation_ - [Spec](http://spec.graphql.org/June2018/#sec-Mutation) | [Docs](https://graphql.org/learn/queries/#mutations)
- _Object Type_ - [Spec](http://spec.graphql.org/June2018/#sec-Objects) | [Docs](https://graphql.org/learn/schema/#object-types-and-fields)
- _Arguments_ - [Spec](http://spec.graphql.org/June2018/#sec-Field-Arguments) | [Docs](https://graphql.org/learn/schema/#arguments)
- _Input Object_ - [Spec](http://spec.graphql.org/June2018/#sec-Input-Object-Values) | [Docs](https://graphql.org/learn/schema/#input-types)

## Magento GraphQL Vocabulary

- _Output Object_: An _Object Type_ used as the response type for a mutation (primarily to support extensibility)


## Extensibility Requirements

It should be possible to make the following changes to a schema, _without_ introducing breaking changes:

- Add a new root query (backwards-compatible)
- Add a new mutation (backwards-compatible)
- Add optional arguments to a query/mutation (some backwards-compat risks)
- Add extra data to the output of a mutation (backwards-compatible)

The following patterns should be followed to ensure our schema remains extensible, with _minimal_ (ideally no) breaking changes.

## Backwards-Compatible GraphQL Schema Development Patterns

### Mutation Output Objects
All _mutations_ should return an "Output Object," rather than some concrete type. An "Output Object" is just an Object Type with the specific purpose of returning data _and_ metadata related to a mutation.

It is not possible for a client to send a mutation _and_ separate root queries in the same request. Because of this, it's critical that the output of a mutation be able to add more data over time, as client needs expand.

#### Example - Bad
```graphql
type Mutation {
    createFoo: Foo
}
```

#### Example - Good
```graphql
type Mutation {
    createFoo: FooOutput
}

type ExampleOutput {
    # Extensions (and core) can extend with more fields at a later date
    foo: Foo
}
```

### Multiple Arguments

If a query or mutation accepts (or will likely accept) > 1 argument, an Input Object should be used instead, and given the argument name `input`.

#### Justification

When using a query or mutation. it's common for clients to create [Named Operation Definitions](http://spec.graphql.org/June2018/#sec-Named-Operation-Definitions). When a query/mutation takes several arguments, the types (and their defaults) have to be kept in sync with the schema:

```graphql
query ClientGetFooOperationNotNice(
    $arg1: String!
    $arg2: Int
    # If the schema has a default value, it won't be used unless it's re-defined here
    $arg3: String = "test"
) {
    getFoo(
        arg1: $arg1
        arg2: $arg2
        arg3: $arg3
    ) {
        # field selection
    }
}
```

Instead, using an Input Object, this can be simplified without loss of functionality:

```graphql
query ClientGetFooOperationNice($input: GetFooInput) {
    # Note the client no longer has to manually keep operation arg definitions
    # and default schema values in sync
    getFoo(input: $input) {
        # field selection
    }
}
```

#### Example - Bad
```graphql
type Query {
    getFoo(
        arg1: String!
        arg2: Int
        arg3: String = "test"
    )
}
```

#### Example - Good
```graphql
type Query {
    getFoo(input: GetFooInput): Foo
}

input GetFooInput {
    # Extensions (and core) can add additional fields if they are nullable/optional
    arg1: String!
    arg2: Int
    arg3: String = "test"
}
```

### Extending Arguments or Input Objects

Magento Framework, unlike many GraphQL server frameworks, allows extending both arguments lists and Input Object types. This can be powerful, but can also easily become a source of backwards-incompatible changes.

When adding a new field to an arguments list or Input Object type, it should:

- Be optional/nullable
- Have identical resolver logic when the new field is not provided by a client

When modifying arguments lists or Input Object Types, you should _not_

- Change the types of any existing arguments/input object fields
- Change the nullability of any existing arguments/input object fields
- Change the name of any existing arguments/input object fields
