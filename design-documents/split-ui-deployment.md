# Split UI deployment

## Problem

We want to allow deploy area specific instances, instances that will be responsible only for specific function: admin UI, cron, store front, web API, indexation, and CLI.

## Design

### Decomposing monolith

Area specific instance will contain only relevant set of modules. For instance, admin instance will contain AdminUi modules and their dependencies and will not contain store front modules.

We also want to allow deployment of area specific services, for instance Catalog and Catalog Cron.

Different options for area specific instances deployment
1. Composer project that depends on area specific components

Currently Magento 2 project depends on `magento/base` component, that contain project structure and entry points. We could decompose it in the following way
`magento/base` (will not contain area specific code anymore)
`magento/root-endpoint` - index.php endpoint
`magento/cron-endpoint` - cron.php endpoint
`magento/cron` - contains cron modules
`magento/admin-ui` - contains admin ui modules
`magento/ui` - contains store front modules

Installing all components will result in monolith. Installing `magento/base` and `magento/admin-ui` will result in admin instance.

Extension will have to depend on Magento module it extends and on endpoint/component that defines area specific instance.

Then we can look at extension dependencies and make modifications in area specific composer.json files.

Extension that don't depend on any areas, will be installed on on instances

2. Tool that uses new type of configuration to separate project into different instances

config.yaml
```
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
```

Configuration can be in the root of the project and tool can use this configuration to read composer.json and split project into multiple composer.json files for each project.

Pros
 * The same mechanism can be used to install extensions (add extension to needed composer.json files)
 
Cons
* Need to add new type of configuration 

3. Distribute different projects (_recommended_)

We create different projects (monolith, admin-ui, ui, etc) and distribute them.

Pros
* Simplifies developer workflow (see below)

Cons
* Will require us to publish more projects and this doesn't solve problem of installing extensions.

4. Convention based

We can have convention similar to this
* Modules that end with *admin-ui go to admin instance
* Modules that end with *ui and not *admin-ui go to store front instance

Cons
* Doesn't look like the convention can be reliable option (developers may not follow it, someone may call storefront module login-as-admin)ю


## Developer workflow

There are 2 options for developer work flows.

1. Work with monolith and split before deployment

Developer works with monolith application, workflow is similar to current.

Pros
* Magento doesn't need to distribute area specific projects
* Easier to install an extension, you just install on monolith and it's always one extension (if it's split, it will be aggregated using meta package)
* Easier to develop

Cons
* Not close to real system, some bugs might be introduced because we don't have code that is separated at the beginning

2. Magento distributes projects per area: admin UI, storefront, cron, etc (_recommended_)

Developer works with application that that have admin UI, storefront, cron, etc deployed separately.

Pros
* Close to production
* Easier to upgrade

Cons
* Upgrade of each instance need to be done separately
* Harder to develop

To install extension we can have a tool (wrapper around composer) that
* Receives paths to code bases of the instances and extension that need to be installed
* Checkout that Magento version
* Install extension on that Magento version
* Figure out parts of the extensions and then adds dependencies on each part of the extension to composer.json's of each instance

In both cases database upgrade need to be run only on admin instance or CLI instance.

## Open questions
