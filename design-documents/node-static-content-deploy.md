# Modernize Static Content Deployment

**Note: This is a proposal for a possible improvement in the future. Even if we reach consensus, there are no guarantees yet that this would be implemented. Exploratory for the moment**

- [Static Content Deployment](#static-content-deployment)
  * [Docs](#docs)
  * [Use Cases](#use-cases)
  * [What does it do?](#what-does-it-do-)
  * [Problems with Static Content Deployment](#problems-with-static-content-deployment)
  * [Why rewrite Static Content Deployment?](#why-rewrite-static-content-deployment-)
    + [Performance](#performance)
    + [Access to Modern Tooling](#access-to-modern-tooling)
    + [Accessible for Front-End Devs](#accessible-for-front-end-devs)
    + [What opportunities open up with a re-write of SCD using node.js?](#what-opportunities-open-up-with-a-re-write-of-scd-using-nodejs-)
  * [How do you re-write Static Content Deployment?](#how-do-you-re-write-static-content-deployment-)
  
## Docs
- [Deploying Static View Files](https://devdocs.magento.com/guides/v2.3/config-guide/cli/config-cli-subcommands-static-view.html)
- [Strategies](https://devdocs.magento.com/guides/v2.3/config-guide/cli/config-cli-subcommands-static-deploy-strategies.html)

## Use Cases
1. Pre-processing and flattening of static assets in local development (i.e. less)
1. Pre-processing and flattening of static assets for production deployment

## What does it do?

- [Theme/module/i18n/"lib/web" file inheritance flattening](https://devdocs.magento.com/guides/v2.3/frontend-dev-guide/themes/theme-inherit.html)
- Locale splitting
    - Each locale gets its own static assets under a theme dir
- Less compilation
- HTML minification
- JS minification
- JS concatenation
- `js-translation.json` generation
- `requirejs-config.js` generation
- Asset signing/versioning

## Problems with Static Content Deployment

There are a number of issues with the current implementation of static content deployment.

1. It's prohibitively slow, sometimes taking 10+ minutes just to copy and process files
1. There are 3 separate deployment strategies, all with different trade-offs that are not ideal. Having 3 separate ways to accomplish one task had led to [an explosion of open SCD bug reports (600+)](https://github.com/magento/magento2/search?q=static+content+deploy+is%3Aissue&type=Issues), where every strategy does not quite work as expected.
    - **Quick**: Duplicates a ton of files, which makes cloud deployments slower
    - **Standard**: Does way more work than it needs to, and duplicates tons of files
    - **Compact**: Basically re-implements symlinks in the application layer, ships 150kb+ of extra JS to the front-end
1. When something fails, it's very difficult to figure out why
1. Lack of reliable cache + incremental builds (most people typically run it from scratch)
1. Unable to introduce modern FE tooling (because it all expects a `node.js` runtime)
1. Lacking granular reporting (unclear which parts of the process cause spikes in deployment times)

## Why rewrite Static Content Deployment?

There are 3 primary motivators for re-writing the Static Content Deployment tool

### Performance

The performance of `bin/magento setup:static-content:deploy` is _not great_, which leads to developers being unhappy about doing front-end work.

It is also a very significant bottleneck for Magento Cloud deployments.

### Access to Modern Tooling
The majority of modern front-end tooling is written in JavaScript and executed using `node.js`. By moving our front-end compilation tool to JavaScript, we're able to leverage a large ecosystem of open-source tooling that cannot easily be interfaced with from PHP.

A great example of this is our usage of LESS. The only [official compiler for the LESS language](http://lesscss.org/usage/) is written in JavaScript. Because of this, Magento uses a 3rd party (incomplete) implementation of the LESS compiler in PHP. However, because of the slowness associated with it, most developers prefer to use Magento's grunt setup _during local development_. The differences between these 2 compilers leads to [very unfortunate bugs](https://github.com/magento/magento2/issues/7231) when a production deployment is done.

### Accessible for Front-End Devs
When building something new, it's important to choose the right programming language for the job.

Often, the "right" language is based on the skillset of those that will be maintaining the tool. As an example, the Magento CLI is written using PHP. PHP is not a language designed with the intent of being used to write CLIs, but it is capable of it. Magento chose to write `bin/magento` in PHP because it makes contribution accessible to those using the tool.

A re-write in JavaScript means that the code for static content deployment can be maintained by front-end developers, who spend most of their time in Magento working on JavaScript/HTML/CSS.

### What opportunities open up with a re-write of SCD using node.js?
- Compilation of modern JS with [Babel](https://babeljs.io/)
- Automatic live reloading of CSS during development
- [Autoprefixer](https://github.com/postcss/autoprefixer) support for CSS
- Linting/Unit-testing integrated in local dev and build pipelines
- Automatic generation of a service worker via [Workbox](https://developers.google.com/web/tools/workbox/) (oh look, PWA functionality)
- Incremental file watcher (build minimal changesets for fast dev mode)

## How do you re-write Static Content Deployment?

**INCREMENTALLY!** Seriously. This is an ideal piece of Magento to re-write piecemeal. The existing static content deploy command can stay intact through-out the entirety of a re-write. Deprecation and removal would only need to happen when feature parity is reached. While it's under active development, people will be able to switch to it as soon as it has the features they need. This makes a re-write extremely low-risk.

A high-level set of steps:

1. Write code to scrape necessary configurations from Magento (mainly `config.php` to determine what modules to work with)
1. Build a quick prototype that proves that file inheritance can effectively be re-implemented with _significantly_ better performance outside of Magento core, leveraging both symlinks and copy-on-write where available
1. Create a base architecture for a CLI that can be made up of various tasks (minify, less, etc etc)
1. Create a backlog of critical tasks needed for a fully functional Static Content Deployment tool, and open up for community contribution
1. Iterate, iterate, iterate

## Prior Art
- https://github.com/SnowdogApps/magento2-frontools/issues/25 Static Asset Processing (used by many Magento FE developers and agencies)