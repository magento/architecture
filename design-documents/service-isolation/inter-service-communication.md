### Inter-service communications
This documents describes how exactly services will be communicating in a distributed setup.
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
        
        if ($result->getException()) {
            throw $result->getException();
        }
        
        return $result->getResult();
    }
    
    ....
}
```

Proxy services will use _InvokerInterface_ to call remote services. It will return a promise which will pass a service execution result
to _then_ callback and an exception to _otherwise_ callback if something during the remote call went wrong (like a network error). The
service execution result may contain the return value of a service or an exception it has thrown. The promise that invoker returns may
be used in both synchronous and asynchronous ways to be able to work with both types of services. More details about the promise
[here](../promises.md).
 
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
address based on given class name and method, send the request, receive response and convert it to either requested service method return type
or an exception if it was thrown by the remote service. It returns a promise to support asynchronous services and because
the requests to remote services will be asynchronous (depends on implementation). More details on the invokers API and SPI
[here](invoker.md).
 
#### Gateway
For actualy delivering remote service requests RESTful API will be used - it's an established way to execute requested methods and to
serialize arguments and return values. For sending HTTP requests [Guzzle](https://github.com/guzzle/guzzle) library will be used - it's
a stable and well supported library which is capable of sending concurrent requests and allows wide customization.
Guzzle will be configured to use cUrl (cUrl multi) as a backend. When asynchronous requests are sent via Guzzle it returns a promise which can easily
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
communications so to distinguish regular endpoints and endpoints meant for inter-service communications we will declare them in a
seperate yet similar xml files - _inter_webapi.xml_. Since we declare services by declaring PHP interfaces there is no reason to provide
human-readable URLs for endpoints meant for inter-service communications, thus we'll make it possible to omit _url_ attribute of the
_route_ element in inter_webapi.xml and have it autogenerated for us. This way _the invoker_ will be able to determine URL
for a service based on class name/method name provided. The _method_ attribute will be dropped because otherwise both nodes
who are calling remote services and nodes hosting the services will have to have the inter_webapi.xml installed.
Also caching service call results via
web server (like for services accessed by GET) may be dangerous because the most fresh results are expected from service calls.
POST method will always be used when sending HTTP requests to remote services.
_resources_ child element of the _route_ element will be dropped as well because when calling a service remotely
we don't need to check privileges again - requests are initiated by our code so we assume they're just. In the end enabling a service
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
 
_inter_webapi.xml_ files will be contained in separate <ModuleName>InterApi modules and will have to be installed
on nodes accepting remote inter-service requests.
 
##### Changes for retries mechanism
The invoker implementation based on RESTful web APIs will offer default retries mechanism, but developers will be able to disable it
by using _di.xml_ (see Invoker API and SPI [here](invoker.md)) for when they would want to delegate this responsibility to a service mash.
 
When default retries mechanism is enabled _X-Magento-Call-Id_ header will be added to inter-service requests to identify calls.
It will be randomly generated. Then if a timeout reached while waiting for a service to respond the invoker will send the same request
again configured number of times with the same call ID. The node receiving requests will store the call ID in cache as a key and
assign an empty value to it. When the node will have the service call result it will store it with the call ID. That way when a
duplicate request comes in if the node can't find the call ID in cache it will execute the requested service method as usual,
if the cache has the call ID that would mean the service method is being executed and we'll have to wait until the original process
finishes it and writes the response to cache, if both call ID and the result are in cache then we just return the result. That way
we will avoid executing duplicate calls when retries mechanism is applied.
 
The timeout and tries limit will be configured via di.xml (see the invoker API and SPI [here](invoker.md)).
    
#### Service mashes and balancers
Since we going to contact services via HTTP it is possible for configured service base URLs to actually lead to service mashes or
balancers - that way the retries and lookup mechanism may be delegated.

#### MQ RPC for gateway
Magento has another way to remotely call services - RPC via a message queue. The problem with it is that right now it can only work
with strings as arguments for service contracts and daemons written in PHP to process queues will show slower results in handling
multiple requests than web servers like Nginx or Apache when using RESTful web API for gateway.

#### Front node
Front node is a Magento installation with Framework and Webapi/GraphQl modules.
It will handle actual authentication and authorization of incoming requests and pass the auth info to remote services
as well as accepting requests from clients (frontend, 3rd party systems).
 
This is the node that will be available on a public network.

#### Flow
Entities:
* Front node
  * Modules installed:
    * Magento\Framework
    * Magento\Authorization (implementation, APIs for auth are moved to Framework)
    * CatalogApi
    * CatalogWebApi
    * CatalogGraphQl
    * CatalogProxy
    * CustomerApi
    * CustomerWebApi
    * CustomerGraphQl
    * CustomerProxy
  * env.php has base URLs provided for all modules (Magento\Catalog and Magento\Customer)
* Catalog node
  * Modules installed:
    * Magento\Framework
    * CatalogApi
    * Catalog
    * CatalogInterApi
    * CustomerApi
    * CustomerProxy
  * env.php has base URL for Magento\Customer
* Customer node
  * Modules installed:
    * Magento\Framework
    * CustomerApi
    * Customer
    * CustomerInterApi
    * CatalogApi
    * CatalogProxy
  * env.php has base URL for Magento\Catalog
  
  
 What will happen:
  
_front node_
* Client sends HTTP request to POST <magento-front-node>/rest/V1/product
  with body containing new product data and _Authorization_ containing previously obtained admin token.
* Request is being processed as usual when it comes to REST web API:
  * it finds corresponding service - ProductRepositoryInterface::save(); ProductRepositoryProxy returned from
  the _CatalogProxy_ module
  * deserializes input into ProductInterface - ProductProxy is returned from the _CatalogProxy_ module
  * calls ProductRepositoryInterface::save() with ProductInterface instance as the 1st argument
* _CatalogProxy_ has di.xml which states that ProductRepositoryProxy is the preference for ProductRepositoryInterface
and ProductProxy is the preference for ProductInterface
* ProductRepositoryProxy::save() calls the InvokerInterface and passes the service's class name, method and the argument
* Invoker looks at env.php to find Magento\Catalog's URL (smth like https://catalog.mydomain.com/ or http://182.1.1.2)
* Invoker serializes the argument as the argument type provided in ProductRepositoryInterface::save() declaration
* Invoker gets user type and user ID from the UserContextInterface instance
* Invoker composes URL to access inter-service endpoint on the _Catalog_ node like __/rest/inter/catalog/product-repository/save__
* Invoker sends request to _Catalog_ node containing serialized ProductInterface argument, user type, user ID
  and returns a promise that will be fulfilled when the request finishes and return value is unserialized
 
_catalog node_
* Receives HTTP request from the front node, let's say on https://catalog.mydomain.com/rest/inter/catalog/product-repository/save
* ProductRepositoryInterface::save() is matched because URL generation rules are the same on all nodes an managed by Framework
* Checks inter_webapi.xml in _CatalogInterApi_ module to find if ProductRepositoryInterface::save() is available
* Reads user type and user ID information from the request and creates a UserContextInterface instance
* Deserializes body into the ProductInterface instance (this node also has _CatalogApi_ modules and gets the service's arguments it provides)
* Catalog\Model\Product is actually used as the argument because this node has _Catalog_ module installed
* Catalog\Model\ProductRepository is used as the service instance because this node has _Catalog_ module with it's di.xml
and no _CatalogProxy_ module
* ProductRepository::save() is executed
* ProductRepository::save() uses CustomerProductManagerInterface::process from _CustomerApi_ module
* this node has only _CustomerProxy_ module so an instance of CustomerProxy\CustomerProductManagerProxy is used
* it sends a request to the _customer node_ using the invoker
* the result is used by the ProductRepository::save() method
* ProductRepository::save() finishes and returns persisted Model\Product instance
* RESTful API mechanism serializes Model\Product to CatalogApi\Data\ProductInterface since CatalogApi\ProductRepositoryInterface::save()
states that the return type is ProductInterface
* HTTP response is generated
 
_front node_
* Invoker has provided a callback to the promise Guzzle returned when asynchronous HTTP request was made
* the callback unserializes return value received from the service call, in this case into ProductInterface
* ProductProxy is actually created because that's the preference provided in the _CatalogProxy_ module this node has
* initiated ProductProxy is passed as resolve value to the promise the invoker returned
* RESTful API mechanism takes the value and serializes it to return to the client
* HTTP response is generated and returned to the client who made request to mydomain.com/rest/V1/product

   