# Overview

Magento actively leverages various testing strategies to ensure product and code quality. 

Product quality matters to the end-users working with the Magento storefront, admin panel, web APIs and CLI. 
Product quality tests verify that the user gets expected results while interacting with the system's interface.

Code quality is important to developers working with Magento codebase including system integrators, extension developers, Magento core team and community contributors.
Code quality matters for:
 * Extensibility - how easy is it to extend or modify existing behavior.
   Good extensibility allows for
    * customizations and extensions using modularity of the platform
    * evolution of the platform with new releases
 * Maintainability - how easy is it to work with the system for developers. 
   It can be improved with
    * complexity management - reduces learning curve and risk of new bugs
    * automatic bug detection - reduces total cost of ownership
    * coding standards - reduces learning curve
    
Automated tests are required by [Magento definition of done](https://devdocs.magento.com/guides/v2.3/contributor-guide/contributing_dod.html) for any code changes. 
Most of the tests mentioned below are executed as part of the automated delivery process for each pull request before one is merged to the mainilne branch.

## Product Quality Tests

 * [Functional](https://devdocs.magento.com/guides/v2.3/mtf/mtf_introduction.html) - for storefront and admin panel UI
 * [Web API Functional](https://devdocs.magento.com/guides/v2.3/get-started/web-api-functional-testing.html) - for REST, SOAP, GraphQL
 * [Integration](https://devdocs.magento.com/guides/v2.3/test/integration/integration_test_execution.html) - for quick testing of low-level steps which make business sense on their own 
 * Performance - track changes in CPU, Memory and other metrics for the predefined list of scenarios. Executed nightly for develop branches. Custom builds can be configured using [performance toolkit](https://devdocs.magento.com/guides/v2.3/config-guide/cli/config-cli-subcommands-perf-data.html)
 * Client-side performance - measure total page load times in the browser. These tests will be outsourced in the future
 * Load - track the trends of system behavior under the load. Executed internally for each release using scenarios defined in the [performance toolkit](https://devdocs.magento.com/guides/v2.3/config-guide/cli/config-cli-subcommands-perf-data.html)
 * Upgrade - ensure that seamless upgrade is possible from previously released product versions to the new one. Executed internally for each release
 * [JavaScript](https://devdocs.magento.com/guides/v2.3/test/js/test_js-unit.html) - for JS modules and other JS portions of the UI. These tests are similar to integration tests used for server-side testing

## Code Quality Tests
 * Static
    * [PhpCs](https://devdocs.magento.com/guides/v2.3/coding-standards/code-standard-sniffers.html) - warns developers about coding standard violations. Can be integrated with IDE to analyze the code as soon as you write it 
    * [PhpMd](https://phpmd.org/) - mostly used for code complexity and potential bugs monitoring, also integrates with IDE for instant feedback
    * Dependency - prevents incorrect dependencies between modules
    * Legacy code - prevents usage of the legacy functionality
 * Unit - mostly suitable for testing of isolated algorithms
 * Extensibility - make sure that system is extensible as designed. Extensibility tests can be written using [Integration](https://devdocs.magento.com/guides/v2.3/test/integration/integration_test_execution.html), [JavaScript](https://devdocs.magento.com/guides/v2.3/test/js/test_js-unit.html) and [Web API Functional](https://devdocs.magento.com/guides/v2.3/get-started/web-api-functional-testing.html) testing frameworks.
 * Backward-compatibility - enforces [Magento backward compatibility policy](https://devdocs.magento.com/guides/v2.3/contributor-guide/backward-compatible-development/) at all levels including source code, DB, message queue, web API 

