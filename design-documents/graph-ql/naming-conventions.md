# GraphQL Naming Conventions

The following naming guidelines should be followed when adding or modifying schemas in the Magento platform.

These guidelines are based on the conventions already used within the schema. Because of this, there will be places where the style diverges from more common styles found in the GraphQL community.

## Object/Interface Types

Object/Interface names should always be _PascalCase_.

```graphql
# PascalCase
type TwoWords {}
interface TwoWords {}
```

## Object/Input Field Names

Object/Input field names should always be _snake_case_.

```graphql
type Query {
    # snake_case
    two_words: String
}

input Data {
    # snake_case
    two_words: String
}
```

## Arguments

Argument names should always be _camelCase_.

```graphql
type Query {
    example(
        # camelCase
        twoWords: String
    ): String
}
```

## Top-level Fields on Query and Mutation

Top-level fields on Query and Mutation should always be _camelCase_.

```graphql
type Query {
    # camelCase
    twoWords: TwoWords
}

type Mutation {
    # camelCase
    twoWords(input: TwoWordsInput!): TwoWordsOutput
}
```

## ID Fields

These rules should apply to any field assigned the scalar type `ID`, both in Object Types and Input Object Types.

There are two types of `ID` fields:

- **Primary Identifier** : An ID owned by the current Object Type
- **Foreign Identifier**: An ID referencing another Object Type

A _Primary Identifier_ should _always_ be given the field name `uid`.
A _Foreign Identifier_ should _always_ be given the field name `source_object_uid`, where `source_object` is a _snake_case_ string referring to the type that owns the `ID` (either an Object or an Interface)

```graphql
type ProductInterface {
    # Good, ProductInterface owns `uid`
    uid: ID!
}

type ConfigurableProductOptions {
    # Good, field refers to ID on `ProductInterface`
    product_interface_uid: ID!
}
```

For additional context on ID naming requirements, see [the ID Improvement Plan](https://github.com/magento/architecture/blob/master/design-documents/graph-ql/id-improvement-plan.md_)