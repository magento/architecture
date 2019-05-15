# Simplified Module Declaration

## Problem

Declaring a Magento module requires following files:

* add `composer.json` - to declare package name, description, version, dependencies, etc
* add `module.xml` - to declare module name and version dependencies
* add `registration.php` - to let Magento know where the module resides, for Magento to be able to read configuration files

The presense of 3 files causes duplication and complicated module management and sorting mechanisms. Current module sorting mechanism is inefficient.

Every `registration.php` is included by Magento on every request. While with opcache this does not cause signfiicant performance impact, it can be avoided.

## Solution

* Rely on **composer** for module management, eliminate part of Magento module management behavior.
* Register composer installation hook that will perform magento-specific module management.
* Eliminate `registration.php` files.
  * Composer installation hook will generate a 'magento_components.php' file that will contain all magento components and their installation paths. Only this file will be included on every request.
  * Move module sorting behavior to composer installation hook. Use third-party topological sorting library. Write sorted modules to `magento_components.php`. This will allow to move module sorting from deploy stage to build stage of publication.
* Eliminate `module.xml` - composer.json contains 'real' module name. Move 'legacy' module names in `composer.json`
* Use `composer.json` for module declaration. 

## POC

POC is implemented in https://github.com/magento-architects/magento2/tree/split-framework (see \Magento\Framework\Component\ComponentInstaller::install) for `di.xml`, `webapi.xml`, `extension_attributes.xml`, and Service Contract metadata.

Use https://github.com/magento-architects/magento-project for easy installation of a "hello world" applciation.
