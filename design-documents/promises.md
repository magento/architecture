### Why?
Service contracts in the future will often be executed in an asynchronous manner
and it's time to introduce a standard Promise to Magento for asynchronous operations to employ
### Requirements
* Promises CANNOT be forwarded with _then_ and _otherwise_ ([see explanation](#forwarding))
* _then_ accepts a function that will be executed when the promise is resolved, the callback will
receive a single argument - result of the execution
* _otherwise_ accepts a function that will be executed if an error occurs during the asynchronous
operation, it will receive a single argument - a _Throwable_
* Promises can be used in a synchronous way to prevent methods that use methods returning promises
having to return a promise as well ([see explanation](#callback-hell)); This will be done by promises having _wait_ method
* _wait_ method does not throw an exception if the promise is rejected
nor does it return the result of the resolved promise ([see explanation](#wait-not-unwrapping-promises))
* If an exception occurs during an asynchronous operation and no _otherwise_ callback is
provided then it will just be rethrown
### API
##### Promise to be used by client code
When client code receives a promise from calling another object's method
it shouldn't have access to _resolve_ and _reject_ methods, it should only be able to
provide callbacks to process promised results and to wait for promised operation's execution.
 
This interface will be used as the return type of methods returning promises.
```php
interface PromiseInterface
{
    /**
     * @throws PromiseProcessedException When callback was alredy provided.
     */
    public function then(callable $callback): void;
    
    /**
     * @throws PromiseProcessedException When callback was alredy provided.
     */
    public function otherwise(callable $callback): void;
    
    public function wait(): void;
}
```
##### Promise to be created
This promise will be created by asynchronous code
```php
interface ResultPromiseInterface extends PromiseInterface
{
    public function resolve($value): void;
    
    public function reject(\Throwable $exception): void;
}
```

### Implementation
A wrapper around [Guzzle Promises](https://github.com/guzzle/promises) will be created to implement the APIs above. Guzzle Promises fit
most important criteria - they allow synchronous execution as well as asynchronous. It's a mature
and well-known library and, while we would have to add guzzle/promises to our composer.json,
the library is already required in Magento indirectly - we won't be actually adding a new dependency.
 
There are other libraries like [Reactphp Promises](https://github.com/reactphp/promise) but they either do not provide synchronous way
to interact with promises or are not as refined.
 
### Explanations
##### Forwarding
Consider this code
```php
$promise = $this->anotherObject->doStuff();
$promise->then($doStuffCallback)
    ->otherwise($processErrorCallback)
    ->otherwise($processAnotherError)
    ->then($doOtherStuff)
    ->otherwise($processErrorCallback);
```
Does 1st _then_ return a forwarded promise? Or is it the same object?
Then what promise is the second _otherwise_ callback for?
Code looking like this is confusing and it will be much cleaner if we don't use forwarding.
```php
$doStuffOperation = $this->someObject->doStuff();
$result = null;
$doStuffOperation->then(function ($response) use (&$result) { $result = $response; });
$simultaneousOperation = $this->otherObject->processStuff();
$doStuffOperation->wait();
$updated = null;
$saveOperation = $this->repo->save($result);
$saveOperation->then(function ($result) use (&$updated) { $updated = $result; });

//Waiting for all
$saveOperation->wait();
$simultaneousOperation->wait();
return $updated;
```
Here we clearly state that for _save operation_ we need to _do stuff operation_ to finish
and _simultaneous operation_ may run up until then end of our algorithm execution.
 
 
##### Callback hell
Consider this code responsible for placing orders
```php
class ServiceA
{
....

    public function process(DTOInterface $dto): ProcessedInterface
    {
        ....
        
        $this->serviceB->processSmth($val)->then(function ($val) use ($processed) {
            $processed->setBValue($val);
        });
        
        return $processed;
    }

....
}
```
We cannot be sure _serviceB_ has finished doing it's stuff when we return _$processed_.
So, if we cannot wait for the promise _serviceB_ returned, the only thing we can do
is to return a promise ourselves instead of _ProcessedInterface_. But then methods using
_ServiceA::process()_ would have to do the same - and PHP code is not supposed to be this way.

##### Wait not unwrapping promises
For _wait_ method to also unwrap promises can result in confusion. It's better to have a single way of retreiving promised results and a single way of retreiving errors.
 
Consider next situation:
```php
class ServiceA
{
    public function doSmth(): void
    {
       $promise = $this->otherService->doSmthElse();
       $promise->otherwise(function ($exception) { $this->logger->critical($exception); });
       try {
           //wait would throw the exception processed in the otherwise callback once again
           $promise->wait();
       } catch (\Throwable $exception) {
           //we've already processed this
       }
    }
}
```
That is a simple situation but it illustrates how having multiple ways of receiving promised results may lead to duplicating code
