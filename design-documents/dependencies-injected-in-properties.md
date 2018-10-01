## Introduce the possibility to inject dependencies directly to properties
### Reasons
* Allowing backward compatible introduction of new dependencies to classes without having to use ObjectManager directly  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Currently we have to add optional arguments to classes and invoke ObjectManager::getInstance() to get a dependency
for cases when the constructor is invoked from a child class. Declaring such dependencies for properties will allow us to avoid this.
* Allowing circular dependencies  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;In some cases circular dependencies can be really hard to avoid - by injecting dependencies into properties
after requested object instances are created we can make circular dependecies possible.
### Details
#### XML
We'd have to add *properties*/*property* elements for *type* elements in di.xml's.
*property* element would have *type* and *shared* attributes just as *argument* does.  
#### Injection
Key points:  
* Dependencies list for properties will be collected and injected after
the requested object's instance is created and added to the shared map (if it's a singleton) to allow circular dependencies
* A dependency will __NOT__ be inject into a property if it's not equal to *null*
for cases when a constructor has already set a value for the property.
* Dependencies for properties will be collected for all of an object's parent classes and injected
starting from dependencies for the highest class in the hierarchy
* Proxies will be injected to properties to avoid initiating objects without a need in case
child classes override all methods using those parent's properties
#### ObjectManagerInterface
* In order to preserve backward compatibility we can't add an argument to ObjectManagerInterface::create(),
so to distinguish values passed in $arguments as property dependencies I propose we name them as *ClassName*::*property*
#### Using ObjectManager::getInstance()
* Mark it as deprecated because the only sensible use-case was introduction of new dependencies to classes.
Manually created factories can just use ObjectManagerInterface as a dependency
* A static test forbidding use of ObjectManager directly (except for phpunit tests) to be created
### ToDo list
* Add *property* and related configuration to di.xml's schema
* Adjust *Magento\Framework\ObjectManager\FactoryInterface* implementations to allow injecting dependencies to properties
* Mark *ObjectManager::getInstance()* as deprecated
* Add PHPMD rule to prevent *ObjectManager::getInstance()* usages
