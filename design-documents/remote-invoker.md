## Remote code invoker
### What for?
Remotely calling services in proxy versions of modules
### Why not just use REST/SOAP?
* Not all service contracts are exposed as web API and nor should they be (for security and workflow reasons)
_(see explaination #1)_
* Web API methods have certain ACL resources assigned to them so the ACL checks would be
performed every time a proxy service calls remote service unlike when a service
is called locally _(see explaination #2)_
* Proxy services would have to generate auth information of already authorized users (tokens etc) to access REST/SOAP
services which is a redundant process, also some of those REST/SOAP endpoints would have limitations in
place for certain type of users which would be useless in case of remote execution
(like requiring an admin user) _(see explaination #3)_
* Other application state spoofing may be required for seamless remote execution of services
### Functional requirements
* Invoker's code is a part of Magento\Framework namespace
* Invoker can call classes and method that were specifically enabled for remote calls via a config
* Multiple remote sources can be configured and assigned unique IDs
* Each HTTP request to remote sources is signed using unique source's key, timestamp and request's attributes
* Each remote call must be given unique ID to be used by retries mechanism
* Timeout limit and retries count can be configured globally and for each call
* Serialized arguments string includes arguments instances' classes information for proper deserialization _(see explanation #4)_
* Serialized remote method return values string must contain type information for
proper deserialization by the caller
* Remote call response can provide either serialized method call result or serialized exception to be
rethrow in order to emulate local method call
* Remote calls are done in a synchronous manner because in order for
to be able to utilize the possibility to make asynchronous remote calls the client code would have to know
that certain services can be proxies and that would complicate introduction of distributed Magento and
make monolith and distributed deploy modes coexistence harder _see explaination #5)_
* Each Magento instance that can be used for remote calls (_Magento server_)
has it's own unique key for signing requests
* Remote calls can be disabled in a Magento server
* Authenticated user's information (ID and type) is passed from caller
to a Magento server to emulate user context
* If timeout is reached while waiting for a Magento server to respond a configured number of retry
requests containing remote call's ID will be sent
* Magento server stores remote calls information that can be found by calls' IDs
* Remote call information contains call result or empty result if the call is not finished yet
* Remote call information can be retrieved multiple times by remote callers as a part of retries mechanism
to prevent multiple call execution, also if remote callers were to generated meaningful call IDs for some cases
this mechanism can be used to ensure idempotency
* Remote call information expires after some time on Magento servers
### API
##### Remote Invoker
Used to make a call to a Magento server
```
interface InvokerInterface {
    /**
     * @param string $sourceId
     * @param string $class
     * @param string $method
     * @param array $arguments Keys - argument names.
     * @param int|null $timeout
     * @param int|null $retries
     * @return mixed Method call result.
     * @throws RemoteInvokeException When remote call failed.
     * @throws \Throwable Exception thrown in the remote method.
     */
    public function invoke(
        string $sourceId,
        string $class,
        string $method,
        array $arguments,
        ?int $timeout = null,
        ?int $retries = null
     );
}
```
##### Remote sources information
A Magento server data
```
interface SourceInterface {
    public function getId(): string;

    public function getUrl(): string;
    
    public function getKey(): string;
}
```
##### Remote sources repository
Find sources' information
```
interface SourceRepositoryInteface {
    public function get(string $id): SourceInterface;
    
    public function getAll(): SourceInterface[];
}
```
### SPI
##### Remote request
Contains remote call data
```
interface RevokeRequestInterface {
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
```
interface RequestGeneratorInterface {
    public function generate(
        InvokeRequestInterface $invokeRequest
    ): RequestInterface
}
```
##### Request signer
Sign request before sending
```
interface RequestSignerInterface {
    public function sign(RequestInterface $request, InvokeRequestInterface $invokeRequest): RequestInterface;
}
```
##### Transport
Send request to remote source.
```
interface Transport {
    public function send(RequestInterface $request): void;
}
```
##### Remote call request validator
Validates whether request signature is correct
```
interface RequestValidatorInterface {
    /**
     * @throws ValidationException
     */
    public function validate(RequestInterface $request): void;
}
```
##### Remote call request reader
Read incoming requests and extract remote call request information
```
interface RequestReaderInterface {
    public function read(RequestInterface $request): InvokeRequestInterface;
}
```
##### Remote call response
Result of executing the call
```
interface InvokeResponseInterface {
    /**
     * @return mixed
     */
    public function getResult();
    
    public function getException(): ?\Throwable
}
```
##### Remote call info storage
Store remote call results for retries mechanism
```
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
```
interface LocalInvokerInterface {
    public function invoke(InvokeRequestInterface $request): InvokeResponseInterface;
}
```
##### Response generator
Transform invoke response to a regular response
```
interface ResponseGeneratorInterface {
    public function generate(string $callId, InvokeResponseInterface $response): ResponseInterface;
}
```
### Explainantions
#### #1
Let's say we have a hypothetical situation: customer places an order _(Checkout)_, but for each item ordered we have to update "ordered" count for related products _(Catalog)_
```
Class OrderManagement implements OrderManagementInterface
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
#### #2
We have a hypothetical situation: admin user with access to orders _(Checkout)_, but not to products _(Catalog)_ changes an order and removes a product from the items list:
```
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
#### #3
We have a hypothetical situation: customer updates their main address _(Customer)_ and we need to update their not-shipped orders to be
shipped to their main address _(Checkout)_
```
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
#### #4
Imagine we have a situation: an order is placed by an admin for a customer _(Checkout)_ and we need to add the shipment address of the order to the customer's address book _(Customer)_
```
class OrderManagement implements OrderManagementInterface
{

....

    public function place(OrderInterface $order): void
    {
        ....
        
        //WTF is $customerAddress instance of?
        $customerAddress = $this->customerAddressFactory->create();
        $customerAddress->setStreet($orderAddress->getStreet());
        ....
        //customerAddressRepository is a proxy
        //WTF is $createdAddress instance of?
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
#### #5
See an example of a proxy service implementation:
```
class ProxyOrderRepsoitory implements OrderRepositoryInterface
{
 
....
 
    public function place(OrderInterface $order): void
    {
        $this->invoker->invoke('checkout', OrderRepositoryInterface::class, 'place', [$order]);
    }
}
```
All proxy methods would look about the same: calling a remote method with the same name, there would never be multiple calls inside a
proxy method so there is no need for the "invoke" to return a promise and be asynchronous.
 
Another possiblity is to change existing methods to return promises and for client code to call different services asynchronously:
```
interface OrderRepositoryInterface
{
 
....
 
    public function place(OrderInterface $order): Promise;
 
....
 
}
 
class AddressRepository implements AddressRepositoryInterface
{

....

    public function save(AddressInterface $address): void
    {
        ....
        foreach ($orders as $order)
        {
            $order->setAddress($address);
            $pool->add($this->orderRepository->save($order));
        }
        $pool->wait();
    }

....

}
```
But this would require us to change all our existing service contracts and to rewrite our code to work with async code, and I don't think that's an option.
 
That's why remote calls should be done in a synchronous way.
