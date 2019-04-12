### Why?
Service contracts in the future will often be executed in an asynchronous manner
and it's time to introduce a standard Promise to Magento for asynchronous operations to employ.
Also operations like sending HTTP requests can be easily performed asynchronously since cUrl multi can be utilized
to send requests asynchronously.
### Requirements
* Avoid callbacks that cause noodle code and generally alien to PHP
* Introduce a way to work with asynchronous operations in a familiar way
* Employ solution within core code to serve as an example for our and 3rd party developers
### API
##### Deferred
A future that describes a values that will be available later.
If a library returns a promise or it's own implementation of a future it can be easily wrapped to support our interface.
 
This interface will be used as the return type of methods returning promises.
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

### Implementation
This interface will be used as a wrapper for libraries that return promises and deferred values.
 
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
functionality for developers to use by employing promises. Since it's an API, and a messy one at that, we should create
a new asynchronous client.
 
This client is being used in Magento\Shipping\Model\Carrier\AbstractCarrierOnline to create package shipments/
shipment returns in 3rd party systems, the process can be optimized by sending requests asynchronously to create
multiple shipments at once.
