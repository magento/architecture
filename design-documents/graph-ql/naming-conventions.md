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