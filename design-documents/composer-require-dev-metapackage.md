# Composer Require-dev Metapackage

This document describes the architectural change to make it easier to update the development packages with composer loaded by Magento.

## Situation

Magento uses some packages for development purposes which are added in the `require-dev` section of the root composer.json file.

## The problem

The root composer.json file is not updated when you update to the latest version of Magento. However sometimes the packages used in the `require-dev` section of the root composer.json file need to be updated to more recent version, or packages need to be added or removed from this section.

Since composer only installs the packages from the `require-dev` section from the root composer.json file we need to (manually) update the root composer.json file to reflect the changes.

## The solution

We can move the packages from the `require-dev` section out of the root composer.json file and create a meta packages referencing the needed packages. If packages need to be added, removed or updated the meta package can simply be updated without changing the root composer.json file. 

### Example root composer.json

```
{
    "name": "magento/magento2",
    "description": "Magento 2 composer.json",
    "type": "project",
    "license": [
        "OSL-3.0",
        "AFL-3.0"
    ],
    "config": {
        "preferred-install": "dist",
        "sort-packages": true
    },
    "require": {
        "magento/product-community-edition": "2.3.3",
        "creaminternet/magento2-setup": "@stable"
    },
    "require-dev": {
        "magento/product-community-edition-dev": "2.3.3"
    }
}
```

### Example meta package composer.json

```
{
  "name": "magento/product-community-edition-dev",
  "description": "Magento 2 Require dev meta package",
  "type": "metapackage",
  "require": {
    "allure-framework/allure-phpunit": "~1.2.0",
    "dealerdirect/phpcodesniffer-composer-installer": "^0.5.0",
    "friendsofphp/php-cs-fixer": "~2.14.0",
    "lusitanian/oauth": "~0.8.10",
    "magento/magento-coding-standard": "*",
    "magento/magento2-functional-testing-framework": "2.5.4",
    "pdepend/pdepend": "2.5.2",
    "phpcompatibility/php-compatibility": "^9.3",
    "phpmd/phpmd": "@stable",
    "phpstan/phpstan": "^0.12.3",
    "phpunit/phpunit": "~6.5.0",
    "sebastian/phpcpd": "~3.0.0",
    "squizlabs/php_codesniffer": "~3.4.0"
  }
}
```