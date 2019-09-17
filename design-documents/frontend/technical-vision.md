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

## Table of Contents

- [React](#react)
- [TypeScript](#typescript)
- [Portability](#portability)
- [Browser Support](#browser-support)
- [React UI Component Library](#react-ui-component-library)
- [Styles](#styles)
- [Accessibility](#accessibility)
- [Code Quality](#code-quality)
- [Testing](#testing)
- [Scaffolding](#scaffolding)

## [React][]

_Portability, Performance, Productivity, Quality, Internationalization._

[React][]'s [continued growth and popularity](https://www.npmtrends.com/@angular/core-vs-react-vs-vue) coupled with its [solid documentation](https://reactjs.org/docs/getting-started.html) make it an excellent choice for building user interfaces. If written appropriately, it enables **portability** by [crossing over into native applications](http://jkaufman.io/react-web-native-codesharing/) with [React Native](https://facebook.github.io/react-native/).

**Performance** is easy to achieve with [React][], because it only re-renders DOM elements where and when necessary. That's not to say that [React][] prevents you from creating **performance** problems, but the docs do well at explaining [what to avoid and how to profile](https://reactjs.org/docs/optimizing-performance.html).

Creating a new [React][] project has never been easier with [CRA][], which comes out-of-the-box with [hot reloading](https://webpack.js.org/concepts/hot-module-replacement/). This increases **productivity** by giving developers [short, positive feedback loops](https://www.youtube.com/watch?v=avKBrgIQXzk). Hot reloading is [also achievable in React Native](https://facebook.github.io/react-native/blog/2016/03/24/introducing-hot-reloading.html).

[CRA][] also comes with [Jest][], a delightful JavaScript Testing Framework with a focus on simplicity. [Jest][] comes with [snapshot testing](https://jestjs.io/docs/en/snapshot-testing), another way to ensure **quality** in that we don't inadvertently introduce breaking changes. Add [react-testing-library](https://www.npmjs.com/package/react-testing-library) to encourage good testing practices. The better the development experience in these cycles, the better **quality** we achieve as a result.

### React Technologies

- Create a new [React][] project with [CRA][].
- [Redux](https://redux.js.org/) as a predictable state container.
  - Handling of data fetching and side effects now have first class support with [React Suspense, React.lazy](https://reactjs.org/blog/2018/10/23/react-v-16-6.html#reactlazy-code-splitting-with-suspense) and [hooks](https://reactjs.org/blog/2019/02/06/react-v16.8.0.html).
  - Logging with [redux-logger](https://github.com/evgenyrodionov/redux-logger).
  - [Redux Starter Kit](https://redux-starter-kit.js.org/).
- [Loadable components](https://www.smooth-code.com/open-source/loadable-components/) for code splitting.
- **Internationalization** with [react-intl](https://www.npmjs.com/package/react-intl).
  - _Warning: the documented path here is to couple the components to this library. This will be fine for most projects, but if ultimate **portability** is required, you will need to decouple the component from the [i18n](https://en.wikipedia.org/wiki/Internationalization_and_localization) library._

## [TypeScript][]

_Maintainability, Productivity, Quality, Consistency._

[TypeScript][] is not going to solve all of our problems, but its static typing features definitely have the potential to improve the **quality** of the code we produce. [TypeScript][] code is also easier to **maintain**, because it purposefully makes it more difficult to inadvertently break something that was already defined to work a certain way. As a superset of JavaScript, it allows us to leverage both the [JavaScript ecosystem](https://www.npmjs.com/) as well as any existing developer knowledge. It also happens to [work quite well with React and JSX syntax](https://www.typescriptlang.org/docs/handbook/jsx.html), replacing [prop-types](https://www.npmjs.com/package/prop-types) (run-time) with [TypeScript Interfaces](https://www.typescriptlang.org/docs/handbook/interfaces.html) (compile-time).

> Types and interfaces are only a compile-time construct and have no effect on the generated code.

Catching errors at compile time also increases **productivity**, because we don't have to run the development server to see the errors. In fact, we can see them directly in the editor before anything is run. Additionally, Microsoft has open sourced the [TypeScript Language Service API](https://github.com/Microsoft/TypeScript/wiki/Using-the-Language-Service-API), which is used across the most popular editors of today. As a result, you'll see **consistent** errors between these editors.

[Type definitions](https://www.typescriptlang.org/docs/handbook/declaration-files/consumption.html) are provided for [many of the most popular npm packages](http://definitelytyped.org/), meaning developers can increase **productivity** by staying in-editor instead of constantly task switching and looking up documentation online. Even internally, there is less need to open a local file when you can extract most of the relevant information you need via [Intellisense](https://code.visualstudio.com/docs/editor/intellisense#_intellisense-features).

_Disclaimer: though possible to introduce [TypeScript][] into an existing [CRA][] app and though [TypeScript][] can co-exist with JavaScript, it may not make sense to introduce it into an existing project if you feel it may hinder momentum._

### TypeScript Technologies

- [Adding TypeScript to CRA](https://facebook.github.io/create-react-app/docs/adding-typescript).
- Code **consistency** is enforced via [configuration](http://www.typescriptlang.org/docs/handbook/tsconfig-json.html), linting with [TypeScript ESLint][] and formatting with [Prettier][].
  - Use the most strict [compiler options](https://www.typescriptlang.org/docs/handbook/compiler-options.html) before you write a single line of code, because it's much harder to introduce these rules later in a project.
  - Use [`eslint-config-prettier`](https://github.com/prettier/eslint-config-prettier) and extend `prettier/@typescript-eslint` to ensure there is no overlap between [TypeScript ESLint][] and [Prettier][]. See [installation instructions](https://github.com/prettier/eslint-config-prettier#installation).
  - Extend `eslint:recommended` as a starting point. Introduce new rules with extreme hesitation to prevent bikeshedding over trivial and subjective preferences.
  - With [`husky`](https://www.npmjs.com/package/husky), ensure errors are caught before CI with a [`lint-staged`](https://www.npmjs.com/package/lint-staged) pre-commit hook and a pre-push hook for checking types.
- Documentation via [TSDoc](https://github.com/Microsoft/tsdoc#tsdoc).

## Portability

We should prepare to achieve as much code sharing as possible, regardless of environment. JavaScript is the only language that can run natively in all browsers; yet, it also [transcends the browser](https://cdb.reacttraining.com/universal-javascript-4761051b7ae9), running on servers and [even native mobile applications](https://facebook.github.io/react-native/). As such, the golden path for sharing code starts with JavaScript. By writing components in a decoupled way, we don't limit ourselves as to where those components can run.

On top of sharing code between different environments, we should also be concerned about sharing code between projects. For this, we should introduce a business logic repository that includes common helper functions – logic that we're likely to use between projects.

### Business Logic Repository

1. Written and [documented](https://github.com/Microsoft/tsdoc#tsdoc) in [TypeScript][].
1. [Semantic versioning](https://semver.org/) to ensure that progress can be made without breaking things for existing - consumers.
1. 100% unit test coverage via [Jest][] for **consistency**. Whether or not you enforce 100% coverage in the rest of your project, there is additional value in thoroughly testing business logic.

For **performance** reasons, JavaScript should only be used for cheap operations on the front end, reserving expensive operations for a services layer. It's worth calling out that we could achieve even more code sharing if this API were written in JavaScript. For practical reasons, however, it makes more sense to leverage internal PHP resources for this task.

## Browser Support

All users should have a **consistent** user experience. For the best **performance**, those with the latest browsers should not be penalized by incurring an additional payload hit than those with older browsers.

In a CRA, [react-app-pollyfill](https://github.com/facebook/create-react-app/tree/master/packages/react-app-polyfill#react-app-polyfill) is the supported path.

## [React][] UI Component Library

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

- [spectrum-css](http://opensource.adobe.com/spectrum-css/)

## Styles

Bundle optimization works best when CSS Modules are imported directly into the JavaScript components that use them. The coupling here is a bit unfortunate for some projects (e.g., libraries), but pretty standard for applications. For ultimate **portability**, reusable components should be provided their styles externally.

### Linaria

If you are interested in a CSS in JS solution, look no further than [Linaria](https://linaria.now.sh/), offering a zero-runtime CSS in JS library. In a CRA, you'll need [craco](https://www.npmjs.com/package/@craco/craco) to [install it without ejecting](https://github.com/adobe/generator-tsx/pull/16/files).

### Style Resources

- [Adding CSS Modules to CRA](https://facebook.github.io/create-react-app/docs/adding-a-css-modules-stylesheet)
- [CSS Modules VSCode extension](https://marketplace.visualstudio.com/items?itemName=clinyong.vscode-css-modules)

## Accessibility

Ideally, **accessibility** will have been thoroughly considered during the Design/UX stage of a project, but this is not always realistic and many people consider it an afterthought.

To prevent having to completely refactor things when **accessibility** becomes a priority, there are some simple steps you can take:

1. Start scaffolding out a component without writing any JavaScript or CSS. As a result, you end up getting a lot of accessibility features for free. For example, the color and size selections on an e-commerce product page might be expressed as simple radio buttons. All of the keyboard and <kbd>tab</kbd> navigation comes for free! Deviating from built-in behavior would be an anti-pattern and unexpected by the user.
1. Build upon step 1 with JavaScript behaviors, taking care to preserve the HTML you have already created. Remember that most of the behavior you need is already provided by the HTML forms &ndash; less is more! At this stage, you should be mindful of what classes need to be added to certain elements in preparation for a style pass.
1. Styles &ndash; do your best to _not_ touch the HTML or JavaScript at this stage. Use your CSS wizardry to achieve the design specifications provided to you. Do everyone a favor and provide screenshots of the final outcome in your pull request so they know what they are reviewing.

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

## Testing

There are many types of tests you can have for an application and many variables that ultimately determine how much value you give to each one. [The Testing Trophy](https://testingjavascript.com/) provides a good visualization for both the importance/size and the placement of each type.

[<img src="https://res.cloudinary.com/dg3gyk0gu/image/upload/v1539186394/theTestingTrophy_2x.png" alt="The Testing Trophy" height="373">](https://testingjavascript.com/)

### Static Type System with [TypeScript][]

When building an application, like building a trophy, we start at the foundation with a static type system and linter (e.g., [TypeScript][] and [TypeScript ESLint][]). This sets up a good foundation and comes at a relatively low cost for new projects.

### Unit & Integration Tests with [Jest][]

The next layer of this trophy is built with unit tests. With [Jest][], these also come at a relatively low cost, because of how simple it is to write snapshot tests. These are especially useful when testing [React][] components with the speed and efficiency of shallow rendering &ndash; a cheap win!

_Note that [the author](https://kentcdodds.com/) of the testing trophy [has encouraged against shallow rendering](https://kentcdodds.com/blog/why-i-never-use-shallow-rendering). This is good advice in the context of testing state changes directly, but otherwise too black and white. Be mindful to write your shallow tests in such a way that you are testing changes in rendered output rather than individual state._

> Ideally, one should adhere to a [TDD process](https://en.wikipedia.org/wiki/Test-driven_development) when writing unit tests.

[Jest][] also covers the 3rd and most valuable layer of this trophy, which is integration tests. But don't let its value & size overshadow the importance of the layers beneath it. You wouldn't want to build upon a weak foundation.

### End-to-End Testing Frameworks

Lastly, the final touch at the top of this trophy is known by many names: End-to-end (E2E), functional, automated and UI tests. Refer to the following list for options in this territory.

- [Cypress.io](https://www.cypress.io/)
- [Nightwatch.js](http://nightwatchjs.org/)
- [WebdriverIO](https://webdriver.io/)

_Disclaimer: the above list is not exhaustive and is in no particular order._

### Performance Testing

Ideally, for each front-end project, there will be a separate project that covers both performance and critical-path testing. This project is separate so that it doesn't cripple the short feedback loops of feature development. Additionally, we would do well to have additional performance testing around the most commonly used components (e.g., used in 3+ places).

## Scaffolding

[Yeoman][] is a tool for scaffolding modern web applications. In an effort to make the above guide as painless as possible and to encourage best practices, a [TSX Yeoman Generator](https://www.npmjs.com/package/generator-tsx) has been created for you. Refer to its documentation for more information.

[cra]: https://facebook.github.io/create-react-app/
[jest]: https://jestjs.io/
[prettier]: https://prettier.io/
[react]: https://reactjs.org/
[typescript]: http://www.typescriptlang.org/
[typescript eslint]: https://github.com/typescript-eslint/typescript-eslint
[yeoman]: https://yeoman.io/
