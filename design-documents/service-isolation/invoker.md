### Invoker API and SPI
Responsible for contacting remote services
#### API
##### Remote service call response
Result of executing the call.
Can be used by 3rd party devs to add data to remote calls results or read it when they choose to modify
remote calling mechanism.
```php
namespace Magento\Framework\Invoker;

interface ResponseInterface {
    /**
     * @return mixed Value that remote service returned.
     */
    public function getResult();
    
    /**
     * Exception that remote service had thrown, if any.
     *
     * @return \Throwable|null
     */
    public function getException(): ?\Throwable
}
```
##### Remote Invoker
Used to make a call to a remote service contract.
Can be used by 3rd party devs to write proxy services and modify invoker functionality/gateway.
```php
namespace Magento\Framework\Invoker;

interface InvokerInterface {
    /**
     * @param string $interface
     * @param string $method
     * @param array $arguments Keys - argument names.
     * @return PromiseInterface Will provide ResponseInterface when resolved,
     *                 InvokeException on reject when networking fails,
     *                 has "wait" method to use in synchronous way.
     */
    public function invoke(
        string $interface,
        string $method,
        array $arguments
     ): PromiseInterface;
}
```
 
#### SPI
##### Service information
Nodes data.
Used to access nodes (services) and provide requests for them.

May be used by 3rd-party developers to read services data for their own gateway implementation.
```php
namespace Magento\Framework\Invoker\Contacts;

interface ServiceInformationInterface {
    public function getUrl(): string;
    
    public function getCertFile(): ?string;
    
    public function getCertPassword(): ?string;
}
```
##### Remote service information repository
Find services contact information using a declared service contract.
May be used by 3rd-party developers to read services data for their own gateway implementation.
```php
namespace Magento\Framework\Invoker\Contacts;

interface ServiceInformationRepositoryInteface {
    public function find(string $interface, string $method): ServiceInformationInterface;
}
```
##### Remote request data
Information about the request to be sent to a remote service.
Can be used by 3rd-party devs to modify remote calls like adding additional data.
```php
namespace Magento\Framework\Invoker\Client;

interface RequestInterface {

    public function getUrl(): string;
    
    public function getInterface(): string;
    
    public function getMethod(): string;
    
    public function getArguments(): array;
    
    public function getUserContext(): UserContextInterface;
    
    public function getRequestId(): string;
    
    public function getCorrelationId(): string;
}
```
##### Request data provider
Gathers information to send.
Generates invoke request based on arguments provided to the invoker.
Can be used by 3rd-party devs to modify remote call requests.
If a developer has a goal of editing or just reading data that is being sent to another service it's better to have a
DTO, than for developers to read HTTP requests, or parse their body, using RequestProviderInterface or RequestInterface
they can just worry about the data being sent without knowing how a request is being rendered.
```php
namespace Magento\Framework\Invoker\Client;

interface RequestProviderInterface
{
    public function create(string $interface, string $method, string $arguments): RequestInterface;
}
```
##### Remote message
Marker interface for messages (like an HTTP request) to be sent to services.
Can be used by 3rd-party devs to modify how the actual requests are being sent or create another type of messages.
```php
namespace Magento\Framework\Invoker\Client;

interface MessageInterface
{

}
```
##### Remote response
Marker interface for responses (like an HTTP response) returned from service nodes.
Can be used by 3rd-party devs to modify how the actual requests are being sent or create another type of messages.
```php
namespace Magento\Framework\Invoker\Client;

interface ResponseMessageInterface
{

}
```

##### Remote message generator
Converts invoker request DTO to a message (like an HTTP request).
Can be used by 3rd-party devs to modify gateway used by invoker.
```php
namespace Magento\Framework\Invoker\Client;

interface RequestGeneratorInterface
{
    public function generate(RequestInterface $invokeRequest): MessageInterface;
}
```
##### Transport
Actually sends messages in an asynchronous way.
Can be used by 3rd-party devs to introduce alternative gateway for the invoker.
```php
namespace Magento\Framework\Invoker\Client;

interface ClientInterface
{
    public function send(MessageInterface $request): PromiseInterface;
}
```
##### Response reader
Reads responses from remote services (processes _then_ of the promise _ClientInterface_ returns).
Can be used by 3rd-party devs when introducing an alternative gateway for the invoker.
```php
namespace Magento\Framework\Invoker\Client;

interface ResponseReaderInterface
{
    public function read(ResponseMessageInterface $response): ResponseInterface;
}
```
 
 
These SPIs should provide enough room for customization as well as adopting the invoker for alternative gateways.
 
#### The flow
* InvokerInterface calls RequestProviderInterface to create invoke request based for the service contract requested
* RequestProviderInterface finds remote service info using ServiceInformationRepositoryInterface
* RequestProviderInterface retrieves data required to fill an instance of RequestInterface
(generates call/correlation ID, passes user context)
* RequestInterface returned to InvokerInterface
* InvokerInterface passes RequestInterface to RequestGeneratorInterface to create a message to be sent
* RequestGeneratorInterface returns MessageInterface
* InvokerInterface uses ClientInterface to send the MessageInterface instance previously generated
* ClientInterface returns a promise
* InvokerInterface creates a promise to be returned
* InvokerInterface provides _then_ callback for ClientInterface's promise that will use ResponseReaderInterface to read
ResponseMessageInterface instance ClientInterface will use to fulfill it's promise and then will fulfil invoker's promise
* InvokerInterface will wrap exceptions coming from ClientInterface's promise rejection into a InvokerExceptions