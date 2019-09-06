# Implement PSR-11: Container Interface
PSR-11 is a set of basic interfaces to standardize dependency injection (obtaining objects and parameters) across frameworks and libraries. 

## Why?
Say a general purpose PHP package makes use of the PSR-11 Container Interface(CI) to instantiate objects in a dependency injection(DI) friendly manner:

``` php
<?php

namespace Acme\SomeComponent;

class SomeService
{
    /**
     * @var ContainerInterface
     */
    private $container;

    public function __construct(ContainerInterface $container)
    {
        $this->container = $container;
    }

    public function doSomething(string $message)
    {
        ...

        if ($this->container->has('\\Psr\\Log\\LoggerInterface')) {
            $this->container->get('\\Psr\\Log\\LoggerInterface')->info('Log entry');
        }

        ...
    }
}
```

When Magento would provide an implementation of the CI, this class would work out of the box, with zero configuration. 

## Benefits
- Better compatibility with general purpose PHP packages.
- Introduce Magento developers to a new PSR standard.
- Fit in better with the PHP ecosystem.
