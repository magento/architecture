### Overview

To improve security and logistics we need to allow limiting Actions to processing only requests with certain HTTP methods and add those limitations to as many existing Actions as possible.
There are many vulnerabilities caused by actions processing both GET and POST requests and thus allowing bypassing security validations like form key validation. Also limiting actions to processing only requests with certain methods would serve as self-documentation for Action classes and improve consistency of server side for client code and functional tests.

### Theory of Operation

Since we use by-convention routing it would be inconsistent to defined actions’ allowed HTTP methods in a config file, so it’s better to create marker interfaces like these:

>interface HttpGetActionInterface extends ActionInterface
>
>{
>
>}
>
>
>interface HttpPostActionInterface extends ActionInterface
>
>{
>
>}
>
>
>interface HttpPutActionInterface extends ActionInterface
>
>{
>
>}

etc.

One interface per each HTTP method (using list of HTTP methods from \Zend\Http\Request::METHOD_* constants). We will have an exentable map *interface* => *HTTP method* passed to the validator to allow custom HTTP methods to be processed.

Request method will be validated in the FrontController before executing a matched action, somewhat like this:

>foreach ($interfaceMap as $interface => $httpMethod) {
>
>&nbsp;&nbsp;&nbsp;&nbsp;if ($actionInstance instanceof $interface && $request->getMethod() !== $httpMethod) {
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;throw new NotFoundException(__(‘Page not found.’));
>
>&nbsp;&nbsp;&nbsp;&nbsp;}
>
>}

 
To limit already existing Actions and get developers to know these new interfaces a temporary event listener will be added for controller_action_postdispatch event to log HTTP methods used with actions and then the full suite of functional tests (MTF and MFTF) would be ran to get information on as much actions as possible.

After corresponding Http*Method*ActionInterface will be added to every Action class we have information on. If an action is logged to be accessed only by GET methods it will allow only GET, if only by POST – allowing only POST etc. If by more than one methods – the Action class will not be updated.

### Work Breakdown
 * Add Http*Method*ActionInterface and implement request validation in the FrontController
 * Create logging mechanism for Actions
 * Log HTTP methods used when running full functional tests’ suite
 * Create script to add Http*Method*ActionInterface to Action classes we’ve collection information about
 * Update Action Classes
