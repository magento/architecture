# Overview
PHP CodeSniffer is a static code analysis tool that tokenizes PHP, JavaScript and CSS files to detect violations of a defined coding standard.

A coding standard in PHP CodeSniffer is a collection of sniff files combined in the `ruleset.xml` file. Each sniff checks one part of the coding standard.

Each sniff can have its own `severity` (priority) and `type` (two options available: `error` or `warning`).

## Problem Overview
There are few Magento 2 related coding standards:
- [Magento 2 Core sniffs](https://github.com/magento/magento2/tree/2.3-develop/dev/tests/static/framework/Magento/Sniffs)
- [Magento Marketplace Extension Quality Program](https://github.com/magento/marketplace-eqp) (in short: MEQP)
- [ExtDN PHP CodeSniffer rules](https://github.com/extdn/extdn-phpcs) (in short: Extdn)

Each of these coding standards contain a lot of sniffs in common. However, there are also some differences in the code checking approach. For example:

- The MEQP coding standard uses a `severity` approach (from 10 to 1). The test fails only in the case of a `severity` 10 finding (`error`). Issues with `severity` from 1 to 9 marked as a `warning` and do not cause test failure.

- In the Magento 2 Core checking strategy, all findings lead to test failure. **Note:** _By default, PHP CodeSniffer assigns a severity of 5 to all errors and warnings._

- MEQP coding standard contains some false-positive sniffs, for which the aim is just to warn developers and draw their attention to the particular code piece.

- Some of Magento Core sniffs require Magento 2 instance to make a kind of `dynamic` check.

As a result it leads to inconsistency and [code duplication](https://github.com/magento/magento2/issues/19477). Extension developers are confused about what code standard to use in their extension development and Magento 2 related projects contribution such as [MFTF](https://github.com/magento/magento2-functional-testing-framework/issues/265).

## Goals
1. Make it easier to run static checks for extensions developers.
2. Store all related to Magento 2 sniffs in one place.
3. Make static check consistent.

## Proposal
1. Consolidate all related Magento 2 sniffs to the new GitHub repository.

2. Create a new Composer package with  `phpcodesniffer-standard` type and add it to the Magento 2 `composer.json` `reqiure-dev` section.

3. Assign severity to each sniff. The higher the severity, the more strict the rule is.

4. Create one `ruleset.xml` file (set of sniffs) for both projects - Magento Core and Magento Extension Quality Program.

5. For sniffs that require Magento 2 instance use [bootstrap file](https://github.com/squizlabs/PHP_CodeSniffer/wiki/Advanced-Usage#using-a-bootstrap-file).

6. Revisit other Magento 2 static tests (`PHPMD` rules and covered by `PHPUnit` static tests) and if it's possible rewrite them to use PHP CodeSniffer.

7. Work with community on improvements to sniffs and make regular releases.
