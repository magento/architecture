# Simplified Module Declaration

## Problem

Magento module metadata is declared in the following files today:

* `module.xml` - declares module name and configuration load order dependencies
* `registration.php` - tells Magento where the module resides for Magento to be able to read the module configuration files
* (optional) `composer.json` - declares package name, description, version, dependencies, etc. It is optional and only becomes required if you want to distribute your module through Composer.

This implementation of Magento module declaration leads to some inconveniences:
* the standard PHP way of declaring packages (`composer.json`) is optional if you want to write a module and becomes required if you're going to distribute it
* Magento has its custom package declaration format that partly duplicates the `composer.json` format
* every `registration.php` is included on every request. OPCache reduces the performance impact. But the whole problem can be avoided

## Proposal

* Make `composer.json` the default way of declaring modules in Magento. `composer.json` is the standard package declaration format in PHP world. Switching to `composer.json` as the module declaration file will make Magento modules more familiar for non-Magento PHP developers and will reduce the need for duplication.
  * Modules in `/app/code` will still be supported. They will not be installed using Composer, but they will declare their metadata in `composer.json`
* Make `registration.php` optional
  * Add Composer installation hook that will:
    * Find all Composer packages installed in `/vendor` with `"type": "magento2-*"`
    * Find all folders in `app/code` containing `composer.json` with `"type": "magento2-*"`
    * Merge the lists and sort components based on their dependencies declared in `require` section of `composer.json` and `sequence` section of `module.xml`
    * Write the sorted list to `/vendor/magento_components.php`
  * This will reduce the number of files included by PHP process on every request
* Make `module.xml` optional. `composer.json` contains 'real' module name
* Using `composer.json` format does not mean that Composer will be the only way to install Magento modules. Modules installed in `app\code` folder will be supported

## POC

POC is implemented in https://github.com/magento-architects/magento2/tree/split-framework (see \Magento\Framework\Component\ComponentInstaller::install) for `di.xml`, `webapi.xml`, `extension_attributes.xml`, and Service Contract metadata.

Use https://github.com/magento-architects/magento-project for easy installation of a "hello world" application.
