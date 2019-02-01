### Inter-service communications
This documents describes how exactly services will be communicating in a distributed setup.
#### Service declaration
Services will be declared in Api modules using interfaces. These modules will be installed on all nodes that would
need to communicate with such services.
 
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
The API modules will just contain these declarations describing module
without describing actual implementation.
 
Even services that are not written using Magento (or even PHP) will be declared in this way for nodes using Magento.
 
#### Publicly available Web APIs
To enable publicly available APIs (that frontend and 3rd party systems can use) the _front node_ (BFF) will have to have
_\<Module\>Webapi_ or _\<Module\>Graphql_ modules installed that would contain necessary configurations
(like webapi.xml or schema.graphqls) or handlers (like TypeResolvers for GraphQL etc).
 
#### Real and proxy modules
If a node wants to use services locally then it will just have real modules installed (e.g. "Catalog"), if a node is supposed
to access services remotely it will have proxy modules installed (e.g. "CatalogProxy").
 
Proxy modules will contain services/DTOs implementations and a di.xml file to declare preferences for proxy
implementations to be used.
 
For a service such implementation will look like this:
```php
namespace Magento\ServiceBProxy;

class BManagerProxy implements Magento\ServiceBApi\BManagerInterface
{
    ....
    
    public function save(BDataInterface $data): BDataInterface
    {
        $promise = $this->invoker->invoke(
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
Usually DTOs implementations are models in real modules but we need a class to be initialized when client code communicates with a
remote service and for that simple DTOs will suffice.
 
#### The invoker
The invoker will be a part of Magento\Framework and will be present in every node installation. Invoker's job is to find a remote node's
address based on given interface name and method, send the request, receive response and convert it to either requested service method return type
or an exception if it was thrown by the remote service. It returns a promise to support asynchronous services and because
the requests to remote services will be asynchronous (depends on implementation). More details on the invoker's API and SPI
[here](invoker.md).

#### Possible gateways
__REST__

Pros
* We already have framework that provides support for it
* Established way to call service contracts
* Easy to adopt by developers creating services with tools other then Magento

Cons
* Not suitable for communication between services because it has limitations on the set of primary function calls (read, replace, modify, delete)
* Has large size of the message
* Not all endpoints should be exposed to frontend developers
(like getting a Gift card account by ID, service for sending emails, service for sending a payment)
* Mix of endpoints meant for frontend devs and endpoints for other services would potentially create security problems
* Multiple endpoints for the same service interface/method may exist with different parameters enforced - invoker
would need only one

__gRPC__

Pros
* Specifically designed for communication between services
* Small size of the message
* Support of HTTP 2
* Client code generation support for many programming languages
* Potentially can save on network communication if services would be demons that invokes service application
 
Cons
* Writing servers is not officially supported for PHP, would have to use a hack by 3rd party developers
* Would require developers of services that are not using Magento to adopt this technology
* Harder to customize for our needs like forwarding authenticated user info,
duplicate operation prevention mechanism (request ID) or sending other metadata
* Would require more development time comparing to using existing gateways
* Developers that are not using Magento would have to have a daemon processing requests from Magento instances
* gRPC servers would have to be daemons and PHP doesn't have native async code support which would make daemons processing
requests much slower than with an HTTP server like apache/nginx
(daemon would use 1 core, HTTP servers effectively distribute processes between cores)
* Magento would have to be adopted to be run as a daemon
* only supports 1 dimensional maps (with values of the same type) and one dimensional arrays

__RPC + REST__

Pros
* We already have framework that provides support for it
* Established way to call service contracts
* Easy to adopt by developers creating services with tools other then Magento
* No limitations on the set of primary function calls
* Easy to add metadata between requests and to customize for Magento needs
* Can be easily used with HTTP cache proxies like Varnish

Cons
* Existing REST web API will be used as an RPC so some of the RESTful principles will be disregarded
* Has large size of the message

__GraphQL__

Pros
* Already existing gateway

Cons
* Has complication logic behind it
* We wouldn't be able to utilize GraphQL's ability to strip properties or load multiple entities at once since we would
have no way to know what properties client service wanted because client services wouldn't know whether they're calling
a  remote service or a local one
 
#### Gateway
For actually delivering remote service requests RESTful API will be used - it's an established way to execute requested
class' methods and to
serialize arguments and return values. Existing web API gateway will be used as an RPC to save time on developing new
gateway - there is no need to preserve RESTful principles (like using certain HTTP methods for certain types of operations)
for inter-service communications since requests will be generated automatically and no human developer will be working
with inter APIs explicitly.
 
For sending HTTP requests [Guzzle](https://github.com/guzzle/guzzle) library will be used - it's
a stable and well supported library which is capable of sending concurrent requests and allows wide customization.
Guzzle will be configured to use cUrl (cUrl multi) as a backend. When asynchronous requests are sent via Guzzle it
returns a promise which can easily be converted to Magento's promise.
#### Lookup
The information on how to reach services will be provided in env.php like this:
```php
return [
    'services' => [
        'Magento\Catalog' => ['address' => 'http://catalog.mydomain.com/'],
        'Magento\Checkout' => ['address' => 'https://checkout.mydomain.com/'],
        'Magento\Customer' => ['address' => 'http://192.168.1.1/'],
    ],
];
```
The invoker will look up the information based on provided services' interface names.
 
#### Changes to RESTful Web API mechanism
In services communications we would have to send and receive additional metadata (like forwarding ID of the user who
originally made the request to BFF etc, providing more detailed information about occurred exceptions) so we'd have to
clearly distinguish requests from other services from requests from front-ends and 1st thing to do is to accept
inter-service requests on a different URL - _/rest/inter/*_. A controller will be created similar to
Magento\Webapi\Controller\Rest to process these requests, do specific to service communications validations and to
initiate unique app area.

Since we declare service contracts with interfaces in \*Api modules and in a monolith deployment every module can use
other module's service contracts when /Api/* interfaces are available, remote service contracts will be always available
for other service nodes as well as long as client nodes have _\*Api_ and _\*Proxy_ installed. Remote service contracts
will be hidden from public access by being either deployed in a private network or by signing requests to each other.

When calling the invoker only interface names and methods names will be given (because those are the service declarations)
and it would be hard to determine a manually created URLs for endpoints assigned to requested services. Also, except for
services written using other technologies than Magento, all the requests to services will generated automatically by
invoker. Therefore there is no need to assign URLs to service contracts manually - they can be generated automatically
based on interfaces/methods names.

HTTP methods in RESTful API are used to provide self-documentation for developers writing clients and for browsers to
adjust their behaviour when processing requests to RESTful APIs. They also can be used for caching rules for web
servers/proxies like Varnish. In services communications requests are being generated automatically, they don't
come through browsers and caching rules for servers must be assessed on a case-to-case basis and cache needs to be
carefully invalidated to avoid breaking services' business logic, therefore there is no need in HTTP methods for
services communications and invoker will always send POST requests to remote services. Services will accept any
HTTP method though so developers of services not based on Magento can send requests with methods they feel
suite a requests best.

While URLs for services will be autogenerated and POST always used to make it easier for Magento's invoker it will still
be possible for developers to write services without using Magento (or even PHP) - they would be able to get see URLs
for certain contracts with a CLI utility and while using POST for every request looks odd, functionally it wouldn't matter.
So developers could contact Magento services just like any other microservice with RESTful web API.
  
The way exceptions thrown by services are serialized right now for RESTful web API needs to be changed for inter-service endpoints.
Exceptions are serialized without their type information provided - this info is crutial for services to communicate with each other so
the type information will be added to the output.
 
To make it easier for debuging and to perhaps later to be used for data consistency _X-Magento-Correlation-Id_ header will
be added to requests to remote services with ID generated on BFF before a 1st request to a service. The ID will be
forwarded between services to identify a single operation.  
 
##### Retries mechanism
The invoker implementation based on RESTful web APIs will offer default retries mechanism, but developers will be able to disable it
by using _di.xml_ (see Invoker API and SPI [here](invoker.md)) for when they would want to delegate this responsibility
to a service mash or a balancer. 
The timeout and tries limit will be configured via di.xml (see the invoker API and SPI [here](invoker.md)).
This default implementation may be used by developers who choose to avoid service mashes or for development environments.
 
Regardless of which retries mechanism is applied when a timeout is configured a service may fail to deliver a result
in time but will deliver it eventually, but the invoker (or a service mash) would've sent a duplicating request
by that time. To avoid duplicating operations for such cases _X-Magento-Call-Id_ header
will be added to inter-service requests to identify calls.
It will be randomly generated. Then if a timeout reached while waiting for a service to respond the invoker
(or a service mash) will send the same request
again configured number of times with the same call ID. The service node receiving requests will store the call ID in cache as a key and
assign an empty value to it. When the service node will have the service call result it will store it with the call ID. That way when a
duplicate request comes in if the service node can't find the call ID in cache it will execute the requested service method as usual,
if the cache has the call ID that would mean the service method is being executed and we'll have to wait until the original process
finishes it and writes the response to cache, if both call ID and the result are in cache then we just return the result. That way
we will avoid executing duplicate calls when retries mechanism is applied.

#### Data consistency
Mechanisms to preserve data consistency will described later in another document and can be built on
top of the communications described in this one.
 
    
#### Service mashes and balancers
Since we going to contact services via HTTP it is possible for configured service base URLs to actually lead to service mashes or
balancers - that way the retries and lookup mechanism may be delegated.

#### Deployment
In default .htaccess and nginx.conf.sample files requests to /rest/inter/* will be disabled because it would be dangerous to
allow calling any installed services from outside - only other service nodes must be able to do this. That's why it will
be recommended to have service nodes reside in a private network (or have firewalls allowing only connections
from trusted nodes) and then enabling requests to /rest/inter/* in .htaccess/nginx.conf.
 
For cases when a service node must be publicly accessible (like BFF having some services deployed locally) developers
would have to create SSL client-side certificates and then create a separate virtual host processing requests to /rest/inter/*
that would require all client service nodes to use this certificates to sign their requests.
 
This measures will prevent anyone other than service nodes using service comm gateway.
 
To notify invoker to send requests signed an option will exist in services configuration:
```php
return [
    'services' => [
        'Magento\Catalog' => [
            'address' => 'http://catalog.mydomain.com/',
            'certificate' => ['file' => '../cert.pem', 'password' => 'qwerty123']
        ],
    ],
];
```

#### BFF
BFF is a Magento installation with Framework, *Api, *Proxy and *Webapi/*GraphQl modules.
 
This is the node that will be available on a public network.

#### Flow
Entities:
* BFF
  * Modules installed:
    * Magento\Framework
    * Magento\Authorization
    * CatalogApi
    * CatalogWebApi
    * CatalogGraphQl
    * CatalogProxy
    * QuoteApi
    * QuoteWebApi
    * QuoteGraphQl
    * QuoteProxy
  * env.php has base URLs provided for all modules (Magento\Catalog and Magento\Quote)
* Catalog service
  * Modules installed:
    * Magento\Framework
    * CatalogApi
    * Catalog
    * CatalogInterApi
    * QuoteApi
    * QuoteProxy
  * env.php has base URL for Magento\Quote
* Quote service
  * Modules installed:
    * Magento\Framework
    * QuoteApi
    * Quote
    * QuoteInterApi
    * CatalogApi
    * CatalogProxy
  * env.php has base URL for Magento\Catalog
  
  
 What will happen:
  
_BFF_
* Client sends HTTP request to POST <magento-bff>/rest/V1/carts/<id>/order
  with body containing payment method data and _Authorization_ header containing previously obtained admin token.
* Request is being processed as usual when it comes to REST web API:
  * it finds corresponding service declaration (in _QuoteWebapi_ module) which points to the method call -
  CartManagementInterface::placeOrder(); CartManagementProxy returned from
  the _QuoteProxy_ module
  * the endpoint declaration also contain ACL resources information - _Magento_Cart::manage_ permission
  is required; Current user's permissions are being validated by ACL service.
  * deserializes input into string and PaymentInterface - string and PaymentProxy is returned from the _QuoteProxy_ module
  * calls CartManagementInterface::placeOrder() with string and PaymentInterface instance as arguments
* _QuoteProxy_ has di.xml which states that CartManagementProxy is the preference for CartManagementInterface
and PaymentProxy is the preference for PaymentInterface
* CartManagementInterface::placeOrder() calls the InvokerInterface and passes the service's interface name, method and the argument
* Invoker looks at env.php to find Magento\Quote's URL (something like https://quote.mydomain.com/ or http://182.1.1.2)
* Invoker serializes the argument as the argument type provided in CartManagementInterface::placeOrder() declaration
* Invoker gets user type and user ID from the UserContextInterface instance
* Invoker composes URL to access inter-service endpoint on the _Quote_ service node
like __http://quote.mydomain.com/rest/inter/quote/cart-management/place-order__
* Invoker finds no active _correlation ID_ so it generates a random string and adds it to the request to be sent to the
_Quote_ service
* Invoker generates a _request ID_ for this remote call
* Invoker sends request to _Quote_ service node containing serialized ProductInterface argument, user type, user ID,
correlation ID, request ID
and returns a promise that will be fulfilled when the request finishes and return value is unserialized
* the request is sent using Guzzle library and it's sent asynchronously (concurrently with other requests)
 
_quote service_
* Receives HTTP request from the BFF, let's say on https://quote.mydomain.com/rest/inter/quote/cart-management/place-order
* CartManagementInterface::placeOrder() is matched because URL generation rules are the same on all service nodes and managed by Framework
* Checks inter_webapi.xml in _QuoteInterApi_ module to find if CartManagementInterface::placeOrder() is available
* Reads _request ID_, tries storage (a cache) to see if an operation with such ID was already started or even finished -
no record, this request is not a duplicate, continue processing
* Reads user type and user ID information from the request and creates a UserContextInterface instance
* Deserializes body into a string and the PaymentInterface instance (this node also has _QuoteApi_ modules and gets the service's arguments it provides)
* Magento\Quote\Model\Quote\Payment is actually used as the argument because this node has _Quote_ module installed
* Magento\Quote\Model\QuoteManagement is used as the service instance because this node has _Quote_ module with it's di.xml
and no _QuoteProxy_ module
* QuoteManagement::placeOrder() is executed
* QuoteManagement::placeOrder() uses StockManagerInterface::decrease() from _CatalogApi_ module
* this node has only _CatalogProxy_ module so an instance of CatalogProxy\StockManagerProxy is used
* it sends a request to the _catalog service_ using the invoker with the _correlation ID_ received from BFF
and a new _request ID_ generated for this operation
* the result is used by the QuoteManagement::placeOrder() method
* QuoteManagement::placeOrder() finishes and returns order ID
* RESTful API mechanism serializes string (order ID)
* HTTP response is generated
 
_BFF_
* Invoker has provided a callback to the promise Guzzle returned when asynchronous HTTP request was made
* the callback unserializes return value received from the service call, in this case into a string
* returned order ID is passed as resolve value to the promise the invoker returned
* RESTful API mechanism takes the value and serializes it to return to the client
* HTTP response is generated and returned to the client who made request to mydomain.com/rest/V1/carts/<id>/order
