# Overview

In order to maximize the speed of the Magento CLI app, extraneous processing should be minimized. Magento ships with multiple commands which require significant processing to determine their "definitions" (`\Magento\Setup\Console\Command\ConfigSetCommand` is noteworthy). Rather than spend effort minimizing the impact of each of these commands, we can introduce Symfony's concept of lazy loaded commands. When a command is added to the application this way, it isn't initialized until it's needed (either to use the command or as part of `list`).

## Terms

- Magento CLI app: a Symfony Console application (subclassed in this case)
- Magento command: a class which extends a Symfony Console Command class, to be used in the Magento CLI app
- definition: the set of [input arguments and options](https://symfony.com/doc/4.4/console/input.html) for a command
- lazy loaded command: A command which isn't initialized until it is needed. See [Symfony's docs](https://symfony.com/doc/4.4/console/lazy_commands.html) for more
- Command Loader: an instance of `\Symfony\Component\Console\CommandLoader\CommandLoaderInterface`

## Problem

_For initial discussion, see https://github.com/magento/magento2/issues/29266_

### Summary
A Magento command added via `\Symfony\Component\Console\Application::add` must be initialized beforehand, including its definition. For some commands, this requires significant processing.

### Examples

Both `\Magento\Setup\Console\Command\InstallCommand` and `\Magento\Setup\Console\Command\ConfigSetCommand` collect the full list of modules, probe them for ConfigOptionsList di configs, and add all discovered options. None of this work is cached, so it happens every time the Magento CLI app is invoked.


## Design

To solve this problem, I propose that the Magento CLI app uses an instance of Symfony's `\Symfony\Component\Console\CommandLoader\CommandLoaderInterface` to provide access to commands which would otherwise require initialization overhead.

### Implementation Challenges

#### DRYness issues

One candidate for direct usage in the Magento CLI app is the `FactoryCommandLoader`. It is provided with an associative array of commands with command names as keys and command initializers as values. The obvious issue with this option is that command names are generally not defined in Magento commands in a statically-available way (such as `const NAME = 'cache:flush;`). As such, command names must be repeated in string literals.

#### Backwards compatibility

At this moment, Magento commands are added to the Magento CLI app via `\Magento\Framework\Console\CommandListInterface`. This interface will not suffice for use with lazy loaded commands because its output is initialized commmands. That means either new functionality for adding lazy-loaded commands is required OR a major BC break will occur.

#### Maintainability

If new functionality is added to support lazy loaded commands in addition to the current system, both of these systems must be maintained.

#### Lack of Integration points

The current structure of the Magento CLI app does not provide an obvious place for either injecting or creating its own Command Loader. All Magento commands are currently added during the Magento CLI app's `init` method, which calls `getDefaultCommands`, which calls other methods, all of which return Magento command instances. It's not ideal to mutate the Magento CLI app inside a getter, so initializing a Command Loader there is not ideal.

### Proposed implementation

#### New Class: MagentoCommandLoader

The exact structure of `$commands` is undecided. For simplicity, this proposal assumes an associative array of Magento command names to classnames.

```php
namespace Magento\Framework\Console;

use Symfony\Component\Console\CommandLoader\CommandloaderInterface;

class MagentoCommandloader implements CommandLoaderInterface
{
    private $commands;
	private $obj;

    public function __construct(ObjectManagerInterface $obj, array $commands = [])
    {
        $this->commands = $commands;
		$this->obj = $obj;
    }
	
	public function get($name)
	{
	    $class = $this->commands[$name];
	    return $this->obj->get($class);
	}
	
	public function has($name)
	{
	    return isset($this->commands[$name]);
    }
	
	public function getNames()
	{
	    return array_keys($this->commands);
    }
}
```

#### Connection to Magento CLI app

Next, the Command Loader must be initialized and attached to the Magento CLI app:
```php

//inside \Magento\Framework\Console\Cli::__construct

try {
    // ...
	$this->initObjectManager();
} catch (\Exception $exception) {
    // ...
}

parent::__construct($name, $version);
// ...
$this->logger = $this->objectManager->get(LoggerInterface::class);

// NEW CODE!
$this->setCommandLoader($this->objectManager->get(MagentoCommandloader::class, [
    'obj' => $this->objectManager
]));
```

#### DI configuration
At this point, when a Magento command's initialization should be deferred for the sake of speed, it can be added via the `MagentoCommandLoader` in DI:

```xml
<type name="MagentoCommandLoader">
    <arguments>
        <argument name="commands" xsi:type="array">
            <item name="a:heavy:command" xsi:type="string">My\Module\Commands\HeavyCommand</item>
        </argument>
    </arguments>
</type>
```

#### Notes

* The current `CommandList` class and related DI configuration will be left alone, except where Magento commands must be lazily loaded
* As mentioned, the structure of the `$commands` parameter of `MagentoCommandLoader` is just one idea. I'm not particularly attached to it

