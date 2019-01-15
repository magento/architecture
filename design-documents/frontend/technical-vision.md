# Front-End Technical Vision

The following document is a guide for all _future_ Magento front-end applications outside of core, for which we should be mindful of the following values:

- Portability
- Extensibility
- Maintainability
- Consistency
- Performance
- Accessibility
- Internationalization
- Productivity (Efficiency)
- Quality (Reliability)

## [React](https://reactjs.org/)

_Portability, Performance, Productivity, Quality, Internationalization._

React's [continued growth and popularity](https://www.npmtrends.com/@angular/core-vs-react-vs-vue) coupled with its [solid documentation](https://reactjs.org/docs/getting-started.html) make it an excellent choice for building user interfaces. If written appropriately, it enables **portability** by [crossing over into native applications](http://jkaufman.io/react-web-native-codesharing/) with [React Native](https://facebook.github.io/react-native/).

**Performance** is easy to achieve with React, because it only re-renders DOM elements where and when necessary. That's not to say that React prevents you from creating **performance** problems, but the docs do well at explaining [what to avoid and how to profile](https://reactjs.org/docs/optimizing-performance.html).

Creating a new React project has never been easier with [CRA][], which comes out-of-the-box with [hot reloading](https://webpack.js.org/concepts/hot-module-replacement/). This increases **productivity** by giving developers [short, positive feedback loops](https://www.youtube.com/watch?v=avKBrgIQXzk). Hot reloading is [also achievable in React Native](https://facebook.github.io/react-native/blog/2016/03/24/introducing-hot-reloading.html). Another opportunity for short, positive feedback loops is testing with [Jest][] and [Enzyme](https://airbnb.io/enzyme/). Tests are fastest when writing unit tests with the [Shallow Rendering API](https://airbnb.io/enzyme/docs/api/shallow.html) and reserving the [Full Rendering API](https://airbnb.io/enzyme/docs/api/mount.html) for integration tests. The better the development experience in these cycles, the better **quality** we achieve as a result. [Jest's snapshot testing](https://jestjs.io/docs/en/snapshot-testing) is another way to ensure **quality** in that we don't inadvertently introduce breaking changes.

### Technologies
- Create a new React project with [CRA][].
- [Redux](https://redux.js.org/) as a predictable state container.
   - Handling of data fetching and side effects now have first class support with [React Suspense, React.lazy](https://reactjs.org/blog/2018/10/23/react-v-16-6.html#reactlazy-code-splitting-with-suspense) and [hooks](https://reactjs.org/docs/hooks-intro.html).
   - Logging with [redux-logger](https://github.com/evgenyrodionov/redux-logger).
- **Internationalization** with [react-intl](https://www.npmjs.com/package/react-intl).
   - _Warning: the documented path here is to couple the components to this library. This will be fine for most projects, but if ultimate **portability** is required, you will need to decouple the component from the [i18n](https://en.wikipedia.org/wiki/Internationalization_and_localization) library._

## [TypeScript][]

_Maintainability, Productivity, Quality, Consistency._

[TypeScript][] is not going to solve all of our problems, but its static typing features definitely have the potential to improve the **quality** of the code we produce. [TypeScript][] code is also easier to **maintain**, because it purposefully makes it more difficult to inadvertently break something that was already defined to work a certain way. As a superset of JavaScript, it allows us to leverage both the [JavaScript ecosystem](https://www.npmjs.com/) as well as any existing developer knowledge. It also happens to [work quite well with React and JSX syntax](https://www.typescriptlang.org/docs/handbook/jsx.html), replacing [prop-types](https://www.npmjs.com/package/prop-types) (run-time) with [TypeScript Interfaces](https://www.typescriptlang.org/docs/handbook/interfaces.html) (compile-time).

> Types and interfaces are only a compile-time construct and have no effect on the generated code.

Catching errors at compile time also increases **productivity**, because we don't have to run the development server to see the errors. In fact, we can see them directly in the editor before anything is run. Additionally, Microsoft has open sourced the [TypeScript Language Service API](https://github.com/Microsoft/TypeScript/wiki/Using-the-Language-Service-API), which is used across the most popular editors of today. As a result, you'll see **consistent** errors between these editors.

[Type definitions](https://www.typescriptlang.org/docs/handbook/declaration-files/consumption.html) are provided for [many of the most popular npm packages](http://definitelytyped.org/), meaning developers can increase **productivity** by staying in-editor instead of constantly task switching and looking up documentation online. Even internally, there is less need to open a local file when you can extract most of the relevant information you need via [Intellisense](https://code.visualstudio.com/docs/editor/intellisense#_intellisense-features).

_Disclaimer: though possible to introduce [TypeScript][] into an existing [CRA][] app and though [TypeScript][] can co-exist with JavaScript, it may not make sense to introduce it into an existing project if you feel it may hinder momentum._

### Technologies
- [Adding TypeScript to CRA](https://facebook.github.io/create-react-app/docs/adding-typescript).
- Code **consistency** is enforced via [configuration](http://www.typescriptlang.org/docs/handbook/tsconfig-json.html), [linting with TSLint](https://palantir.github.io/tslint/) and formatting with [Prettier](https://prettier.io/).
   - Use the most strict tsconfig rules before you write a single line of code, because it's much harder to introduce these rules later in a project.
   - Use [`tslint-config-prettier`](https://www.npmjs.com/package/tslint-config-prettier) to ensure there is no overlap between TSLint and Prettier.
   - Extend `tslint-recommended` as a starting point. Introduce new rules with extreme hesitation to prevent bikeshedding over trivial and subjective preferences.
   - With [`husky`](https://www.npmjs.com/package/husky), ensure errors are caught before CI with a [`pretty-quick`](https://www.npmjs.com/package/pretty-quick) pre-commit hook and a pre-push hook for linting.
- Generate [TypeScript][] declarations for [CSS Modules](https://github.com/css-modules/css-modules) via [css-modules-typescript-loader](https://www.npmjs.com/package/css-modules-typescript-loader).
- Documentation via [TSDoc](https://github.com/Microsoft/tsdoc#tsdoc).

## Portability

We should prepare to achieve as much code sharing as possible, regardless of environment. JavaScript is the only language that can run natively in all browsers; yet, it also [transcends the browser](https://cdb.reacttraining.com/universal-javascript-4761051b7ae9), running on servers and [even native mobile applications](https://facebook.github.io/react-native/). As such, the golden path for sharing code starts with JavaScript. By writing components in a decoupled way, we don't limit ourselves as to where those components can run.

On top of sharing code between different environments, we should also be concerned about sharing code between projects. For this, we should introduce a business logic repository that includes common helper functions – logic that we're likely to use between projects.

### Business Logic Repository
1. Written and [documented](https://github.com/Microsoft/tsdoc#tsdoc) in [TypeScript][].
1. [Semantic versioning](https://semver.org/) to ensure that progress can be made without breaking things for existing - consumers.
1. 100% unit test coverage via [Jest][] for **consistency**.

For **performance** reasons, JavaScript should only be used for cheap operations on the front end, reserving expensive operations for a services layer. It's worth calling out that we could achieve even more code sharing if this API were written in JavaScript. For practical reasons, however, it makes more sense to leverage internal PHP resources for this task.

## Browser Support
All users should have a **consistent** user experience. For the best **performance**, those with the latest browsers should not be penalized by incurring an additional payload hit than those with older browsers.

There are [various ways](https://polyfill.io) to achieve this goal; however, an in-house approach might be warranted to prevent the stability issues of relying on 3rd party resources. One idea would be to publish multiple build targets, tailored to various environments.

## React UI Component Library

_Consistency, Productivity, Scalability, Maintainability, Accessibility._

We should lean against downloading various UI components from different sources for the following reasons:

- **Inconsistent** implementations, documentation, theming and customization.
- Limited **accessibility** and support.

If you have the budget to pay for [a product](https://www.telerik.com/kendo-react-ui/), it might be a good choice in this space. Why? Because it's much better to get all of your components from a single source than to have completely different paradigms scattered everywhere.

_Disclaimer: there may be licensing limitations that prevent an open-source product from implementing a paid-for product._

### Downloading a 3rd-party package

There are times when you may consider downloading a 3rd party package (e.g., a component). When this happens, consider the following criteria before making a final choice:

1. First thing's first – what is the license?
   - Is it open source?
   - You may need to reach out to your legal department if things are a bit blurry.
1. Does the package have a bright future?
   - Popularity – are there a significant number of weekly downloads?
   - Is the downloads line graph going up or down?
1. Look for tests, ideally with 100% test coverage.
1. Look at the bundle size with [Bundle Phobia](https://bundlephobia.com/).
1. Does it have good/clear documentation?
1. Is it currently maintained?
   - Look at the number of issues vs. pull requests.
   - How long have these issues and PRs been open?
   - When was the last comment made on them and by whom?
   - **Reliability** – how serious are the issues?
1. Search and compare this package with other similar packages to gauge which package is the best choice and feel free to reach out to other team members for guidance.

### Resources
- [spectrum-css](http://opensource.adobe.com/spectrum-css/2.6.0/docs/)

## Styles

Bundle optimization works best when CSS Modules are imported directly into the JavaScript components that use them (a better [TypeScript][] experience is being talked about in [this CRA issue](https://github.com/facebook/create-react-app/issues/5677)).

The coupling here is a bit unfortunate for some projects (e.g., libraries), but pretty standard for applications. For ultimate **portability**, reusable components should be provided their styles externally.

### Resources

- [Adding CSS Modules to CRA](https://facebook.github.io/create-react-app/docs/adding-a-css-modules-stylesheet)
- [css-modules-typescript-loader](https://www.npmjs.com/package/css-modules-typescript-loader)
- [CSS Modules VSCode extension](https://marketplace.visualstudio.com/items?itemName=clinyong.vscode-css-modules)

## Code Quality

_Quality, Maintainability, Productivity._

We should leverage the built-in code coverage reports that [Jest][] provides, but also the following GitHub integrations/apps:

- Continuous integration with [Travis-CI](https://travis-ci.com/).
- [CodeClimate](https://codeclimate.com/)
  - [**Quality** by Code Climate](https://codeclimate.com/quality/) provides automated code review for test coverage, **maintainability** and more so that you can **save time** and merge with confidence.
  - Code Climate’s engineering process insights and automated code review for GitHub and GitHub Enterprise help you ship **better software, faster**.

### Resources
- [Code Climate for TypeScript is here!](https://codeclimate.com/changelog/5a147fcac7d08102b700084e/)
- [Code Climate | TSLint](https://docs.codeclimate.com/docs/tslint)

## Performance Testing

Ideally, for each front-end project, there will be a separate project that covers both performance and critical-path testing. This project is separate so that it doesn't cripple the short feedback loops of feature development. Additionally, we would do well to have additional performance testing around the most commonly used components (e.g., used in 3+ places).

[CRA]: https://facebook.github.io/create-react-app/
[Jest]: https://jestjs.io/
[Typescript]: http://www.typescriptlang.org/
