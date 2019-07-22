# Magento Functional Test packaging
The purpose of this document is to define a strategy of functional test packaging.

## Terminology
`MFTF` - Magento Functional Testing Framework

`MFTF tests` - means tests written to cover Magento functionality using MFTF

## Overview
### Current situation
In git repo all MFTF tests currently located under `app/code/Magento/*/Test/Mftf` directory.
When Magento releasing new version we package all modules to own packages and distribute them via repo.magento.com.
It means that all MFTF tests currently also distributed as a part of `magento2-module`.
When SIs or Extension developers installs Magento via Composer they are able to run and use MFTF tests for their own needs.

### Problem
This approach leads to situation where production environment always contains code (in our case xml configurations) which will never be used.

>There are couple reasons to separate tests from core distribution:
>
>Security:
> - reduce possible attack surface (less code)
> - the test code is not written with security in mind
> - even if we protect the access via .htaccess, some sites run Nginx or skip .htaccess making this code accessible
> - we had security issues in the past that resulted in remote code exec using code related to tests
>
>Future-proof:
> - ability to release separately, on separate cadence
> - matches the general plan for all other non-critical modules - we want to modularize and release separately things like payment extensions, shipping and more.


### Proposed solution
In order to be able to package Functional Tests separately and match [Module separation naming](https://github.com/magento/architecture/blob/master/design-documents/module-separation-naming.md)
proposal this proposal goes to direction where for each module which can be covered with functional tests we will have sibling module with suffix `{ModuleName}FunctionalTest`.
To avoid mixing application and test modules they should be moved to `dev/tests/acceptance` directory in git repo.

For example:
 - `CatalogAdminUi` module will have sibling module `CatalogAdminUiFunctionalTest`
 - `CatalogAdminUiFunctionalTest` will be dedicated only to have Magento Automated Functional Tests for `CatalogAdminUi` module
 
This module will have his own structure according to MFTF requirements:
```   
   `Magento`
       `CatalogAdminUiFunctionalTest`
           `ActionGroup`
           `Data`
           `Metadata`
           `Page`
           `Section`
           `Suite`
           `Test`
           `LICENSE.txt`
           `README.md`
           `composer.json`
```

`composer.json` - will have his own dependencies on other test modules and in `suggest` section it will have information about what Magento Module this test package covers.

For example:
```
{
    "name": "magento/module-catalog-admin-ui-functional-test",
    "description": "N/A",
    "config": {
        "sort-packages": true
    },
    "require": {
        "magento/magento2-functional-testing-framework": "*",
        "magento/module-backend-functional-test": "*",
        "magento/module-catalog-inventory-functional-test": "*",
        "magento/module-store-functional-test": "*",
        "magento/module-ui-functional-test": "*"
    },
    "suggest": {
        "magento/module-catalog-admin-ui": "Test coverage for magento/module-catalog-admin-ui: ~100.0.0"
    },
    "type": "magento2-functional-test-module",
    "license": [
        "OSL-3.0",
        "AFL-3.0"
    ]
}
```

#### Magento metapackage
In Magento `magento/project-community-edition` we will be able to specify 2 metapackage under different blocks.
Magento application will go to `require` section as it is right now and we will be able to put `magento/functional-test-project-community-edition` under `require-dev` section.
In case someone want to have tests and Magento in 1 project they will be able to do it in a very simple and sophisticated way.
But if someone needs only tests they are able to install only `magento/functional-test-product-community-edition` by running `composer require magento/functional-test-product-community-edition 2.3.1`

This approach also fits Service Isolation proposal because any isolated service will have own set of test packages and they can be collected in Service metapackage.

#### NOTE:
Module type `magento2-functional-test-module` will not be installed via Web Setup Wizard since they must not be installed on production environment.
This packages will be used only on CICD systems and Development environments where they should be installed via `composer` command.


### Benefits:
1. Current solution gives us possibility to separate tests from Production code.
2. Agile in terms of independent releases from `magento2-module` package type.
3. Gives us possibility to install Magento application and Tests separately.
4. Visioning can be independent for test package and module itself.

### Disadvantages:
1. Dependencies between test modules must be maintained manually by Automation engineer.

### Implementation tasks
1. Move all existing tests from modules to modules by pattern `{ModuleName}FunctionalTest` and relocate them from `app/code` to `dev/tests/acceptance`
2. Update MFTF framework configuration to read XML entities based on new module structure
3. Update infrastructure to put `magento2-functional-test-module` package types into separate metapackage and put this metapackage under `require-dev` section
