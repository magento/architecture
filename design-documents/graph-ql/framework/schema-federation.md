# Apollo Federation

## An alternative to Magento schema stitching

[Apollo Federation][] is a replacement for their [schema stitching](https://www.apollographql.com/docs/graphql-tools/schema-stitching/). Schema federation is intended to:

- [Solve modularity and extensibility challenges of schema stitching](https://www.apollographql.com/docs/apollo-server/federation/migrating-from-stitching/)
- [Allow code separation by concern](https://www.apollographql.com/docs/apollo-server/federation/concerns/)
- [Provide support for distributed GraphQL](https://principledgraphql.com/integrity#2-federated-implementation)

## [Apollo Federation][] vs Magento server-side schema stitching

### Feature parity

- [Modularity](https://principledgraphql.com/integrity#2-federated-implementation)
- [Types extensibility](https://www.apollographql.com/docs/apollo-server/federation/core-concepts/#extending-external-types)
- [Separation of concerns](https://www.apollographql.com/docs/apollo-server/federation/concerns/)

### Benefits of [Apollo Federation][]

- [Works with microservice architecture](https://principledgraphql.com/integrity#2-federated-implementation)
- [Mechanism for entity reference resolution between independent services](https://www.apollographql.com/docs/apollo-server/federation/core-concepts/#entities-and-keys)

### Concerns

- No implementation of federation gateway in PHP
- Requires refactoring of existing schema. Potentially in a backward incompatible manner

## Migration path

To start supporting federated schema, we need to:

1. Rewrite modular GraphQL schemas to include [entity `@key` directives](https://www.apollographql.com/docs/apollo-server/federation/core-concepts/#entities-and-keys) and [type extensions using `extend type` syntax](https://www.apollographql.com/docs/apollo-server/federation/core-concepts/#referencing-external-types).
1. Implement federation gateway in PHP according to the [federation spec](https://www.apollographql.com/docs/apollo-server/federation/federation-spec/). Its responsibility is to generate a final schema and execute federated queries (resolve references).

## Resolution

Considering that Magento is a monolithic application which supports GraphQL schema modularity and extensibility using its custom implementation of server-side schema stitching algorithm, it does not make sense to implement schema federation yet; however, schema federation may be needed when Magento is decomposed into multiple domain services.

### Apollo Gateway

Using an [Apollo Gateway](https://www.apollographql.com/docs/apollo-server/api/apollo-gateway/), we can expose a single endpoint that captures data across multiple underlying GraphQL microservices (federated services), each of which may be written in a different language.

```ts
import { ApolloGateway } from "@apollo/gateway";

const gateway = new ApolloGateway({
  serviceList: [
    {
      name: "products",
      url: "http://magento.com/products"
    },
    {
      name: "inventory",
      url: "http://magento.com/inventory"
    }
  ]
});
```

Alternatively, we can merge local schema files into one by using the [`localServiceList`](https://spectrum.chat/apollo/apollo-federation/is-the-localservicelist-option-intentionally-undocumented~2ea5d8b5-1d75-4485-9445-9371df804a58):

```ts
import fooTypeDefs from './foo.gql'
import barTypeDefs from './bar.gql'

const gateway = new ApolloGateway({
  localServiceList: [
    { name: "foo", typeDefs: fooTypeDefs },
    { name: "bar", typeDefs: barTypeDefs }
  ]
});
```

Unfortunately, this distributed architecture has more complexity and points of failure than operating a monolith. As such, we need to be strategic about [how we run, manage and deploy a federated graph](https://www.apollographql.com/docs/platform/federation/).

## Conclusion

The [Apollo Federation][] ecosystem provides exciting opportunities and ideas for our business needs; yet, it's still very new and needs time to mature. Some of the features on [the roadmap](https://github.com/apollographql/apollo-server/blob/master/ROADMAP.md) appear to be open source in nature; however, discussions on the [Spectrum channel](https://spectrum.chat/apollo/apollo-federation?tab=posts) indicate a trajectory that implies [paid-for services](https://spectrum.chat/apollo/apollo-federation/federated-schemas-changes-require-gateway-redeploy~4a839c03-4549-43df-975d-a6732c255707) will pave the road for complex, enterprise solutions. 

If we decide to migrate to [Apollo Federation][] we also need to decide whether to invest in these paid-for services or whether to roll our own.

[Apollo Federation]: https://www.apollographql.com/docs/apollo-server/federation/introduction/
