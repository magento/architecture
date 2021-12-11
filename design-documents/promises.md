### Why?
Service contracts in the future will often be executed in an asynchronous manner
and it's time to introduce a standard Future to Magento for asynchronous operations to employ.
Also operations like sending HTTP requests can be easily performed asynchronously since cUrl multi can be utilized
to send requests asynchronously.
### Requirements
* Avoid callbacks that cause noodle code and generally alien to PHP
* Introduce a way to work with asynchronous operations in a familiar way
* Employ solution within core code to serve as an example for our and 3rd party developers
### API
##### Deferred
A future that describes a values that will be available later.
Only contains basic methods so that any asynchronous operations can be wrapping using this interface.
If a library returns a promise or it's own implementation of a future it can be easily wrapped to support our interface.
 
This interface will be used as the return type of methods returning futures.
```php
interface DeferredInterface
{
    /**
     * Wait for and return the value.
     *
     * @return mixed Value.
     * @throws \Throwable When it was impossible to get the value.
     */
    public function get();

    /**
     * Is the process of getting the value is done?
     *
     * @return bool
     */
    public function isDone(): bool;
}
```

Advanced interface that allows canceling asynchronous operations that have been started.
```php
interface CancelableDeferredInterface extends DeferredInterface
{
    /**
     * Cancels the opration.
     * 
     * Will not cancel the operation when it has already started and given $force is not true.
     * 
     * @param bool $force Cancel operation even if it's already started.
     * @return void
     * @throws CancelingDeferredException When failed to cancel.
     */
    public function cancel(bool $force = false): void;
    
    /**
     * Whether the operation has been cancelled already.
     * 
     * @return bool
     */
    public function isCancelled(): bool;
}
```

