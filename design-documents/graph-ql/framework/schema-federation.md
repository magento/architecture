## Apollo schema federation as an alternative to Magento schema stitching

Apollo [schema federation](https://www.apollographql.com/docs/apollo-server/federation/introduction/) is a replacement for their [schema stitching](https://www.apollographql.com/docs/graphql-tools/schema-stitching/). Schema federation is intended to:
- Solve modularity and extensibility challenges of schema stitching
- Allow code separation by concern
- Provide support for distributed GraphQL

## Apollo federation vs Magento server-side schema stitching

Feature parity:
- Modularity
- Types extensibility
- Separation of concerns

Benefits of Apollo Federation:
- Works with microservice architecture
- Mechanism for entity reference resolution between independent services

Concerns:
- No implementation of federation gateway in PHP
- Requires refactoring of existing schema. Potentially in a backward incompatible manner

## Migration path from Magento schema stitching to Apollo federation

To start supporting federated schema, we need to:
1. Rewrite modular GraphQL schemas to include entity `@key` directives and type extensions using `extend type` syntax
1. Implement federation gateway in PHP according to the [federation spec](https://www.apollographql.com/docs/apollo-server/federation/federation-spec/). Its responsibility is to generate final schema and execute federated queries (resolve references).

## Resolution
Considering that Magento is a monolithic application which supports GraphQL schema modularity and extensibility using its custom implementation of server-side schema stitching algorithm, it does not make sense to implement schema federation yet. However, schema federation may be needed when Magento is decomposed into multiple domain services.


