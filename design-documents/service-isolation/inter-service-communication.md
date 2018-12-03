### Inter-service communications
This documents describes how exacly services will be communicating in a distributed setup.
#### Service declaration
Services will be declared in Api modules with interfaces. These modules will be installed on all nodes.
Example:
```php
namespace Magento\ServiceBApi;

interface BManagerInterface
{
    public function save(BDataInterface $data): BDataInterface;
    
    public function delete(string $id): void;
}
```
or for DTOs
```php
namespace Magento\ServiceBApi\Data;

interface BDataInterface
{
    public function getId(): string;
  
    public function getName(): string;
}
```
The API modules will just contain these declarations and nothing else (no di.xml, schema.xml, webapi.xml etc).
 
Even services that are not written using Magento (or even PHP) will be declared in this way for nodes using Magento.
 
#### Publicly available Web APIs
To enable publicly available APIs (that frontend and 3rd party systems can use) the _front node_ will have to have -Module-Webapi
or -Module-Graphql modules installed.
 
#### Real and proxy modules
If a node wants to use services locally then it will just have real modules installed (e.g. "Catalog"), if a node is supposed
to access services remotely it will have proxy modules installed (e.g. "CatalogProxy").
 
Proxy modules will contain services/DTOs implementations and a di.xml file to declare preferences for when a services declared in an API
module is called a proxy implementation to be returned.
 
For a services such implementation will look like this:
```php
namespace Magento\ServiceBProxy;

class BManagerProxy implements Magento\ServiceBApi\BManagerInterface
{
    ....
    
    public function save(BDataInterface $data): BDataInterface
    {
        $promise = $this->invoker->invoker(
            \Magento\ServiceBApi\BManagerInterface::class,
            'save',
            [$data]
        );
        $result = null;
        $promise->then(function ($serviceExecResult) use (&$result) { $result = $serviceExecResult; });
        $promise->otherwise(function (InvokerException $exception) { throw $exception; });
        $promise->wait();
        
        if ($result instanceof \Throwable) {
            throw $result;
        }
        
        return $result;
    }
    
    ....
}
```

Proxy services will use _InvokerInterface_ to call remote services. It will return a promise which will pass a service execution result
to _then_ callback and an exception to _otherwise_ callback if something during invokation went wrong. The service execution result may
contain the return value of a service or an exception it has thrown. The promise that invoker returnes may be used in both synchronous
and asynchronous ways to be able to work with both types of services. More details about the promise <<<<<<<<_here_>>>>>>>.
