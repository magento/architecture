# [GraphQL][] Schema Development

Before developing a [GraphQL schema](https://graphql.org/learn/schema/), we need to consider the following:

- [GraphQL Ecosystem](#graphql-ecosystem)
- [GraphQL Server](#graphql-server)
- [Code-First vs. SDL](#code-first-vs-sdl)
- [Code-First Frameworks](#code-first-frameworks)
   - [TypeGraphQL](#typegraphql)
   - [Nexus](#nexus)

## GraphQL Ecosystem

[GraphQL][] is [a specification](https://graphql.github.io/graphql-spec/) for which a good handful of implementations are written across various languages (e.g., [Rust](https://github.com/graphql-rust), [PHP](https://github.com/webonyx/graphql-php), [Python](https://graphene-python.org/), [.NET](https://github.com/graphql-dotnet/graphql-dotnet) and [JavaScript](https://www.npmjs.com/package/graphql)). There is no wrong choice between these implementations; however, when considering the ecosystem, there is a clear winner.

In 2015, the [GraphQL][] spec was [first published](https://graphql.github.io/graphql-spec/July2015/) along with the reference implementation in JavaScript. It should be no surprise then, that today, JavaScript holds the strongest ecosystem of [GraphQL][] libraries, with [4,833 related npm packages](https://www.npmjs.com/search?q=graphql) at the time of this writing (10/2019). As such, the JavaScript ecosystem is a solid and safe choice, moving forward.

## GraphQL Server

Within the JavaScript ecosystem, there are [a few server choices available](https://www.npmtrends.com/apollo-server-vs-koa-graphql-vs-prisma), but the most popular and mature choice is [Apollo Server][]. Optionally, [`graphql-yoga`](https://www.npmjs.com/search?q=graphql-yoga) uses [Apollo Server][] under the hood.

## Code-First vs. SDL

As [previously mentioned](#graphql-ecosystem), there are myriad JavaScript libraries available to assist in the creation of a [GraphQL][] server, but before we explore them, we should consider the pros & cons of defining our schema as code-first vs. SDL. Many articles [have been written](#references) about this topic and there are excellent arguments on both sides, but code-first appears to be the direction things are headed.

In this document, we will explore the code-first approach.

> Instead of increasing the complexity of GraphQL server development with more tools, we should strive for a simpler development model. Ideally, one that lets developers leverage the programming language they're already using – [this is the idea of code-first](https://www.prisma.io/blog/the-problems-of-schema-first-graphql-development-x1mn4cb0tyl3?source=post_page-----cf0e50d5ccff----------------------#code-first-a-language-idiomatic-way-for-graphql-server-development).

### Pros

- SDL is a generated artifact, not manually maintained.
- The code-first schema is the single source of truth.
   - No inconsistencies. Schema definition & resolvers are always in sync.
   - No redundancy (duplication).
   - Easier to maintain.
- The shape of the API is closer to the data model.
- Better IDE support & developer experience.
- Better code reuse and type safety.
- Can split by features/modules.
- Terse (less boilerplate).
- Declarative.
- Scalable.

### Cons

- Some of [the latest code-first libraries](https://www.prisma.io/blog/introducing-graphql-nexus-code-first-graphql-server-development-ll6s1yy5cxl5) are not yet mature enough.
- Tightly coupled to a specific language (e.g., [TypeScript][]).

## Code-First Frameworks

Both [TypeGraphQL](#typegraphql) and [Nexus](#nexus) are the key players that provide a code-first solution, but they go about it in entirely different ways. One thing they have in common, however, is that they are designed to use [TypeScript][].

_At this point, we're talking about developing a [GraphQL][] API in [Node.js][] with [TypeScript][]._

### [TypeGraphQL][]

> [TypeGraphQL][] is a library that makes this process enjoyable by defining the schema using only classes and a bit of decorator magic.

```ts
@ObjectType()
class Recipe {
  @Field()
  title: string;

  @Field(type => [Rate])
  ratings: Rate[];

  @Field({ nullable: true })
  averageRating?: number;
}
```

As you can see, the implementation is quite human-readable!

_[TypeGraphQL][] is [compatible](https://www.reddit.com/r/graphql/comments/ap32fv/typegraphql_prisma/) with [Prisma][]._

### [Nexus][]

Schema-first _and_ code-first?

> While being a code-first framework, [Nexus][] can still be used for schema-first development. Schema-first and code-first are not opposing approaches: they become even more useful when combined.

Unfortunately, [Nexus][] is still very new and subject to change before its supposed [Q1 2020 stable release](https://github.com/prisma-labs/nexus/issues/219#issuecomment-530997906). For now, read [Why Nexus?](https://nexus.js.org/docs/why-graphql-nexus) for an idea of what to expect.

## Conclusion

The [GraphQL][] ecosystem has evolved quite a bit, but it has much farther to go. If you need something in production before Q2 2020, you might consider [TypeGraphQL][], but if you can afford to wait? Or if you can at least accept that the API will change? Give [Nexus][] a shot!

> At Novvum, we have been using [Nexus][] pretty extensively... After also trying [TypeGraphQL][], we feel that [Nexus][] has been much friendlier to work with, given us more flexibility, and enabled us to move more quickly ([source](https://dev.to/novvum/typegraphql-and-graphql-nexus-a-look-at-code-first-apis-6k0#which-one-we-prefer)).

## References

- [GraphQL Code-First and SDL-First, the Current Landscape in Mid-2019](https://medium.com/novvum/graphql-code-first-and-sdl-first-the-current-landscape-in-mid-2019-699f68b31a65)
- [The Problems of "Schema-First" GraphQL Server Development | Prisma](https://www.prisma.io/blog/the-problems-of-schema-first-graphql-development-x1mn4cb0tyl3?source=post_page-----cf0e50d5ccff----------------------)
- [Schema-First GraphQL: The Road Less Travelled](https://blog.mirumee.com/schema-first-graphql-the-road-less-travelled-cf0e50d5ccff)
- [Prisma 2 Preview: Type-safe Database Access & Declarative Migrations](https://www.prisma.io/blog/announcing-prisma-2-zq1s745db8i5)
- [TypeGraphQL and GraphQL Nexus &mdash; A Look at Code-First APIs](https://dev.to/novvum/typegraphql-and-graphql-nexus-a-look-at-code-first-apis-6k0)
- [What is GraphQL: History, Components, and Ecosystem](https://levelup.gitconnected.com/what-is-graphql-87fc7687b042)

[apollo]: https://www.apollographql.com/
[apollo server]: https://www.apollographql.com/docs/apollo-server/
[graphql]: https://graphql.org/
[nexus]: https://www.prisma.io/blog/introducing-graphql-nexus-code-first-graphql-server-development-ll6s1yy5cxl5
[node.js]: https://nodejs.org/
[prisma]: https://www.prisma.io/with-graphql/
[typegraphql]: https://typegraphql.ml/
[typescript]: http://www.typescriptlang.org/
