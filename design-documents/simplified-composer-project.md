# Simplified Composer Project

Composer project is intended for production use.
Development needs can be covered by a custom/adopted Composer project.
We can provide such `composer.json` in documentation.

## Proposed Changes on High-Level

1. Remove `dev` sections
1. Remove `autoload` parts that cover development scenarios, such as git-based Magento paths
1. Move component-specific parts to corresponding Composer packages. See examples in "Changes" section

As a result the project becomes simple and would have fewer reasons to change ([Single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle).
Responsibilities move to packages really containing corresponding code instead of delegating it to the project's `composer.json`.

## Deployment scenarios

All scenarios are focused on production deployment and so should lead to absence of development resources (wherever possible).

### Initial installation

Performed as described on [DevDocs](https://devdocs.magento.com/guides/v2.3/install-gde/composer.html):
```
composer create-project --repository=https://repo.magento.com/ magento/project-community-edition <install-directory-name>
```

### Upgrade with the proposed project structure

```
composer require magento/product-community-edition <new-product-version>
```

### Upgrade with the old project structure and preserve it 

Use Magento's composer plugin.

### Upgrade from the old project structure to the new one

Modify `composer.json` to eliminate non-production dependencies, as described in the proposal.
After that use approach described in "Upgrade with the proposed project structure"

## Low-level changes

Based on current `composer.json` the following changes are proposed:
1. Remove `require-dev` section
1. Move `"conflict": { "gene/bluefoot": "*" }` to `magento/page-builder` module
1. Remove `"Magento\\Framework\\": "lib/internal/Magento/Framework/"`, `"Magento\\": "app/code/Magento/"` and `"app/code/"` from `autoload` section
1. Move `"psr-4": { "Magento\\Setup\\": "setup/src/Magento/Setup/", "Zend\\Mvc\\Controller\\": "setup/src/Zend/Mvc/Controller/" }` to `magento/base` package
1. Move `"psr-0": { "": [ "generated/code/" ] }` to `magento/framework`
1. Remove `"files": [ "app/etc/NonComposerComponentRegistration.php" ]`
1. Remove `autoload-dev`

Current `composer.json`:
```json
{
"name": "magento/project-community-edition",
    "description": "eCommerce Platform for Growth (Community Edition)",
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
        "magento/product-community-edition": "2.3.1"
    },
    "require-dev": {
        "friendsofphp/php-cs-fixer": "~2.13.0",
        "lusitanian/oauth": "~0.8.10",
        "magento/magento2-functional-testing-framework": "~2.3.13",
        "pdepend/pdepend": "2.5.2",
        "phpmd/phpmd": "@stable",
        "phpunit/phpunit": "~6.5.0",
        "sebastian/phpcpd": "~3.0.0",
        "squizlabs/php_codesniffer": "3.3.1",
        "allure-framework/allure-phpunit": "~1.2.0"
    },
    "conflict": {
        "gene/bluefoot": "*"
    },
    "autoload": {
        "psr-4": {
            "Magento\\Framework\\": "lib/internal/Magento/Framework/",
            "Magento\\Setup\\": "setup/src/Magento/Setup/",
            "Magento\\": "app/code/Magento/",
            "Zend\\Mvc\\Controller\\": "setup/src/Zend/Mvc/Controller/"
        },
        "psr-0": {
            "": [
                "app/code/",
                "generated/code/"
            ]
        },
        "files": [
            "app/etc/NonComposerComponentRegistration.php"
        ],
        "exclude-from-classmap": [
            "**/dev/**",
            "**/update/**",
            "**/Test/**"
        ]
    },
    "autoload-dev": {
        "psr-4": {
            "Magento\\Sniffs\\": "dev/tests/static/framework/Magento/Sniffs/",
            "Magento\\Tools\\": "dev/tools/Magento/Tools/",
            "Magento\\Tools\\Sanity\\": "dev/build/publication/sanity/Magento/Tools/Sanity/",
            "Magento\\TestFramework\\Inspection\\": "dev/tests/static/framework/Magento/TestFramework/Inspection/",
            "Magento\\TestFramework\\Utility\\": "dev/tests/static/framework/Magento/TestFramework/Utility/"
        }
    },
    "version": "2.3.1",
    "minimum-stability": "stable",
    "repositories": [
        {
            "type": "composer",
            "url": "https://repo.magento.com/"
        }
    ],
    "extra": {
        "magento-force": "override"
    }
}
```

Proposed `composer.json`:
```json
{
  "name": "magento/project-community-edition",
  "description": "eCommerce Platform for Growth (Community Edition)",
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
    "magento/product-community-edition": "2.3.1"
  },
  "autoload": {
    "exclude-from-classmap": [
      "**/dev/**",
      "**/update/**",
      "**/Test/**"
    ]
  },
  "version": "2.3.1",
  "minimum-stability": "stable",
  "repositories": [
    {
      "type": "composer",
      "url": "https://repo.magento.com/"
    }
  ],
  "extra": {
    "magento-force": "override"
  }
}
```
