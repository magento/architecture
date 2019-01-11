# Config file cache

1. On every request Magento application reads, validates, and merges configuration files from `app/etc/` folder and `/etc` folders of all components. Every configuration type has its own file (`di.xml`, `webapi.xml`, `config.xml`). Every module can provide configuration for different application areas (`/etc/di.xml`, `/etc/adminhtml/di.xml`, `/etc/frontend/di.xml`)

2. Merged content is then cached to avoid expensive operations on consequent requests.

3. Together with configuration, configuration file metadata is cached:
  * Files read to load the configuration and their timestampls.
  * Folders checked for configuration files, together with their timestamps. For folders that did not exist, paths are stored.
  
4. If [ini.opcache.validate-timestamps](http://php.net/manual/en/opcache.configuration.php#ini.opcache.validate-timestamps) is enabled in php configuration, appliaction will check if configuration files and folders were updated or created. The validate frequency can be configured with (http://php.net/manual/en/opcache.configuration.php#ini.opcache.revalidate-freq). This way, confugration cache invalidation is synchronised with PHP opcache. In development environments, revalidation should be frequent (2 seconds by default in PHP). In production revalidation should be disabled. 
