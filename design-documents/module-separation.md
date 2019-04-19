# Module separation

## About

This document clarifies the following two questions of the original [service isolation proposal](service-isolation.md)
1. What new modules should be called
2. What to do with module namespaces, keep old or change so they match new module name

## Design

### Web API
It makes sense to deploy admin and storefront API to different instances.

### Module naming

1. Module naming

    The following is proposed naming for modules:
    API (service contracts)
    * CatalogApi :white_check_mark:
    
    The following is proposed naming for modules:
    Admin Ui and admin related business logic (blocks, controllers, static resources, models and resources potentially)
    * CatalogAdmin
    * CatalogAdminUi :white_check_mark:
    * CatalogBackoffice
    
    Storefront Ui and storefront related business logic (blocks, controllers, static resources, models and resources potentially)
    * CatalogStorefrontUi :white_check_mark:
    * CatalogStorefront
    * CatalogStore
    
    Cron (models and resources potentially)
    * CatalogCron :white_check_mark:
    
    WebApi
    * CatalogWebApi :white_check_mark:
    
    WebApi GraphQL
    * CatalogGraphQl :white_check_mark:
    
    Common code (business logic used by multiple areas, models, resource modelss)
    * CatalogCommon
    * CatalogShared
    * CatalogDomain :white_check_mark:
    * CatalogBusinessLogic
    * CatalogBl
    * CatalogImpl
    
    Common static resources
    * CatalogCommonUi
    * CatalogUi :white_check_mark:

2. Keep old or change namespaces

    Per [service isolation proposal](service-isolation.md) each module will be split on the admin, storefront, cron, webapi, cli and common parts. For example, `Catalog`, will be split on `CatalogAdminUi`, `CatalogStorefrontUi`, `CatalogCron`, etc.
    
    For dealing with namespaces, there are the following options.
    
    1. Change, introduce new namespaces and preserve old ones for backwards compatibility. Fix namespaces in the modules. Old module Catalog will contain classes under old namespace, marked with @deprecated annotation, that extend classes with new namespaces and di configuration for backwards compatibility. Then in the future minor release these classes will and di configuration will be removed.
        
        Pros
        * We going to have correct namespaces after module split, for instance \Magento\CatalogAdminUi\Block\…
        * Allows transition period
        
        Cons
        * Introduce configuration for virtual types under new namespace that will duplicate configuration for virtual types with old namespace
        * Duplicate configuration for types (params definition)
        * Double configuration for autoloader (not sure if it’s concern, but would be good to measure)
        * Amount of modules will double, will be harder to navigate codebase and more overhead to support until we deprecate modules with old namespaces
        * May need to add new tests to make sure everything is consistent (edited) 
        * One minor release probably is not enough time for developers to fix their extensions
    
    2. Change, introduce new namespaces and don't preserve backwards compatibility
        
        Pros
        * We going to have correct namespaces after module split
        * No overhead from preserving keeping backwards compatibility
        
        Cons
        * Backwards incompatible, all of the extensions will be affected
    
    3. Preserve old namespaces. Classes in CatalogAdminUi, CatalogStorefrontUi, CatalogCron, etc will have Catalog namespace :white_check_mark:
        
        Pros
        * No overhead from preserving keeping backwards compatibility
        
        Cons
        * We going to have namespaces that don't follow the pattern
        * Will need to fix existing tests (not a big con really)
        * Will have to decide what to do with new classes we introduce in the modules that have classes with old namespaces. Introduce with old namespace for consistency or with new namespace that matches module name to have correct namespace. Either way this might be a little confusing, and different from the case when we create brand new module with correct namespace

## Open question
Is it ok that we going to have database upgrade command on storefront?