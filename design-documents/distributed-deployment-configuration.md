# Distributed deployment configuration

## Problem

We want support distributed deployment of Magento: admin UI, cron, storefront UI, web API, and CLI.

Benefits
* Splitting modules decreases number of dependencies
* When deploying area specific instances, unnecessary dependencies not loaded and it should improve performance
* We don't have private interfaces (admin UI, cron) exposed, should improve security
* It's a step towards splitting monolith into services

This proposal solves problem of adding necessary configuration to allow distributed deployment.

## Design

To distinguish between different instances, we have these options

1. Don't add any special information on instances, developer always knows which instance is which.
2. Introduce marker component packages to mark instances (`magento/admin-ui`, `magento/storefront-ui`, `magento/cron`, etc).
3. Use projects, if we publish different projects, we can just use them _recommended_.

Approaches to specify where extension parts need to be installed
1. Reuse existing type field in composer configuration. Will need to add more types (`magento2-admin-ui-module`, `magento2-storefront-ui-module`, `magento2-cron-module`, etc).
    
    Cons
    * Will not work for specifying on which services extension parts need to be installed
    * Might need require adding configuration on instances to track which extensions installed on the system
    
2. Extend composer configuration _recommended_

    Modules would define on which instances they need to be installed.
    
    CatalogExtensionAdminUi/composer.json
    ```
    {
        "name": "vendor/catalog-extension-admin-ui",
        ...
        "extra": {
            "instances": [
                "admin-ui"
            ]
        }
    }
    ```
    
    To track which extensions installed on distributedly deployed Magento, developer can add metapackage names in composer.json files on instances.
    
    admin-ui-instance/composer.json
    ```
    {
        "name": "magento/admin-ui-project",
        "type": "project",
        "require": {
            "vendor/catalog-extension-admin-ui": "*",
            ...
        },
        ...
        "extra": {
            "extensions": [
                "vendor/catalog-extension"
            ]
        }
    }
    ```
    
    Pros
    * The same approach can be used later to define on which services which extension parts need to be installed.
    
3. If packages named by convention (`vendor/catalog-extension-admin-ui`, `vendor/catalog-extension-storefront-ui`, `vendor/catalog-extension-cron`, etc), developer can figure out where extension parts should be installed by package names.
    
    Cons
    * Will not work for specifying on which services extension parts need to be installed
    * Potentially not reliable
    * Might need require adding configuration on instances to track which extensions installed on the system
