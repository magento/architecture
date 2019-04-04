# Split modules/extensions strategy

## Overview

According to the [general schema](https://github.com/magento/architecture/blob/master/design-documents/service-isolation.md#split-modules), we want to split current modules into multiple modules to decouple dependencies.
As Magento has a lot of extensions, which can be also affected, we need to propose a strategy for splitting old modules|extensions into new modules.
The desired state that all modules/extensions should depend only on used component's parts like API, UI, etc. but not on whole component like now.

![Components Dependencies](img/components-dependencies.png)

## Split strategy

There are multiple strategies how to split existing modules and extensions. Each strategy may imply a gradual separation into new modules.
A tool might be used to split modules like in [prototype](https://github.com/magento-architects/magento2/tree/split-framework/app/code/Magento). It does not guarantee that all modules can be split automatically but minimize the routine work.

### New Module - new class namespace

This strategy assumes splitting existing modules into new with namespaces changes according to the new modules names.
All dependant code should use new class namespaces.

Desired state for dependencies:

```json
"require": {
    "magento/module-catalog-api": "^104.0.0",
    "magento/module-catalog-adminui": "^104.0.0",
    "magento/module-catalog-ui": "^104.0.0"
}
```

Pros:
 - Introduce "right" dependencies
 - Dependant code use only needed modules
 
Cons:
 - Cause BIC as all dependant code should you new namespaces
 - All extensions should be affected
 
### New module - old class namespace
 
This approach proposes split modules but leave class namespaces as is. So all dependant code should require new packages in `composer.json` but does not require changes in codebase.
 
 Desired state for dependencies:
 
 ```json
"require": {
    "magento/module-catalog-api": "^104.0.0",
    "magento/module-catalog-adminui": "^104.0.0",
    "magento/module-catalog-ui": "^104.0.0"
}
```
 
Pros:
 - Introduce "right" dependencies
 - Dependant code use only needed modules
 - No changes in the dependant code
  
Cons:
 - Dependant modules and extensions should update `composer.json`
 - Extensions should split their modules in the same way as core modules and specify the correct dependencies

### New module - old composer

This strategy proposes to expose all new modules via one (currently existing) `composer.json` and came from [this proposal](https://github.com/magento/architecture/issues/88).
The main idea that extensions will use the same dependency in `composer.json` which will be distributed as metapackage. Magento core modules will depend on new modules like `catalog-ui` but not to `catalog` metapackage, such metapackage will be distributed separately.
Such strategy assumes that existing modules will use the old namespaces but new modules will have the new namespaces.

 ```json
"require": {
    "magento/module-catalog": "^103.0.0|^104.0.0"
}
```

Unlike the [original proposal]((https://github.com/magento/architecture/issues/88)), the `composer.json` probably will have another structure. As an example for `catalog` metapackage:

```json
"require": {
    "magento/module-catalog-api": "^104.0.0",
    "magento/module-catalog-adminui": "^104.0.0",
    "magento/module-catalog-ui": "^104.0.0"
}
```

The original proposal uses `self.version` for `catalog` related modules but such approach does not allow bump modules versions independently and update just their versions in the metapackage (according to the original proposal, all versions of `catalog` related modules should be bumped).
Also, the usage of `provide` package link, probably is not the best option, as in our case `catalog` metapackage knows about exact packages not virtual interfaces.

Pros:
 - No changes in the dependant code
 - Allows to specify multiple package versions
  
Cons:
 - Introduce a metapackage with all needed dependencies
 - `composer.json` anyway should be changed to specify needed version of module
 - All extensions still have coupling to all dependant code even if they do not use it
 - Such approach won't get benefits of modules decoupling as an extension still depends on the whole component
 - To get benefits of distributed deployment extensions should be split as well
 
## Summary

The third approach looks like more convenient for now as allow combining two previous strategies: old modules will have old namespaces, new modules - new namespaces. Also, this approach allow splitting extensions according to new modules decomposition iteratively.
The new modules won't be introduced in metapackage and extension developers should require them in `composer.json` manually.

Once a metapackage released it might be marked as deprecated to provide only the one recommended approach how the dependencies should be required.