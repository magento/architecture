### Overview

To improve security and logistics we need to allow limiting Actions to processing only requests with certain HTTP methods and add those limitations to as many existing Actions as possible.
There are many vulnerabilities caused by actions processing both GET and POST requests and thus allowing bypassing security validations like form key validation. Also limiting actions to processing only requests with certain methods would serve as self-documentation for Action classes and improve consistency of server side for client code and functional tests.

### Theory of Operation

Since we use by-convention routing it would be inconsistent to defined actions’ allowed HTTP methods in a config file, so it’s better to add an interface for Actions to implement:

>interface HttpMethodAwareActionInterface extends ActionInterface
>
>{
>
>&nbsp;&nbsp;&nbsp;&nbsp;/**
>
>&nbsp;&nbsp;&nbsp;&nbsp; * @return string[]|null
>
>&nbsp;&nbsp;&nbsp;&nbsp;*/
>
>&nbsp;&nbsp;&nbsp;&nbsp;public function getAllowedHttpMethods(): ?array;
>
>}

If getAllowedHttpMethods returns null – all methods are allowed.

This interface will be used in the FrontController and requests will be validated there somewhat like this:

>if ($actionInstance instanceof HttpMethodAwareActionInterface) {
>
>&nbsp;&nbsp;&nbsp;&nbsp;$allowedMethods = $actionInstance-> getAllowedHttpMethods();
>
>&nbsp;&nbsp;&nbsp;&nbsp;If ($allowedMethods && !in_array($request->getMethod(), $allowedMethods, true) {
>
>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;throw new NotFoundException(__(‘Page not found.’));
>
>&nbsp;&nbsp;&nbsp;&nbsp;}
>
>}

 
To limit already existing Actions and get developers to know this new interface a temporary event listener will be added for controller_action_postdispatch event to log HTTP methods used with actions and then the full suite of functional tests (MTF and MFTF) would be ran to get information on as much actions as possible.

After the HttpMethodAwareActionInterface will be added to every Action class we have information on. If an action is logged to be accessed only by GET methods it will allow only GET, if only by POST – allowing only POST, if by more than one methods – the Action class will not be updated.

### Work Breakdown
 * Add HttpMethodAwareActionInterface and implement request validation in the FrontController
 * Create logging mechanism for Actions
 * Log HTTP methods used when running full functional tests’ suite
 * Create script to add HttpMethodAwareActionInterface to Action classes we’ve collection information about
 * Update Action Classes
