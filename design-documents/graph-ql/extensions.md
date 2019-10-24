# GraphQL Extensions

Extension developers need a way to extend our GraphQL API. This document is a proposal of how this can be done in the context of a _Node.js_ GraphQL API, should we decide to go in that direction.

## Package registry

First, we need a place to publish both public and private JavaScript extensions and in a secure way. At the time of this writing, [npm][] is the de facto package registry for JavaScript; however, the [GitHub Package Registry][] is currently in beta and offers the following features:

- Safely publish and consume packages within your organization or with the entire world.
- Publish privately for your team or publicly for the open source community.
- Discover and publish public and private packages in one place.
- Seamlessly use and reuse any package as a dependency in a project by downloading it straight from GitHub.

As merchants currently have the ability to install public extensions or purchase private ones, having them both published all in one place is extremely beneficial for our needs.

_This topic needs further investigation after the official release, which is expected to be during the [GitHub Universe](https://githubuniverse.com/) event, November 13-14, 2019._

## Configuration

Once the merchant has installed any number of extensions, this list needs to be provided to the GraphQL API server, preferrably merged with the existing [`package.json`](https://docs.npmjs.com/files/package.json):

```json
  "dependencies": {
    "top-products": "1.0.0"
  },
  "magento-extensions": [
    "top-products"
  ],
```

This way, the list of extensions will be installable and discoverable to the Node.js application.

## Extending Apollo Server

The [Apollo Server][] configuration object is already extensible in the same way that any JavaScript object is extensible, but we need to be careful about how we do it. We also need to make it as clear and painless as possible for the best developer experience

### Extend vs. modify

Historically, Magento extension developers have had the ability to _modify_ default functionality without limitations; however, once this ability is granted, we simply can't take it away.

With a new GraphQL API, however, we have an opportunity to take a fresh look at the features and limitations (if any) that we require. We need to consider the implications of what would happen if we followed the [open/closed principle][] until it proves insufficient, instead of _assuming_ that modification is required. The [open/closed principle][] is one of the five [SOLID](https://en.wikipedia.org/wiki/SOLID) principles of object-oriented design.

> Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.

We can follow this principle in the design of our GraphQL API by providing base or default fields, mutations, etc. **Extension developers may extend them with any number of additional fields or mutations, but the default ones shall never change.** This is similar to [the GraphQL specification's strong opinions about versioning](https://graphql.org/learn/best-practices/#versioning), in that you are encouraged to add more fields to solve a problem instead of changing/breaking existing ones.

#### Discussion points

- If this is the route we take (i.e., extend, don't modify), do we also limit extensions from _modifying_ fields that other extensions have exposed? Or do we throw errors when there are conflicts?
- If we allow _modification_, do we continue to call them extensions or find a better name that would more accurately represent the mutable nature of these &hellip; plugins?

### Developer experience

In order to provide the best developer experience, we should create and publish some simple utilities that extension developers can use, both for convenience and to keep them on the right track. To this end, [TypeScript][] will provide [the most rich editor experience](https://code.visualstudio.com/docs/editor/intellisense), even if they are using JavaScript to consume it.

At the very least, extension developers should be able to extend the `typeDefs`, `resolvers` and `context` of an [Apollo Server][] configuration object. If we were to publish a `createExtension` utility function, the implementation of it could be as simple as this:

```ts
import { createExtension, gql } from 'magento-utils'

export default createExtension({
  typeDefs: gql`...`,
  resolvers: {},
  context: {},
})
```

Because the utilities are written in [TypeScript][], we can generate and publish JavaScript in tandem with automatically generated [type definitions](http://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html). These definitions provide the best convenience for developers. For example, if they attempt to access the context object in a resolver, not only will they see the shape of their own context object, but it will be combined with Magento's default context object, which contains properties for things like ElasticSearch and SQL.

<details>
<summary>Click to see utility code</summary>

### Magento utilities

```ts
import { GraphQLResolverMap } from 'apollo-graphql'
export { gql } from 'apollo-server'
import { ContextFunction, Context } from 'apollo-server-core'
import { ExpressContext } from 'apollo-server-express/dist/ApolloServer'
import { DocumentNode } from 'graphql'

import context from '../context'

/**
 * A convenience function to ensure your extension properly implements the
 * `Extension` interface. It also merges your `context` type with the root
 * context and feeds it into your resolvers, again, for convenience.
 */
export default function createExtension<T extends {} = {}>(
  options: Extension<T>,
) {
  return options
}

export interface Extension<T = object, U = T & typeof context> {
  context?: ContextFunction<ExpressContext, Context<T>> | Context<T>
  resolvers?: GraphQLResolverMap<Context<U>>
  typeDefs?: DocumentNode
}
```

</details>

## Conclusion

TBD

[apollo server]: https://www.apollographql.com/docs/apollo-server/
[github package registry]: https://github.com/features/package-registry
[npm]: https://www.npmjs.com/
[open/closed principle]: https://en.wikipedia.org/wiki/Open%E2%80%93closed_principle
[typescript]: http://www.typescriptlang.org/
