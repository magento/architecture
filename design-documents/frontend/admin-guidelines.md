# Magento Admin Guidelines

## Problem

There are already a handful of React applications built for Magento Admin use, but they have different dependencies and versions between them and we don't yet have a micro front-end architecture to manage them.

## Solution (for now)

We must do our best to align common dependencies manually.

The following [`package.json`](#packagejson) includes a list of dependencies that you should use instead of their alternatives. Please also stick within the version ranges specified to support a graceful transition to a future architecture.

To aid in the process of using these dependencies, consider using the [TSX Yeoman Generator](https://www.npmjs.com/package/generator-tsx).

For more information behind these technology choices, refer to the [Technical Vision](./technical-vision.md).

## package.json

Each section of the `package.json` file has been broken down into different topics.

- [version](#version)
- [scripts](#scripts)
- [browserslist](#browserslist)
- [dependencies](#dependencies)
  - [Utility functions](#utility-functions)
  - [SSR](#ssr)
  - [React](#react)
  - [State management](#state-management)
  - [Code splitting](#code-splitting)
  - [i18n](#i18n)
  - [Theming](#theming)
  - [GraphQL client](#graphql-client)
  - [TypeScript import helpers](#typescript-import-helpers)
- [devDependencies](#devdependencies)
  - [Craco](#craco)
  - [Redux mock store](#redux-mock-store)
  - [Testing library](#testing-library)

### version

The version of this `package.json` follows [semver](https://semver.org/) and will change with this document accordingly.

```json
"version": "0.1.0",
```

### scripts

Customize your CRA configuration with [craco][].

- [Don't eject](https://create-react-app.dev/docs/alternatives-to-ejecting).
- Avoid forking, because you can oftentimes do the same thing by creating your own [craco][] plugin. See [craco plugins on npm](https://www.npmjs.com/search?q=craco%20plugin) and the [craco recipes](https://github.com/sharegate/craco/tree/master/recipes) for guidance.

```json
"scripts": {
  "start": "craco start",
  "build": "craco build",
  "test": "craco test"
},
```

### [browserslist](https://github.com/browserslist/browserslist#readme)

Don't forget your [polyfills](https://create-react-app.dev/docs/supported-browsers-features#supported-language-features)!

```json
"browserslist": [
  ">0.2%",
  "not dead",
  "not ie < 11",
  "not op_mini all"
],
```

Refer also to [Magento DevDocs | Supported browsers](https://devdocs.magento.com/guides/v2.3/install-gde/system-requirements_browsers.html).

### dependencies

#### Utility functions

Use vanilla (native) JS if possible, utility functions if not. [You don't (may not) need Lodash/Underscore](https://github.com/you-dont-need/You-Dont-Need-Lodash-Underscore#you-dont-may-not-need-lodashunderscore-). See also [just](https://github.com/angus-c/just#just).

```json
"@queso/camel-case": "^0",
"@queso/kebab-case": "^1",
```

#### SSR

Prepare for [SSR](https://medium.com/walmartlabs/the-benefits-of-server-side-rendering-over-client-side-rendering-5d07ff2cefe8) by using [universal fetching](https://www.npmjs.com/package/ky-universal). Don't use [axios](https://www.npmjs.com/package/axios) or [fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) directly.

```json
"ky": "^0",
"ky-universal": "^0",
```

#### React

```json
"react": "^16",
"react-dom": "^16",
"react-helmet": "^5",
```

#### Routing

See [The Future of React Router and @reach/router](https://reacttraining.com/blog/reach-react-router-future/). Basically, the new API will be much closer to `@reach/router` than React Router, so this prepares us for that future.

```json
"@reach/router": "^1",
```

#### State management

[Immutability Helper](https://www.npmjs.com/package/immutability-helper) was [rewritten in TypeScript](https://github.com/kolodny/immutability-helper/pull/123) and strikes a balance between features and size.

```json
"immutability-helper": "^3",
"react-redux": "^7",
"redux": "^4",
"redux-thunk": "^2",
```

#### Code splitting

For context, read [Optimize your React application with Loadable Components](https://www.smooth-code.com/articles/code-splitting-react-loadable-components) and don't use dynamic imports in your components.

```json
"@loadable/component": "^5",
```

#### i18n

```json
"react-intl": "^3",
```

#### Theming

```json
"react-theme-context": "^2",
```

#### GraphQL client

[Why a GraphQL client?](https://www.prisma.io/blog/relay-vs-apollo-comparing-graphql-clients-for-react-apps-b40af58c1534#why-a-graphql-client)

```json
"babel-plugin-relay": "^5",
"react-relay": "^5",
```

#### TypeScript import helpers

```json
"tslib": "^1",
```

Combined with the following TypeScript [compiler options](https://www.typescriptlang.org/docs/handbook/compiler-options.html):

```json
{
  "compilerOptions": {
    "importHelpers": true,
    "noEmitHelpers": true,
  }
}
```

### devDependencies

#### [Craco][]

```json
"@craco/craco": "^5",
```

#### Redux mock store

This is [a TypeScript fork](https://www.npmjs.com/package/@jedmao/redux-mock-store) of a somewhat stale, but highly successful package that doesn't require much maintenance.

```json
"@jedmao/redux-mock-store": "^2",
```

#### Testing library

Don't use [enzyme](https://airbnb.io/enzyme/). Read [the problem statement of React Testing Library](https://testing-library.com/docs/react-testing-library/intro).

```json
"@testing-library/react": "^9",
```

## Feedback

Please [submit an issue](https://github.com/magento/architecture/issues) if you want to have a discussion or suggest a change to any of these technology choices!

[craco]: https://www.npmjs.com/package/@craco/craco
