# PHPCompatibility in Code Style Ruleset

## Goal

Utilize [PHPCompatibility](https://github.com/PHPCompatibility/PHPCompatibility) tool in static test builds, so that rules specific to PHP version are validated.

Requirements include determining PHP version from `composer.json` of the Magento project or Magento module, so validation against correct version happens automatically.
This requirement though introduces additional complexity to the ruleset/sniff implementation:
- `composer.json` may be absent
- it requires custom bootstrapping logic for `phpcs`
- it's not clear whether we should analyze different files differently in case they are from different modules and those modules have different PHP constraints

Based on the above it is proposed to put the responsibility of specifying PHP version to the user (owner of the project or CI).

## PHPCompatibility Tool

PHPCompatibility provides sniffs for [phpcs](https://github.com/squizlabs/PHP_CodeSniffer).
It allows to specify PHP version to run against, as well as to run it without any specific version.

* If PHP version is specified, rules that are not applicable to it will not be reported as issues. For example, if a constant has been removed in PHP 7.2, and `phpcs` is run with `--runtime-set testVersion 7.0`, no issue will be reported if the constant is used in the validated code.
* If PHP version is not specified, the validation will happen as if the latest PHP is used (supported by PHPCompatibility rules).

## Design 

Add [PHPCompatibility](https://github.com/PHPCompatibility/PHPCompatibility) to Magento repository and use its ruleset in the CI builds.
Specialized static test is implemented, which determines necessary PHP version (or range of versions) based on `composer.json` of the Magento Git project and passes it as an argument to PHPCS.

Developers can include use included PHPCompatibility as described in the PHPCompatibility documentation if they want.

### Note on Including PHPCompatibility in Magento Coding Standard

[Magento Coding Standard](https://github.com/magento/magento-coding-standard) contains `phpcs` rules applicable to any Magento code.
Due to default behavior of PHPCompatibility, which reports all issues as if the latest PHP version is used, it can't be included in the Magento Coding Standard because it will fail Magento code which is not supposed to support latest PHP versions.
That's why for better dev experience and to provide choice to developers, PHPCompatibility should be added on top of other standards, and can't be included in them.