This interface can be used for operations that take to long and can be cancelled
(like stopping waiting for a server's response) or for delayed operations that could be
canceled even before they start (like cancelling an aggregated SQL query to DB after all the required criteria has been collected).

### Implementation
This interface will be used as a wrapper for libraries that return promises and futures.
 
### Explanations
##### Why not promises?
Promises mean callbacks. One callback is fair enough but multiple callbacks within the same method, callbacks for forwarded
promises create noodle-like hard to support code. Closures are a part of PHP but still are a foreign concept complicating
developer experience. Also it is an extra effort to ensure strict typing of return values and arguments with anonymous
functions.
 
Other thing is that promises are meant to be forwarded which complicates things. It can be hard to understand what are
you writing a callback for - promised result? Another callback for promised result introduced earlier? OnFulfilled callback
for resolved value in a OnRejected callback to the initial promise?

##### Typing
Methods returning Deferred can still provide types for their actual returned values - they can extend the original interface
and add return type hint to the _get()_ method.

##### Advantage
Since deferred does not require any confusing callbacks and forwarding it's pretty easy to just treat it as a values
and only calling _get()_ when you actually need it. Client code will look mostly like it's just a regular synchronous code.

### Using Deferred for service contracts
#### Why use futures for service contracts?
Another way that was proposed to execute service contracts in an asynchronous manner was to use async web API, but there
are number of problems with that approach:
* Async web API allows execution of the same operation with different sets of arguments, but not different operations
* Async web API was meant for execution big number of operations at the same time (thouthands) which is not the case
  for most functionality and mostly fits only import
* Since only 1 type of operation can be executed at the same time it will be impossible to execute service contracts
  from different domains at the same time
* Async web API uses status requests to check whether operations are completed which is alright for large numbers
  of operations but for a small number (like 2, 3) it will just generate more requests than just sending 1 request
  for each operation

So to allow execution of multiple service contracts from different domains it's best to send 1 request per operation
and to let client code to work with asynchronously received values almost as they would've with synchronous ones.
 
#### How will it look?
There are two ways we can go about using Deferred for asynchronous execution of service contracts:
* Service interfaces themselves returning deferred values for client code to use
 
  _contract's deferred_:
  ```php
  interface DTODeferredInterface extends DeferredInterface
  {
      /**
       * @inheritDoc
       * @return DTOInterface
       */
      public function get(): DTOInterface;
  }
  ```
 
  _service contract_:
  ```php
  interface SomeRepositoryInterface
  {
      public function save(DTOInterface $data): DTODeferredInterface;
  }
  ```
  
  _client code_:
  ```php
  class AnotherService implements AnotherServiceInterface
  {
      /**
       * @var SomeRepositoryInterface
       */
      private $someRepo;
    
      public function doSmth(): void
      {
          ....
        
          //Both operations running asynchronously
          $deferredDTO = $this->someRepo->save($dto);
          $deferredStuff = $this->someService->doStuff();
          //Started both processes at the same time, waiting for both to finish
          $dto = $deferredDTO->get();
          $stuff = $deferredStuff->get();
      }
  }
  ```
* Using a runner that will accept interface name, method name and arguments that will return a deferred

  _async runner_:
  ```php
  interface AsynchronousRunnerInterface
  {
      public function run(string $serviceName, string $serviceMethod, array $arguments): DeferredInterface;
  }
  ```
  _regular service_:
  ```php
  interface SomeRepositoryInterface
  {
    public function save(DTOInterface $dto): DTOInterface;
  }
  ```
  _client code_:
  ```php
  class AnotherService implements AnotherServiceInterface
  {
      /**
       * @var SomeRepositoryInterface
       */
      private $someRepo;
      
      /**
       * @var AsynchronousRunnerInterface
       */
      private $runner;
    
      public function doSmth(): void
      {
          ....
        
          //Both operations running asynchronously
          $deferredDTO = $this->runner->run(SomeRepositoryInterface::class, 'save', [$dto]);
          $deferredStuff = $this->runner->run(SomeServiceInterface::class, 'doStuff', []);
          //Started both processes at the same time, waiting for both to finish
          $dto = $deferredDTO->get();
          $stuff = $deferredStuff->get()
      }
  }
  ```

### Using deferred for existing code
We have a standard HTTP client - Magento\Framework\HTTP\ClientInterface, it can benefit from allowing async requests
functionality for developers to use by employing futures. Since it's an API, and a messy one at that, we should create
a new asynchronous client.
 
This client is being used in Magento\Shipping\Model\Carrier\AbstractCarrierOnline to create package shipments/
shipment returns in 3rd party systems, the process can be optimized by sending requests asynchronously to create
multiple shipments at once.

#### Asynchronous HTTP client API
Client:
```php
interface AsyncClientInterface
{
    /**
     * Perform an HTTP request.
     *
     * @param Request $request
     * @return HttpResponseDeferredInterface
     */
    public function request(Request $request): HttpResponseDeferredInterface;
}
```
 
Request:
```php
class Request
{
    const METHOD_GET = 'GET';

    const METHOD_POST = 'POST';

    const METHOD_HEAD = 'HEAD';

    const METHOD_PUT = 'PUT';

    const METHOD_DELETE = 'DELETE';

    const METHOD_CONNECT = 'CONNECT';

    const METHOD_PATCH = 'PATCH';

    const METHOD_OPTIONS = 'OPTIONS';

    const METHOD_PROPFIND = 'PROPFIND';

    const METHOD_TRACE = 'TRACE';

    /**
     * URL to send request to.
     *
     * @return string
     */
    public function getUrl(): string;

    /**
     * HTTP method to use.
     *
     * @return string
     */
    public function getMethod(): string;

    /**
     * Headers to send.
     *
     * Keys - header names, values - array of header values.
     *
     * @return string[][]
     */
    public function getHeaders(): array;

    /**
     * Body to send
     *
     * @return string|null
     */
    public function getBody(): ?string;
}
```
 
Response:
```php
class Response
{
    /**
     * Status code returned.
     *
     * @return int
     */
    public function getStatusCode(): int;

    /**
     * With header names as keys (case preserved) and values as header values.
     *
     * If a header's value had multiple values they will be shown like "val1, val2, val3".
     *
     * @return string[]
     */
    public function getHeaders(): array;

    /**
     * Response body.
     *
     * @return string
     */
    public function getBody(): string;
}
```
 
Future containing response:
```php
interface HttpResponseDeferredInterface extends DeferredInterface
{
    /**
     * @inheritdoc
     * @return Response HTTP response.
     * @throws HttpException When failed to send the request,
     * if response has 400+ status code it will not be treated as an exception.
     */
    public function get(): Response;
}
```

### Prototype
To demonstrate using DeferredInterface for asynchronous operations I've created prototype where requests sent to a
shipment provider were updated to be sent asynchronously using new asynchronous HTTP client.
 
To start check out an integration test that shows the difference between sending requests one by one or at the same time
available [here](https://github.com/AlexMaxHorkun/magento2/blob/futures-prototype/dev/tests/integration/testsuite/Magento/Ups/Model/ShipmentCreatorTest.php).
To make it work you would have to set up couple of environment variable before running it, list of those variables and
what they are meant for can be found inside the fixture used in the test.

### Sources
* DeferredInterface is based on Java's [Future interface](https://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Future.html)
* For real asynchronous operations [Guzzle HTTP client](https://github.com/guzzle/guzzle) was used
