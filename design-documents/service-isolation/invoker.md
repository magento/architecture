### Invoker API and SPI
Responsible for contacting remote services
#### API
##### Remote call response
Result of executing the call.
Can be used by 3rd party devs to add data to remote calls results or read it when they choose to modify
remote calling mechanism.
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
Used to make a call to a Magento server.
Can be used by 3rd party devs to write proxy services and modify invoker functionality/gateway.
```php
interface InvokerInterface {
    /**
     * @param string $interface
     * @param string $method
     * @param array $arguments Keys - argument names.
     * @return PromiseInterface Will provide InvokeResponseInterface when resolved,
     *                 RemoteInvokeException on reject when networking fails,
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
##### Base URL information
Nodes data.
Used to access nodes and provide requests for them.
```php
interface ServiceSourceInterface {
    public function getUrl(): string;
    public function isSignatureRequired(): bool;
}
```
##### Remote service sources repository
Find service sources information
```php
interface ServiceSourceRepositoryInteface {
    public function find(string $class, string $method): ServiceSourceInterface;
}
```
##### Remote request data
Information about the request to be sent for a remote service.
Can be used by 3rd-party devs to modify remote calls like adding additional data.
```php
interface InvokeRequestInterface {

    public function getUrl(): string;
    
    public function getClass(): string;
    
    public function getMethod(): string;
    
    public function getArguments(): array;
    
    public function getUserContext(): UserContextInterface;
    
    public function getCallId(): ?string;
}
```
##### Request data gatherer
Gathers information to send.
Generates invoke request based on arguments provided to the invoker.
Can be used by 3rd-party devs to modify remote call requests.
If a developer has a goal of editing or just reading data that is being sent to another service it's better to have a
DTO, than for developers to read HTTP requests, or parse their body, using gatherer or invokerRequest they can just
worry about the data being sent without knowing how a request is being rendered.
```php
interface InvokerRequestGathererInterface
{
    public function gather(string $class, string $method, string $arguments): InvokeRequestInterface;
}
```
##### HTTP request generator
Converts invoker request DTO to an HTTP request.
Can be used by 3rd-party devs to modify gateway used by invoker.
```php
interface RequestGeneratorInterface
{
    public function generate(InvokeRequestInterface $invokeRequest): \Psr\Http\Message\RequestInterface;
}
```
##### Transport
Actually sends the request.
Can be used by 3rd-party devs to introduce alternative gateway for the invoker.
```php
interface TransportInterface
{
    public function send(\Psr\Http\Message\RequestInterface $request): PromiseInterface;
}
```
##### Response reader
Reads responses from remote services.
Can be used by 3rd-party devs when introducing an alternative gateway for the invoker.
```php
interface ResponseReaderInterface
{
    public function read(\Psr\Http\Message\ResponseInterface $response): InvokeResponseInterface;
}
```
 
 
These SPIs should provide enough room for customization.