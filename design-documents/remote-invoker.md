## Remote code invoker
### What for?
Remotely calling services in proxy versions of modules
### Overview
This documents describes remote invoking mechanism, it's functional requirements, the module's API and SPI.
It also describes a recommended RPC-style networking gateway to call services remotely, but there can be
multiple _Remote Code Invoker_ implementations based on existing Web APIs, Message Queue RPC API or any other.
### How are remote services are going to look like with Remote Code Invoker
\<Module name\>Proxy module will contain \<Module name\>API module's implementations that would look like this:
```php
class ServiceProxy implements \Magento\ModuleNameApi\ServiceInterface
{

....

    public function doStuff(\Magento\ModuleNameApi\Data\DTOInterface $dto): \Magento\ModuleNameApi\Data\ResultDTOInterface
    {
        $remoteResult = null;
        $promise = $this->invoker->invoke(
            'catalog',
            \Magento\ModuleNameApi\ServiceInterface::class,
            'doStuff',
            [$dto]
        );
        $promise->then(function (\Magento\ModuleNameApi\Data\ResultDTOInterface $result) use (&$remoteResult) {
            $remoteResult = $result;
        });
        $promise->wait();
        
        return $remoteResult;
    }

....

}
``` 
 
di.xml in \<Module name\>Proxy module will specify that ServiceProxy
is the preference for \Magento\ModuleNameApi\ServiceInterface.

So when a local service uses \<Module name\>API's service it will not be aware whether the service is actually
remote and it won't affect how we write code.
Example:
```php
class LocalServiceA implements ServiceAInterface
{

    /**
     * @var \Magento\BModuleApi\ServiceBInterface
     */
    private $serviceB;

....

    public function doAStuff(): void
    {
        ....

        $stuff = $this->serviceB->doBStuff($someVar);
        
        ....
    }

....

}
```

