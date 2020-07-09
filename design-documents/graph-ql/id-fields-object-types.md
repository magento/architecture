# ID Fields in Object Types

When a GraphQL Object Type is used to represent an entity that can be referenced by ID, it _should_ follow these best-practices.

## Best Practices

### Do

- Use the `ID` scalar type
- Make the `ID` field non-nullable (syntax: `ID!`)
- Use the field name `id` or `_id`
    - Most GraphQL client libs automatically use this field for caching. Any other field name will require manual caching logic on the client
        - https://www.apollographql.com/docs/react/caching/cache-configuration/#assigning-unique-identifiers
        - https://formidable.com/open-source/urql/docs/graphcache/normalized-caching/#key-generation
- Include an ID field anytime an Object Type _could_ have an ID field

### Do Not
- Include info in the client-facing description describing how the field is encoded/decoded (should be opaque)
    - Example: The client shouldn't know if an `ID` is a base64-encoded integer
- Use `String` or `Int` type
- Use duplicate IDs across a concrete type. In other words, if the client wants to produce a cache key, the concetenation of a `__typename` + `id` field should _always_ be unique

## Examples where an ID is not helpful

- Wrapper types, like `SearchResultPageInfo`