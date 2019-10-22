# Enums

## Overview

There is a need to restrict values of a parameter or return value of the method. This is commonly solved with [enumerate types](https://en.wikipedia.org/wiki/Enumerated_type). Introduction of the enums required in order to add [support of different databases](https://github.com/magento/architecture/pull/318).

## Design

Using enums gives the following benefits:
* You can specify list of possible values `function setEngine(EngineEnum $engine) {`
* You can get list of all possible values
* You can prohibit adding new values by making enum class final

There is native PHP extension [SplEnum](http://www.php.net/manual/en/class.splenum.php). But it's not bundled with PHP by default and adding requirement on it means requiring to [install dependency](https://www.php.net/manual/en/spl-types.installation.php).

There is [widely used](https://github.com/myclabs/php-enum#related-projects) PHP implementation [myclabs/php-enum](https://github.com/myclabs/php-enum) of enums.

### Example of usage

```php
class Engine extends MyCLabs\Enum\Enum
{
    private const BASIC = 'basic';
    private const MEMORY = 'memory';
}

$myEnum = new MyEnum('basic');
echo $myEnum; // Outputs basic
```

We can add `myclabs/php-enum` to Magento by introducing new framework library that will be a wrapper.
