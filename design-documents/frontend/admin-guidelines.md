# Magento Admin Guidelines

## Problem

There are already a handful of React applications built for Magento Admin use, but they have different dependencies and versions between them and we don't yet have a micro front-end architecture to manage them.

## Solution (for now)

We must do our best to align common dependencies manually.

The following [`package.json`](#packagejson) includes a list of dependencies that you should use instead of their alternatives. Please also stick within the version ranges specified to support a graceful transition to a future architecture.

To aid in the process of using these dependencies, consider using the [TSX Yeoman Generator](https://www.npmjs.com/package/generator-tsx).

For more information behind these technology choices, refer to the [Technical Vision](./technical-vision.md).

## package.json

```jsonc
{
  "version": "0.1.0",

  "scripts": {
    // https://www.npmjs.com/package/@craco/craco
    "start": "craco start",
    "build": "craco build",
    "test": "craco test"
    // don't eject
  },

  // npm install --save-dev core-js
  // https://github.com/browserslist/browserslist#readme
  "browserslist": [
    ">0.2%",
    "not dead",
    "not ie <= 11",
    "not op_mini all"
  ],

  "dependencies": {

    // Some utility functions (use native JS, don't install Lodash)
    "@queso/camel-case": "^0.2",
    "@queso/kebab-case": "^1",

    // Universal fetching (don't use axios or fetch)
    "ky": "^0.12",
    "ky-universal": "^0.3",

    // React
    "react": "^16",
    "react-dom": "^16",
    "react-helmet": "^5",
    "react-router-dom": "^5",

    // Redux
    "immutability-helper": "^3", // dont' use immer or immutable
    "react-redux": "^7",
    "redux": "^4",
    "redux-thunk": "^2",

    // Code splitting (don't use dynamic imports in components)
    "@loadable/component": "^5",

    // Internationalization
    "react-intl": "^2",

    // Theming
    "react-theme-context": "^2",

    // Relay
    "babel-plugin-relay": "^5",
    "react-relay": "^5",

    // Add "importHelpers": true to tsconfig.json
    "tslib": "^1"
  },

  "devDependencies": {

    // don't eject CRA
    "@craco/craco": "^5",

    // TypeScript fork
    "@jedmao/redux-mock-store": "^2",

    // don't use enzyme
    "@testing-library/react": "^8"
  }
}
```
