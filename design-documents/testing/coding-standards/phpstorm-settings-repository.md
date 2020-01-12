### Overview

There could be a problem with code formatting standards: the same code which is valid from
the static test's point of view can be formatted in different ways.

E.q. space between colon, type hint and method arguments list:
```php
public function execute(int $orderId): void;
```
vs
```php
public function execute(int $orderId) : void;
```

As a solution a "Settings Repository" PhpStorm built-in functionality can be used:
https://www.jetbrains.com/help/phpstorm/sharing-your-ide-settings.html#settings-repository

It provides a possibility to create Magento-maintained PhpStorm coding standard which all the contributors/core developers will follow.
All the code style updates (in the repository itself) will be synchronized with workstations without any manual actions.

There is already an existing example:
https://github.com/novikor/magento2-phpstorm-settings

This repository was created to follow Magento MSI coding standards for less painful PRs code review process.

  
