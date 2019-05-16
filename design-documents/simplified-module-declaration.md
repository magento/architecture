# Simplified Module Declaration

## Problem

To create a Magento module you need to create 3 files:

* `composer.json` - to declare composer package name (`'magento/module-catalog'`), description, version, dependencies, etc
* `module.xml` - to declare Magento module name (`'Magento_Catalog'`) and configuration loading dependencies
* `registration.php` - to let Magento know where the module resides, so Magento can load its configuration files

This implementation has the following issues:

* more things to learn and remember (`module.xml`, `registration.php`, and how are they different from `registration.php`)
* more boilerplate code
* module sorting mechanism is slow
* need to maintain infrastructure to read/parse/process `module.xml`
* need to know about ComponentRegistrar is and different types of components (ComponentRegistrar::MODULE, ComponentRegistrar::THEME, ...), even though you already defined the type of your module in `composer.json`
* `registration.php` of _every_ module/theme/translation is included by composer autoloader on every request.

## Solution

* Eliminate `module.xml`.
  * move "Magento name" (`Magento_Catalog`) to "extra" section of `composer.json`. Make it optional with default composer_name => magento_name mapping mechanism.
  * move "sequence" section to "extra" section of `composer.json`
* Eliminate `registration.php`
  * create and register a composer hook that after `composer install` & `composer update` will:
   * find all installed packages with `"type": "magento2-\*"`  in `composer.json`
   * find all packages by pattern `app/code/*/*`, `app/design/*/*/*`, and `app/i18n/*/*`  with `"type": "magento2-\*"` in their `composer.json` files
   * sort all packages using topological sorting of dependencies
   * generate `magento_components.php` that will register the sorted list of components in `ComponentRegistrar`
   * remove magento built-in module sorting mechanism. Rely on component list being sorted in composer hook

> NOTE: This change DOES NOT eliminate `app/code`. The changes described above should preserve the ability to load modules from `app/code`

## Open Questions

As described by Vinai Kopp, performance of composer dependency resolution impacts developer experience. 

`registration.php` file allows to quickly install a module without composer. Moving to Composer will remove this optimization. 

The solution to this problem should be provided before `registration.php` can be removed. Possible options:

* make `registration.php` optional instead of getting rid of it
* add a command to Composer that will to re-read `composer.json` files of Magento modules without full dependency resolution

## POC

POC that partially implements the proposal is available in https://github.com/magento-architects/magento2/tree/split-framework (see \Magento\Framework\Component\ComponentInstaller::install)