Depending on whether _BModule_ or _BModuleProxy_ is deployed on the local node _ServiceBInterface_ will either
work locally or remotely.
### Why not just use cURL and existing Web APIs
We need a mechanism, an abstraction around an actual network interactions that would provide a retries mechanism,
a standard to pass local call's arguments, a request signing mechanism, authentication mechanism which would allow
to perform authentication only once on the node that accepts user requests and not on each node, data consistency
mechanism etc.
_([see explanation](#need-for-an-abstraction-around-network-calls))_.
### Functional requirements
* Invoker's code is a part of Magento\Framework namespace
* Invoker can call classes and method that were specifically enabled for remote calls via a config
* Invoker provides a way for services to written as usual, not knowing whether other modules' services are remote or not
* Multiple remote sources can be configured and assigned unique IDs
* Each HTTP request to remote sources is signed using unique source's key, timestamp and request's attributes
* Each remote call must be given unique ID to be used by retries mechanism
* Timeout limit and retries count can be configured globally and for each call
* Serialized arguments string includes arguments instances' classes information for proper
deserialization _([see explanation](#inability-to-just-use-interfaces-for-arguments-and-return-types))_
(serializer employed for RESTful gateway will probably be reused)
* Serialized remote method return values string must contain type information for
proper deserialization by the caller
* Remote call response can provide either serialized method call result or serialized exception to be
rethrow in order to emulate local method call
* Remote calls can be done in a synchronous manner to ease remote invoking for
existing services _([see explanation](#using-synchronous-remote-invoking-in-existing-services))_
and in an asynchronous manner for new services to be able to interact with other services with remote invoking in mind
* Each Magento instance that can be used for remote calls (_Magento server_)
has it's own unique key for signing requests
* Remote calls can be disabled in a Magento server
* Authenticated user's information (ID and type) is passed from caller
to a Magento server to emulate user context
* If a network error occurs during a remote invoking (timeout, connection lost etc.) a configured number of retry
requests containing remote call's ID will be sent, the call ID will keep _Magento server_ from initiating a
new service execution process
* Magento server stores remote calls information that can be found by calls' IDs
* Remote call information contains call result or empty result if the call is not finished yet
* Remote call information can be retrieved multiple times by remote callers as a part of retries mechanism
to prevent multiple call execution, also if remote callers were to generated meaningful call IDs for some cases
this mechanism can be used to ensure idempotency
* Remote call information expires after some time on Magento servers

### API
##### Remote call response
Result of executing the call
```php
interface InvokeResponseInterface {
    /**
     * @return mixed Value that remote service returned.
     */
    public function getResult();
    
    /**
     * Exception that remote service has thrown, if any.
     *
     * @return \Throwable|null
     */
    public function getException(): ?\Throwable
}
```
##### Remote Invoker
Used to make a call to a Magento server
```php
interface InvokerInterface {
    /**
     * @param string $sourceId
     * @param string $class
     * @param string $method
     * @param array $arguments Keys - argument names.
     * @param int|null $timeout
     * @param int|null $retries
     * @return Promise Will provide InvokeResponseInterface when resolved,
     *                 RemoteInvokeException on reject when networking fails,
     *                 has "wait" method to use in synchronous way.
     */
    public function invoke(
        string $sourceId,
        string $class,
        string $method,
        array $arguments,
        ?int $timeout = null,
        ?int $retries = null
     ): Promise;
}
```
##### 
##### Remote sources information
_Magento server_ information
```php
interface SourceInterface {
    public function getId(): string;

    public function getUrl(): string;
    
    public function getKey(): string;
}
```
##### Remote sources repository
Find sources' information
```php
interface SourceRepositoryInteface {
    public function get(string $id): SourceInterface;
    
    public function getAll(): SourceInterface[];
}
```
### SPI
##### Remote request
Contains remote call data
```php
interface InvokeRequestInterface {
    public function getCallId(): string;
    
    public function getSource(): SourceInterface;
    
    public function getClass(): string;
    
    public function getMethod(): string;
    
    public function getArguments(): array;
    
    public function getUserContext(): UserContextInterface;
}
```
##### Request generator
Create request based on the remote call data
```php
interface RequestGeneratorInterface {
    public function generate(
        InvokeRequestInterface $invokeRequest
    ): RequestInterface
}
```
##### Request signer
Sign request before sending
```php
interface RequestSignerInterface {
    public function sign(RequestInterface $request, InvokeRequestInterface $invokeRequest): RequestInterface;
}
```
##### Transport
Send request to a remote source.
Returns chainable and waitable promise.
```php
interface Transport {
    public function send(RequestInterface $request): Promise;
}
```
##### Remote call request validator
Validates whether request's signature is correct
```php
interface RequestValidatorInterface {
    /**
     * @throws ValidationException
     */
    public function validate(RequestInterface $request): void;
}
```
##### Remote call request reader
Read incoming requests and extract remote call request information
```php
interface RequestReaderInterface {
    public function read(RequestInterface $request): InvokeRequestInterface;
}
```
##### Remote call info storage
Store remote call results for retries mechanism
```php
interface InvokeResponseRepositoryInterface {
    public function createStarted(string $callId, int $timeout): void;
    
    public function saveResponse(string $callId, InvokeResponseInterface $response): void;
    
    /**
     * @throws NoSuchEntityException When there were no such call registered.
     * @return InvokeResponseInterface|null Null if call is not yet finished.
     */
    public function get(string $callId): ?InvokeResponseInterface;
}
```
##### Remote call processor
Perform remote call
```php
interface LocalInvokerInterface {
    public function invoke(InvokeRequestInterface $request): InvokeResponseInterface;
}
```
##### Response generator
Transform invoke response to a regular response
```php
interface ResponseGeneratorInterface {
    public function generate(string $callId, InvokeResponseInterface $response): ResponseInterface;
}
```


### Networking gateway

_Remote code invoker_ is an abstraction, it's not enforcing how requests and responses are actually sent and parsed.

#### RPC-style gateway

HTTP protocol will be used. A controller will be added to process a single URL like _magento-base-url_/invoker.
Requests' body will contain requested class name, method name, arguments and it all encoded in json.
Requests' headers will contain information such as authenticated user ID, call ID, issued timestamp, signature.
Response body will contain serialized service call result or a serialized thrown exception encoded in json.

#### Why RESTful web API should not be used as the network gateway for the invoker
* Not all service contracts are exposed as web API and nor should they be (for security and workflow reasons)
_([see explanation](#exposing-internal-logic-as-an-external-web-api))_
* Web API methods have certain ACL resources assigned to them so the ACL checks would be
performed every time a proxy service calls remote service unlike when a service
is called locally _([see explanation](#redundant-acl-validations-when-using-existing-web-api))_
* Proxy services would have to generate auth information of already authorized users (tokens etc) to access REST/SOAP
services which is a redundant process, also some of those REST/SOAP endpoints would have limitations in
place for certain type of users which would be useless in case of remote execution
(like requiring an admin user) _([see explanation](#redundant-authorization-validation-when-using-existing-web-api))_
* Other application state spoofing may be required for seamless remote execution of services
* RESTful Web API provides access to service contracts by assigning a unique human-readable
HTTP method/URL pair for external systems - we don't need that when writing a proxy service since we
would always already have class and method's names, the only other thing existing RESTful Web API mechanism
provides is it's serializer and it will be reused in the _Remote Code Invoker_
* When an exception occurs during a service call via RESTful API the response does not contain
all exception information which is important for a seamless Proxy modules integration
(especially exceptions' types)

#### Why webhook based network gateway shouldn't be used as the network gateway for the invoker
Being able to send an invoking request to a _Magento server_ and to drop the connection immediately for
the _Magento server_ to send us the result later is tempting, but given way presents such problems:
* Webhooks are meant for situations when we can afford to receive a remote operation's results anytime in
the future, with remote code invoking that is not the case - we need the results ASAP
* We would need _Magento clients_ to provide web endpoint to receive results and store them in a DB - that's
an additional work for _Magento clients_ which would be avoided if we would just receive results as responses
for HTML requests
* During the remote invoking we would have to check the DB where the results are stored constantly with
an interval, make the interval short - too many queries to the DB, too decent - we would not be getting
the results ASAP

#### Why existing MQ based RPC gateway shouldn't be used
* it works in a synchronous way anyway, the MQ service is needlessly employed here
* currently it only supports string argument and will need to be updated heavily anyway - no time saved
comparing to writing a proper HTTP gateway based RPC designed for internal services communications

#### Invoker based on RESTful web API
To differentiate outer web APIs and inner web APIs their configuration must differ, there are 2 ways to do it:
  * add _inner_ attribute to _route_ (webapi.xsd) element which when _true_ will mean that inner call mode would be
  enabled for Magento when the API is used (requests would have to be signed, call ID provided etc.) and
  some additional sub-elements/attributes maybe used/omitted (like not having to use _url_ attribute for such APIs)
  * add another .xsd and new config files to modules to describe inner web APIs
 
There is another way - not to differentiate inner and outer web APIs, that may be proven a bit more confusing
but it is a viable option. In this case Magento will know it is accessed as a micro-service by requests providing
call IDs. Also we should add a possibility to omit _url_ attribute of the _route_ element, when it's omitted
the URL for such endpoint will be generated based on class name and method name - that is useful for
inner endpoints since we don't care for how URLs look.

### Explanations
#### Need for an abstraction around network calls
Say we're writing a Proxy version of a ProductRepositoryInterface service using HTTP client and RESTful API.
```php
class ProductRepoProxy implements ProductRepositoryInterface
{

....

    public function save(ProductInterface $product): ProductInterface
    {
        $baseUrl = $this->config->get('product-node-url');
        $request = new HttpRequest($baseUrl .'/V1/product');
        $request->setHeader('X-Magento-User-Type', $this->userContext->getUserType());
        $request->setHeader('X-Magento-User-Id', $this->userContext->getUserId());
        $request->setBody(
            $this->jsonSerializer->serialize(
                ['product' => $this->objectSerializer->toArray($product, ProductInterface::class)]
            )
        );
        $productCreated = null;
        $this->transport->send($request)->then(function ($httpResponse) use (&$productCreated) {
            $data = json_decode($httpResponse->getBody());
            $productCreated = $this->objectSerializer->fromArray(ProductInterface::class, $data);
        })->error(function ($exception) use ($product) {
            if ($exception instanceof NetworkFailException) {
                //Retrying
                return $this->save($product);
            }
        })->wait();
        
        return $productCreated;
    }
    
    public function delete(string $id): void
    {
        //Same stuff: using config, userContext, serializers etc.
        ....
    }

....

}
```

Clearly sending such requests involves a lot of logic and dependencies on other classes,
and an abstraction (InvokerInterface) which will provide a simple way to pass arguments and remote source data
to execute a remote service request.

#### Exposing internal logic as an external Web API
Let's say we have a hypothetical situation: customer places an order _(Checkout)_, but for each item ordered we have to update "ordered" count for related products _(Catalog)_
```php
class OrderManagement implements OrderManagementInterface
{
 
....
 
public function place(OrderInterface $order): void
{
 
....
 
    foreach ($order->getItems() as $product) {
        //Calling a service from another domain (Catalog), maybe local, maybe remote
        $this->productRepository->incrOrdederedCount($product->getId());
    }
 
....
 
}
```
If the remote call to _ProductRepositoryInterface_ service was done via RESTful (or GraphQL) web API the _incrOrdederedCount_ would have
to be exposed as Web API and could be requested via HTTP by anyone and independently from context which we cannot afford since we can't just increase the counter without an order happening so only another service responsible for orders may call _incrOrdederedCount_.
 
If done via the Invoker the _incrOrdederedCount_ wouldn't be exposed to be requested by HTTP clients via web API and wouldn't show up on
RESTful endpoints list.
#### Redundant ACL validations when using existing Web API
We have a hypothetical situation: admin user with access to orders _(Checkout)_, but not to products _(Catalog)_ changes an order and removes a product from the items list:
```php
class OrderRepository extends OrderRepositoryInterface
{
 
....
 
    public function save(OrderInterface $order): void
    {
        ....
        
        foreach ($removedItems as $product)
        {
            //Decreasing ordered count since the product is not ordered anymore, product repository is a Catalog service
            $this->productRepository->decrOrderedCount($product->getId());
        }
    }
 
....
 
}
```
If the remote call to _ProductRepositoryInterface::decrOrderedCount_ was done via RESTful web API the exposed endpoint would have
ACL configured for it so it requires _Catalog edit access_ from admins, but our current admin only has access to _orders_ and the call would fail. However if the Catalog domain modules were not remote and the _ProductRepositoryInterface::decrOrderedCount_ call was made then no ACL validation would occur and we would save the order successsfully.
 
With Invoker such ACL validations wouldn't occur and the remote call would be completed just as if it was local.
#### Redundant authorization validation when using existing Web API 
We have a hypothetical situation: customer updates their main address _(Customer)_ and we need to update their not-shipped orders to be
shipped to their main address _(Checkout)_
```php
class AddressManagement implements AddressManagementInterface
{
 
....
 
    public function save(AddressInterface $address): void
    {
        
        ....
        
        //Receiving customer's order in progress
        foreach ($this->orderRepository->getList($searchCriteria) as $order) {
            //Setting the new address
            $order->setShipmentAddress($address);
            //Saving the order, order repository maybe local or remote
            $this->orderRepository->save($order);
        }
    }
 
....
 
}
```
If the remote call to _OrderRepositoryInterface::save_ was done via RESTful web API we would pass current user's token to the endpoint
but, obviously, PUT /V1/order/:id would require admin access and the user who changed their address is just a customer. And certainly we can't just pick a 1st random admin user and send the HTTP request to the RESTful endpoint from their name.
 
With Invoker we would emulate current customer user, but _OrderRepositoryInterface::save_ doesn't actually have any requirement for current user to be admin and would be executed just fine.
#### Inability to just use interfaces for arguments, return types and exceptions
Imagine we have a situation: an order is placed by an admin for a customer _(Checkout)_ and we need to add the shipment address of the order to the customer's address book _(Customer)_
```php
class OrderManagement implements OrderManagementInterface
{

....

    public function place(OrderInterface $order): void
    {
        ....
        
        //WTF is $customerAddress instance of?
        $customerAddress = $this->customerAddressInterfaceFactory->create();
        $customerAddress->setStreet($orderAddress->getStreet());
        ....
        //customerAddressRepository is a proxy
        //WTF is $createdAddress actually instance of?
        /** @var CustomerAddressInterface $createdAddress */
        $createdAddress = $this->customerAddressRepository->save($customerAddress);
        
        ....
    }

....

}

/**
 * Inside Customer domain.
 */
interface AddressRepositoryInterface
{

....

    public function save(Data\AddressInterface $address): Data\AddressInterface;

....

}
```
There is a question - what would _$this->customerAddressFactory->create();_ return?
It cannot be _Magento\Customer\Model\Address_ because Checkout domain does not have that class, but there must be a class since we
cannot just initialize _AddressInterface_. And it has to be class that is both present in _Checkout Magento installation_
and in _Customer Magento installation_ so it can be sent and unserialized when received. Both _Checkout_ and _Customer_ installation
have _CustomerApiModule_ enabled in them so the class must reside in that module. I propose we create actual implementations alongside
* \Api\Data\ * Interface interfaces, those would be dumb DTOs with just getters and related properties and they would be used for such remote (and local) calls.
 
Same goes for return types - _$this->customerAddressRepository->save($customerAddress)_ must return something, and it cannot be just
_AddressInterface_, must be a class.
 
Also it is of upmost importance to preserve exceptions instances' types during serialization, consider this:

```php
class OrderManagement implements OrderManagementInterface
{

....

    public function place(OrderInterface $order): void
    {
        ....
        
        try {
            $this->customerAddresses->create($this->convertAddress($order->getShipingAddress()));
        } catch (AlreadyExistsException $exception) {
            $order->getShipingAddress()->setCustomerAddressId($exception->getExistingId());
        }
    }
}
```

If the _customerAddresses_ is a remote service and we just deserialize the exception it has thrown
as a \Throwable the code processes certain types of exception would just fail unlike when the _customerAddresses_
service is local.

Also, also consider this situation:

```php
class LocalServiceA implements ServiceAInterface
{

....

    public function doStuff(): void
    {
        ....
        
        $this->serviceB->doOtherStuff($entityB);
    }

....

}


class LocalServiceB implements ServiceBInterface
{

....

    public function doOtherStuff(DTOInterface $dtoB): void
    {
        ....
        
        if ($dtoB instanceof DTOWithAdditionalFieldsInterface) {
            $this->doAdditionalStuff($dtoB);
        }
        
        ....
    }
}
```

When ServiceBInterface is used locally this will work fine, but if ServiceB is a remotely deployed service
and _$dtoB_ parameter is serialized as _DTOInterface_ only the code will fail so we need to preserve arguments
(and return values and exceptions) classes for remote services to work as expected.

#### Using synchronous remote invoking in existing services
Say we're placing an order and need to check whether the products are available
```php
class OrderManagement implements OrderManagementInterface
{

....

    public function place(OrderInterface $order): void
    {
        ....
        
        foreach ($order->getItems() as $item)
        {
            if (!$this->inventoryManagement->isAvailable($item->getProductId(), $item->getQty())) {
                throw new LocalizedException('Product is not available');
            }
        }
        
        ....
    }

....

}
```
Now we trying to run this code on a distributed Magento deployment so the _inventoryManagement_ is a remote
service. We cannot change the _InventoryManagementInterface::isAvailable_ return type to be a promise
(because of backward-compatibility)
and we don't want to rewrite OrderManagement to consider _isAvailable_ being asynchronous
(because rewriting all existing services in order for them to use all other modulus' services in an async way
is way too big of a task) so _isAvailable_ must keep returning boolean.
 
So the promise the invoker returns must have "wait" method so we can wait for result to be in and then return it
```php
class InventoryManagementProxy implements InventoryManagementInterface
{

....

    public function isAvailable(string $productId, int $qty): bool
    {
        $isIt = null;
        $this->invoker->invoke('inventory', InventoryManagementInterface::class, 'isAvailable', [$productId, $qty])
            ->then(function (InvokeResponseInterface $response) use (&$isIt) { $isIt = $response->getResult(); })
            ->wait();
        //$isIt is definetly set because we've waited for when the promise is fulfilled.
        //If we didn't have the "wait" method $isIt would probably always return "null"
        return $isIt;
    }

....

}
```
