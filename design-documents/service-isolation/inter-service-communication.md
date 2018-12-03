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
 
Proxy modules will contain services/DTOs implementations and a di.xml file to declare preferences for proxy implementations to be
used.
 
For a service such implementation will look like this:
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
to _then_ callback and an exception to _otherwise_ callback if something during the remote call went wrong (like a network error). The
service execution result may contain the return value of a service or an exception it has thrown. The promise that invoker returnes may
be used in both synchronous and asynchronous ways to be able to work with both types of services. More details about the promise
<<<<<<<<_here_>>>>>>>.
 
Proxy DTOs will look like this:
```php
namespace Magento\ServiceBProxy;

class BDataProxy implements BDataInterface
{
    private $id;
    
    private $name;
    
    public function __construct(string $name, ?string $id) {
        $this->name = $name;
        $this->id = $id;
    }
    
    public function getId(): ?string
    {
        return $this->id;
    }
    
    public function getName(): string
    {
        return $this->name;
    }
}
```
Usually DTOs implementations are models in real modules so we need a class to be initialized when client code communicates with a
remote service and simple DTOs will sufice.
 
#### The invoker
The invoker will be a part of Magento\Framework and will be present in every node installation. Invoker's job is to find a remote node's
address based on given class name and method, send the request, receive response and convert it to either requested service method return type or an exception if it was thrown by the remote service. It returns a promise to support asynchronous services and because
the requests to remote services will be asynchronous (depends on implementation). More details on the invokers API and SPI
<<<<here>>>>.
 
#### Gateway
For actualy delivering remote service requests RESTful API will be used - it's an established way to execute requested methods and to
serialize arguments and return values. For sending HTTP requests [Guzzle](https://github.com/guzzle/guzzle) library will be used - it's
a stable and well supported library which is capable of sending concurrent requests and allows wide customiztion. Guzzle will be configured to use cUrl (cUrl multi) as a backend. When asynchronous requests are sent via Guzzle it returns a promise which can easily
be converted to Magento's promise.
#### Lookup
The information on how to reach services will be provided in env.php like this:
```php
return [
    'service_base_urls' => [
        'Magento\\Catalog' => 'http://catalog.mydomain.com/',
        'Magento\\Checkout' => 'https://checkout.mydomain.com/',
        'Magento\\Customer' => 'http://192.168.1.1/',
    ],
];
```
The invoker will look up the information based on provided services' class names.
 
#### Changes to RESTful Web API mechanism
We need to able so send and receive more information when using RESTful web API for inter-service communication than for regular
communications so to distinguish regular endpoints and enpoints meant for inter-service communications we will declare them in a
seperate yet similar xml files - _inter_webapi.xml_. Since we declare services by declaring PHP interfaces there is no reason to provide
human-readable URLs for endpoints meant for inter-service communications, thus we'll make it possible to omit _url_ attribute of the
_route_ element in inter_webapi.xml and have it autogenerated for us. This way _the invoker_ will be able to determine URL for a service based on class name/method name provided. The _method_ attribute will be dropped because otherwise both nodes who are calling remote services and nodes hosting the services will have to have the inter_webapi.xml installed and caching service call results via
web server (like for services access by GET) maybe dangerous because the most fresh results are expected from service calls.
POST method will always be used.
_resources_ child element of the _route_ element will be dropped as well because when calling a service remotely
we don't need to check privilliges again - requests are initated by our code so we assume they're just. In the end enabling a service
to be called by other remote services will look like this:
_inter_webapi.xml_
```xml
<?xml version="1.0"?>
<services xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Webapi:etc/inter_webapi.xsd">

    <!-- Product Service -->
    <service class="Magento\Catalog\Api\ProductRepositoryInterface" method="save"/>
</services>
```
 
We will have to pass authenticated user information to remote services so we'll add _X-Magento-User-Type_ and _X-Magento-User-Id_
headers to requests for a _UserContextInterface_ instance to be initialized. There's no need for nodes to validate auth information of
incoming
requests since those requests will come from other nodes in a closed network or they will be encrypted. These headers will only be
accepted by endpoints declared in _inter_webapi.xml_ files.
 
The way exceptions thrown by services are serialized right now for RESTful web API needs to be changed for inter-service endpoints.
Exceptions are serialized without their type information provided - this info is crutial for services to communicate with each other so
the type information will be added to the output.
 
 
##### Changes for retries mechanism
The invoker implementation based on RESTful web APIs will offer default retries mechanism, but developers will be able to disable it
by using _di.xml_ (see Invoker API and SPI <<<<here>>>>) for when they would want to delegate this responsibility to a service mash.
 
When default retries mechanism is enabled _X-Magento-Call-Id_ header will be added to inter-service requests to identify calls.
It will be randomly generated. Then if a timeout reached while waiting for a service to respond the invoker will send the same request
again configured number of times with the same call ID. The node receiving requests will store the call ID in cache as a key and
assign an empty value to it. When the node will have the service call result it will store it with the call ID. That way when a
duplicate request comes in if the node can't find the call ID in cache it will execute the requested service method as usual,
if the cache has the call ID that would mean the service method is being executed and we'll have to wait until the original process
finishes it and writes the response to cache, if both call ID and the result are in cache then we just return the result. That way
we will avoid executing duplicate calls when retries mechanism is applied.
 
The timeout and tries limit will be configured via di.xml (see the invoker API and SPI <<here>>).
    
#### Service mashes and balancers
Since we going to contact services via HTTP it is possible for configured service base URLs to actualy lead to service mashes or
balancers - that way the retries and lookup mechanism may be delegated.

#### MQ RPC for gateway
Magento has another way to remotely call services - RPC via a message queue. The problem with it is that right now it can only work
with strings as arguments for service contracts and daemons written in PHP to process queues will show slower results in handling
multiple requests than web servers like Nginx or Apache when using RESTful web API for gateway.

#### Front node
Front node is a Magento installation with Framework and Webapi/GraphQl modules. It will handle actual authentication and authorization and pass the auth info to remote services as well as accepting requests from clients (frontend, 3rd party systems).
 
This is the node that will be available on a public network.
