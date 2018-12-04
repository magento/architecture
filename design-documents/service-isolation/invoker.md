### Invoker API and SPI
Responsible for contacting remote services
#### API
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
     * @param string $class
     * @param string $method
     * @param array $arguments Keys - argument names.
     * @return PromiseInterface Will provide InvokeResponseInterface when resolved,
     *                 RemoteInvokeException on reject when networking fails,
     *                 has "wait" method to use in synchronous way.
     */
    public function invoke(
        string $class,
        string $method,
        array $arguments
     ): PromiseInterface;
}
```
 
#### SPI
##### Base URL information
URLs to access nodes
```php
interface ServiceSourceInterface {
    public function getUrl(): string;
}
```
##### Remote service sources repository
Find service sources information
```php
interface ServiceSourceRepositoryInteface {
    public function find(string $class, string $method): ServiceSourceInterface;
    
    public function getAll(): ServiceSourceInterface[];
}
```
##### Remote request data
Information about the request to be sent for a remote service
```php
interface InvokeRequestInterface {

    public function getUrl(): string;
    
    public function getClass(): string;
    
    public function getMethod(): string;
    
    public function getArguments(): array;
    
    public function getUserContext(): UserContextInterface;
}
```
##### Request data gatherer
Gathers information to send
```php
interface InvokerRequestGathererInterface
{
    public function gather(string $class, string $method, string $arguments): InvokeRequestInterface;
}
```
##### HTTP request generator
Creates HTTP request based on request data
```php
interface RequestGeneratorInterface
{
    public function generate(InvokeRequestInterface $invokeRequest): \Psr\Http\Message\RequestInterface;
}
```
##### Transport
```php
interface TransportInterface
{
    public function send(\Psr\Http\Message\RequestInterface $request): PromiseInterface;
}
```
##### Response reader
Reads responses from remote services
```php
interface ResponseReaderInterface
{
    public function read(\Psr\Http\Message\ResponseInterface $response): InvokeResponseInterface;
}
```
 
 
These SPIs should provide enough room for customization.