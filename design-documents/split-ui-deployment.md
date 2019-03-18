# Split UI deployment

## Problem

We want to allow deploy area specific instances, instances that will be responsible only for specific function: admin UI, cron, store front, web API, indexation, and CLI.

## Design

### Decomposing monoliths

Area specific instance will contain only relevant set of modules. For instance, admin instance will contain AdminUi modules and their dependencies and will not contain store front modules.

We also want to allow deployment of area specific services, for instance Catalog and Catalog Cron.


Different options for area specific instances deployment
1. Composer project that depends on area specific components

Currently Magento 2 project depends on `magento/base` component, that contain project structure and entry points. We could decompose it in the following way
magento/base
magento/root-endpoint - index.php endpoint
magento/cron-endpoint - cron.php endpoint
magento/cron - contains cron modules
magento/admin-ui - contains admin ui modules
magento/store-front - contains store front modules

Installing all components will result in monolith. Installing magento/base and magento/admin-ui will result in admin instance.

Cons
* Doesn't solve problem of installing extensions

2. Tool that uses new type of configuration to separate project into different instances

instances.yaml
admin-ui
    magento/catalog-admin-ui
    magento/customer-admin-ui
    magento/cms-admin-ui
    ...
ui
    magento/catalog-ui
    magento/customer-ui
    magento/cms-ui
cron
    magento/catalog-cron

Configuration can be in the root of the project and tool can use this configuration to read composer.json and split project into multiple composer.json files for each project.

Cons
 * The same mechanism can be used to install extensions (add extension to needed composer.json files)

Tool may have the following interface
php split-deployment.php path/to/instances/config.yaml path/to/directory/with/composer-files

The output of the tool would be composer.json files.

3. Distribute different projects

We create different projects (monoliths, admin-ui, ui, etc) and distribute them.

Cons
* Will require us to publish more projects and this doesn't solve problem of installing extensions.

4. Convention based

We can have convention similar to this
* Modules that end with *admin-ui go to admin instance
* Modules that end with *ui and not *admin-ui go to store front instance

Cons
* Doesn't look like the convention can be reliable option (developers may not follow it, someone may call storefront module login-as-admin)ю


### Installing extensions

There are should be a way to install extension on a new type of deployment. Composer doesn't have enough abstractions to cover the use cases. Multiple parts of the same extension need to be installed on different instances.

Extension can come with config that would list on which instances it should be installed.

instances.yaml
admin-ui
    vendor/extension-admin-ui
ui
    vendor/extension-ui
    
Extension will have to be installed on monoliths first and then, monolith composer.json can be split into multiple composer.json files using tool.
    

Tool should allow us
* Allow to deploy area specific apps
* Allow to deploy different services
* Allow to deploy area specific services
