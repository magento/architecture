### Overview

Create new separate file called modules.php that contain
```php 
[
    modules => [...]
}
```

### Problem

File config .php contains shared website/stores and shared configuration. 
This file include in git index and check changes
This helps when we need to share some configuration across developers. 


The problem is when we install new module or change sequence, after command:

```bash
php bin/magento setup:upgrade
```
Magento changes __modules__ array and git marks this file as changed. 
We can't add this file to ___.gitignore___ because it contain shared configuration


### Proposal 

I think we can separate file ___config.php___ in 2 files:
* ___config.php___ that contains shared configuration and stores/website etc
* ___modules.php___ that contains only ___modules___ array that contains installed modules

Pluses:
* We can add ___modules.php___ to ___.gitignore___ file since magento can generate it, so we dont need to track the changes
* We can easy track changes in ___config.php___ because it contains only shared configuration
* If we needed to share status of module(Enabled/Disabled) we can use ___config.php___ 

### Description

The situation from my side

Yes I know that __env.php__ is merged with __config.php__ and has priority.

I have a big project and 2 different teams(work remote). What my code review flow?
* Change branch.
* Start command ```bash setup:upgrade```, ```bash setup:di:compile```, ```bash setup:static-content:deploy``` for check that changes not broken deployment or migration scripts and functionality are working
* Optionality make some changes in branch

Then if I create commit I need manually remove config.php from commit because it contained config data for different websites and stores but I not needed commit data from modules.

If I change branch I need create stash because I can't change branch while config.php are changed.

If module changes(add new modules) and we had conflict it's seems like merge hell.

For me add separate file is help me, because on my project we using cloud and deployment tools and I not needed check module order. 

And I can add it to .gitignore If we need disabled module we can do it from config.php,

How I see it:

* File __module.php__ contains information only for modules and has low priority with array merge

* File __config.php__ contain information for system config, websites, stores and optionality contains data from __module.php__ and can add manually(this can be used by disabled modules or manually sorting). 
This file can high priority then __modules.php__

* File __env.php__ contain locally config data and has higher priority than __config.php__ and __modules.php__
