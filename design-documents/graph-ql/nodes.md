# GraphQL Nodes - Global Object Identification

To provide options for GraphQL clients to elegantly handle caching and data refetching, GraphQL servers need to expose object identifiers in a standardized way.

For this to work, a client will need to query via a standard mechanism to request an object by ID. Then, in the response, the schema will need to provide a standard way of providing these IDs.

Example
```graphql
{
  node(uid: "MTM0MzQ=") {
    id
    ... on SimpleProduct {
      name
      uid
    }
    ... on CategoryTree {
      name
      children
    }
  }
}
```

Nodes IDs should be unique throughout the entire schema and ideally for every store since store code is unique

To implement this the uid has to be composed out of:
```
base64Encode("StoreCode/__typeName/id")
```

This way we can know what is the type requested and render it to the actual type.
It will help our schema to be more compliant as all entities should be nodes.

More information in the Graphql Spec
- [https://graphql.org/learn/global-object-identification/](https://graphql.org/learn/global-object-identification/)


Schema implementation
- [nodes.graphqls](coverage/nodes.graphqls)
