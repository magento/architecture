# PHPCompatibility in Code Style Ruleset

## Goal

Utilize PHPCompatibility tool in `phpcs` rules, so that rules specific to PHP version are validated.

Requirements include determining PHP version from `composer.json` of the Magento project or Magento module, so validation against correct version happens automatically.
This requirement though introduces additional complexity to the ruleset/sniff implementation:
- `composer.json` may be absent
- it requires custom bootstrapping logic for `phpcs`
- it's not clear whether we should analyze different files differently in case they are from different modules and those modules have different PHP constraints

Based on the above it is proposed to put the responsibility of specifying PHP version to the user (owner of the project).

## Implementation Details

In https://github.com/magento/magento-coding-standard repository:
1. Add `phpcompatibility/php-compatibility` to Composer dependencies
2. Add necessary configuration to the ruleset, so that PHPCompatibility ruleset is included
   1. Add `<config name="installed_paths" value="./../../../../../vendor/phpcompatibility/php-compatibility" />`
   2. Add `<rule ref="PHPCompatibility"/>`

There are two options for https://github.com/magento/magento2 repository:
1. Add `<config name="testVersion" value="7.1.3-7.3.0"/>` to Magento static tests ruleset
   1. Different branches will have different PHP versions
   2. Drawback: PHP versions in the ruleset should be updated every time Magento changes supported PHP versions
2. Determine PHP version in `\Magento\Test\Php\LiveCodeTest::testCodeStyle()` based on the project's `composer.json` and pass it as an argument to `phpcs`
   1. Drawback: IDEs using Magento ruleset won't take into account any PHP version. This should not be a big issue as usually IDEs have settings for runtime version and highlight incompatibilities with selected version. If this is not enough, a project owner can add `<config name="testVersion" value="their-php-version"/>` in their ruleset
