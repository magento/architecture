### Overview

Create new separate file called modules.php that contain
```php 
[
    modules => [...]
}
```

###Problem

File config .php contains shared website/stores and shared configuration. 
This file include in git index and check changes
This helps when we need to share some configuration across developers. 

The problem is when we install new module or change sequence, after command:

```bash
php bin/magento setup:upgrade
```
Magento changes __modules__ array and git marks this file as changed. 
We can't add this file to ___.gitignore___ because it contain shared configuration


###Proposal 

I think we can separate file ___config.php___ in 2 files:
* ___config.php___ that contains shared configuration and stores/website etc
* ___modules.php___ that contains only ___modules___ array that contains installed modules

Pluses:
* We can add ___modules.php___ to ___.gitignore___ file since magento can generate it, so we dont need to track the changes
* We can easy track changes in ___config.php___ because it contains only shared configuration
* If we needed to share status of module(Enabled/Disabled) we can use ___config.php___ 
