# Automated config file cache invalidation

## Problem

Every time a configuration file (any xml file) or Service contract is created/edited/removed in Magento, the corresponding configuration cache has to be cleared manually. This degrades developer experience, and makes invalidations more expensive as all configuration types are reloaded when config cache is cleared.

External solutions for automated cache invalidation exist:
- PHPStorm allows to configure a command to be executed whenever a file changes.
- https://github.com/mage2tv/magento-cache-clean - IDE agnostic solution by [Vinai](https://github.com/Vinai|Vinai)

## Solution

To improve developer experience without reliance on external tools and additional setup, the approach similar to Opcache can be used:
- When configuration is written to cache, in addition to content, also write the names of files used, and directories checked for the configuration type. Keeping information about directories is requried to invalidate cache when new files are created in modules that did not have them before.
- Use [ini.opcache.validate-timestamps](http://php.net/manual/en/opcache.configuration.php#ini.opcache.validate-timestamps) and [ini.opcache.revalidate-freq](http://php.net/manual/en/opcache.configuration.php#ini.opcache.revalidate-freq) to define when configuration file timestamps should be checked
- Whenever a file or directory modification time is updated, reload the corresponding configuration type

## POC

POC is implemented in https://github.com/magento-architects/magento2/tree/split-framework (see https://github.com/magento-architects/magento2/blob/split-framework/lib/internal/Magento/Framework/Config/Config/Loader.php) for `di.xml`, `webapi.xml`, `extension_attributes.xml`, and Service Contract metadata.

Use https://github.com/magento-architects/magento-project for easy installation of a "hello world" applciation.
