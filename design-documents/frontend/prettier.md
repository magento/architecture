# Replace Stylistic Static Checks with Prettier

## Context

Magento 2 has a concept of [static tests](https://devdocs.magento.com/guides/v2.3/test/testing.html). For JavaScript static tests in particular, a large number of tests are lint rules that ensure code follows the formatting guidelines outlined in the [Magento JavaScript Code Standards][].

There are a few reasons that this current setup is not ideal:

1. Only a limited number of auto-fixes are available in ESLint. This means that a developer authoring code in the Magento core has to continually format their file manually anytime they run static tests and see an error
1. JSCS has been [deprecated since mid 2016](https://eslint.org/blog/2016/07/jscs-end-of-life) and is no longer maintained
1. The JavaScript language's grammar keeps expanding annually. Any time we adopt new syntax, we'll end up pausing to bikeshed new rules to keep the codebase consistent

## Prettier

[Prettier][] is an opinionated code formatter for a large variety of languages, written in JavaScript, and running on `node.js`. It has experienced explosive adoption in the front-end ecosystem over the last 2 years. The benefit to Prettier is that it exposes only a minimal set of configuration options:

- Tab Width
- Tabs or Spaces
- Semicolons
- Quotes styles
- Trailing commas
- Bracket Spacing

The following large OSS projects are using [Prettier][] to format their code:

- [React (Facebook)](https://github.com/facebook/react/blob/9d483dcfd6aad53ce082d249845436a56eb39248/.prettierrc.js)
- [Babel (Community)](https://github.com/babel/babel/blob/56044c7851d583d498f919e9546caddf8f80a72f/.prettierrc)
- [webpack (Community)](https://github.com/webpack/webpack/blob/07d4d8560060c102a1aed97844d452547d02b1d4/.prettierrc.js)
- [Gatsby (Community)](https://github.com/gatsbyjs/gatsby/blob/aa4f9397665d6d1e7ea6cdd3bfd6f40b449daccf/.prettierrc)
- [Next.js (Zeit)](https://github.com/zeit/next.js/blob/f4f3649de33830d6758172a8d9df8b9a3e7142e9/package.json#L18)

It's also worth noting that [`pwa-studio`](https://github.com/magento-research/pwa-studio/blob/20ff0d13b4d1cbd0ac8d809cdcae19a5f5f355b5/prettier.config.js) already uses [Prettier][] (along with some internal, non OSS projects), and our current [tech vision](https://github.com/magento/architecture/blob/f45e2a6634f673c05c42e32ee0727fbae5658d8d/design-documents/frontend/technical-vision.md#technologies-1) recommends using it for new projects.

## Proposed Solution

I am proposing the following changes be made:

1. Remove JSCS usage in static tests
1. Remove all stylistic rules from the ESLint configuration (only keep rules that try to prevent common bugs)
1. Use [`Prettier`](https://prettier.io/) for formatting of all files ending with the extensions `.js`, `.html`, `.less`. Use a configuration that matches current Code Standards as much as possible
1. Update [Magento JavaScript Code Standards][] to specify that all stylistic formatting rules are dictated by [Prettier][]
1. Update `static` task in build configuration to run Prettier + ESLint instead of JSCS + ESLint
1. Add note in DevDocs recommendating adding a plugin for [Prettier][] to your editor, to get auto-formatting while working

## Why?

I strongly believe that, when possible, a tool should just do the right thing, instead of instructing you to do it manually. The goal here is to automate tedious manual work that a computer is better at doing.

[Prettier]: https://prettier.io/
[Magento JavaScript Code Standards]: https://devdocs.magento.com/guides/v2.3/coding-standards/code-standard-javascript.html