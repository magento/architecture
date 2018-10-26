## Remote code invoker
### What for?
Remotely calling services in proxy versions of modules
### Why not just use REST/SOAP?
* Not all service contracts are exposed as web API and nor should they be (for security and workflow reasons)
* Web API methods have certain ACL resources assigned to them so the ACL checks would be
performed every time a proxy service calls remote service unlike when a service
is called locally
* Proxy services would have to generate auth information of already authorized users (tokens etc) to access REST/SOAP
services which is a redundant process, also some of those REST/SOAP endpoints would have limitations in
place for certain type of users which would be useless in case of remote execution
(like requiring an admin user)
* Other application state spoofing may be required for seamless remote execution of services
### Functional requirements
* Invoker's code is a part of Magento\Framework namespace
* Invoker is able to call any class' method
* Multiple remote sources can be configured and assigned unique IDs
* Each HTTP request to remote sources is signed using unique source's key, timestamp and request's attributes
* Each remote call must be given unique ID to be used by retries mechanism
* Timeout limit and retries count can be configured globally and for each call
* Serialized arguments string includes arguments instances' classes information for proper deserialization
* Serialized remote method return values string must contain type information for
proper deserialization by the caller
* Remote call response can provide either serialized method call result or serialized exception to be
rethrow in order to emulate local method call
* Remote calls are done in a synchronous manner because in order for
to be able to utilize the possibility to make asynchronous remote calls the client code would have to know
that certain services can be proxies and that would complicate introduction of distributed Magento and
make monolith and distributed deploy modes coexistence harder
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
