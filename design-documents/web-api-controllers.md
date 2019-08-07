# Middleware for web APIs, user input processing on top of service contracts execution
## RESTful and SOAP API
#### Current situation
Service contracts, their arguments and return types are supposed to be used directly
as web API endpoints. The problem is service contracts are also used as API
and are supposed to be a part of business layer. By using them for presentation developers may encounter various
problems like having to put additional logic related to specifically processing web API requests (fields validation
for instance) inside models, DTOs used as arguments and return types may have sensitive data that is fine to use programmatically but
should not be exposed to web APIs or simply having properties needed for implicit business logic that would confuse web
API consumers.
 
###### Example of service contracts not being tailored to be used directly as web API endpoints
For instance let's take a look at creating a customer account:
 
```php
    AccountManagementInterface::createAccount(
        \Magento\Customer\Api\Data\CustomerInterface $customer,
        $password = null,
        $redirectUrl = ''
    );
```
 
The service contract responsible for it has this method signature, right off the bat we can see an argument that should
not be accessible to web API - _redirectUrl_ which is used to set redirect URL inside confirmation E-mail.
And what about password? From a web API client's perspective it should be the $customer's property but it's a separate
account because _CustomerInterface_ is meant to represent business layer entity which doesn't store original password
and only has the password's hash. Also _CustomerInterface_ has _defaultShipping/Billing_ properties, but then
_AddressInterface_ also has _isDefaultShipping/Billing_ properties - which one should a web API client use to set a
customer's default billing/shipping address? Another problem - customer's addresses have _customerId_ property which
doesn't make sense in context of a customer managing itself since a customer can manage only their own addresses.
All of these problems stem from having business layer contracts/entities exposed as presentation layer operations/data.
 
 
###### Examples of classes that contain presentation layer logic
While service contracts implementations usually can contain all the logic needed to execute an operation various
presentation areas would require additional logic to convey results to users or to extract operations' arguments from
user input. Since service contracts are not supposed to know about context they are also not feet to contain additional
authorization related logic.
 
Examples of cases when we need additional context aware/presentation logic:
* When we need to validate entity ownership before performing an operation using authenticated user entity
* Limit frequency of operation execution (for anti-bruteforce measures)
* Prepare service layer DTOs for presentation, use additional service contracts for reading data to add to an
  operation's result (most used in HTML area)
* Plugin that would execute an additional operation based on session data/user context
* Request throttling for a specific operation
* Save additional data to session after performing an operation for presentation purposes
* Log operation result/execution including current context
 
Logic for cases described above is usually contained in controllers for HTML area, resolvers for GraphQL area and
_nowhere_ for REST/SOAP area. REST/SOAP area not having an ability to execute presentation level/context aware logic
is not the only problem - another problem is that often such logic can be reused across different areas and classes
containing it will be placed in _Model_ namespace which may confused developers into using them inside service contracts.
 
 
## GraphQL
#### Current situation
GraphQL (sometimes) utilizes service contracts and relies on them to have execute complex validations if any needed.
It also uses simple validation provided by graphql schema (like a property not allowed to be null). However if an
endpoint requires complex validation/authorization that is not performed by a service contract (and it shouldn't since
service contracts must not be aware of context) resolvers would duplicate to match functionality provided by regular
controllers providing HTML presentation. Furthermore since it's easier to define simple validation rules via graphQL
schema GraphQL gateway may provide more proper user input validation than HTML/REST/SOAP gateways.
 
 
## HTML Presentation
#### Current situation
Controllers implementing _ActionInterface_ do complex validation/authorization themselves which is fine as they are
supposed to be responsible for user input processing, still there's a problem with other areas not being able to reuse
this logic.
 
 
## What needs to be done to solve this?
#### Using operation-specific DTOs
In order to make it more clear what data will be actually used/allowed for a certain operation and to avoid transferring
of extensive data sets we should move from having generic DTOs describing entities to operation-specific DTOs describing
arguments and results of an operation.
 
Example on how would we change DTOs for operations regarding customer creation/update:
 
First we would have DTO that describes a customer in the way it's convenient for client code when updating a customer:
```php
interface CustomerUpdateInterface
{
    //No getId() since storefront clients must not be able to control their customer's ID
    ...
    
    //For setting new password
    public function getPassword(): ?string;
    
    public function getAddresses(): CustomerAddress[];
}
```
 
Then we would have DTO for customer's addresses:
```php
interface CustomerAddressUpdateInterface
{
    public function getId(): ?string;
    ....
    
    //No getCustomerId() because we don't give control over that
    ....
    
    public function getPostcode(): string
}
```

Since we don't want original password in customer responses we would not return it as read operations result:
```php
interface CustomerReadInterface
{
    //Now we have getId() since it's OK for storefront clients to know their customer's ID.
    public function getId(): string;
    
    ....
    //No getPassword()
    ....
}
```
 
By having these operation-specific DTOs we would only have the data we need as arguments and responses.
 
###### How to introduce it to Magento?
There are couple of ways we can introduce this concept to Magento:
* use it for new storefront API
* introduce new APIs to a module (for in stance _Customer_) in order to provide an example for 3rd-party developers
* create a devdocs page/update technical guidelines to inform community
 
 
#### Separate presentation layer/context aware classes 
Sometimes before passing arguments to a service contracts we would have to perform user input processing that cannot
be covered with declarative validation or existing ACL policies regarding of our current area (HTML/REST/SOAP/GraphQL).
 
Example is checking whether is a customer allowed to use a saved shipping address for an order by checking whether the
address was created by the same customer - we'd have to do it programmatically since validation rules wouldn't work
because of not having access to the full context and our ACL system is not fit to process those cases. HTML controllers
and GraphQL resolvers would be able to include such logic but it would not be reusable by other areas. This validation
could be performed by the service contract responsible for posting order but service contracts are not supposed to know
about authenticated users.
 
The solution to these problems would be to employ reusable area-agnostic context aware classes that would perform
such actions. These would be arbitrary classes with no common interface but they would be reusable across different areas
and reside inside a special namespace `ContextAware` to inform developers that this classes belong to presentation
layer and are not supposed to be used inside other layers. These classes could be used explicitly by HTML controllers
and graphQL resolvers and declared for REST/SOAP inside webapi.xml.
 
These classes would include classes for:
* working with user sessions
* complex authorization
* context-aware validation
* working directly with request/response objects
* extracting data from requests into service layer DTOs
* gathering data for view from service layer DTOs
 
 
In addition to examples above let's see how we could perform the ownership validation for shipping address with
a context aware class.
 
Let's say we have a service contract responsible for updating shipping address for an order:
```php
interface OrderManagerInterface
{
    ....
    
    public function updateShippingAddress(string $cartId, CustomerAddressInterface $shippingAddress): void
}
```
 
and then we create context aware class to check the ownership:
```php
namespace Magento\Sales\Presentation;

class OrderShippingAddressValidator
{    
    .....

    /**
     * @throws SecurityViolationException
     */
    public function validateAddressForOrder(UserContextInterface $context, CustomerAddressInterface $shippingAddress): void
    {
        $customerId = $context->getUserId();
        if ($shippingAddress->getId() && $shippingAddress->getCustomerId() !== $customerId) {
            throw new SecurityViolationException('Wrong shipping address used');
        }
    }
}
```
Notice how it uses authenticated user's ID. These classes would often use different context
objects like _RequestInterface_, _UserContextInterface_ and _SessionManagerInterface_ thus indicating they are not part of
service layer.
 
Then we should use it across different endpoints designed for customers applying shipping addresses to orders:
 
HTML controller:
```php
namespace Magento\Sales\Controller;

class EditAddress implements ActionInterface
{
    /**
     * @var OrderShippingAddressValidator
     */
    private $addressValidator;
    
    ....
    
    public function execute()
    {
        ....
        
        try {
            $this->addressValidator->validateAddressForOrder($this->userContext, $shippingAddress);
            
            $this->orderManager->updateShippingAddress($cartId, $shippingAddress);
        } catch (SecurityViolationException $exception) {
            $this->messageManager->addError($e);
        }
        
        ....
    }
}
```
 
GraphQL resolver:
```php
namespace Magento\SalesGraphQL\Model\Resolver;

class EditOrderAddress  implements ResolverInterface
{
    /**
     * @var OrderShippingAddressValidator
     */
    private $addressValidator;
    
    ....
    
    public function resolve(
            Field $field,
            $context,
            ResolveInfo $info,
            array $value = null,
            array $args = null
    ) {
        ....
        
        try {
            $this->addressValidator->validateAddressForOrder($userContext, $shippingAddress);
            $this->orderManager->updateShippingAddress($cartId, $shippingAddress);
        } catch (SecurityViolationException $exception) {
            throw new GraphQlInputException($exception);
        }
        ....
    }
}
```
 
When it comes to REST/SOAP we don't have a dedicated presentation layer processors where to invoke the validator.
To solve this we must introduce the ability to insert __middleware__ to be used before arguments are passed to defined via
webapi.xml service contracts and after they are executed. _Before_ middleware classes will have to have the same signature as the service contracts
that are responsible for endpoints in order to process arguments and will return arrays with modified arguments
in order to be able to change data before it reaches the service contracts (this is due to service contracts' arguments
being immutable). _After_ middleware classes will have to accept service contracts' return values and return same type of values.
 
Developers will be able to specify these middleware classes to be used for endpoints via webapi.xml.
 
Example on how we would introduce the shipping address validation to the endpoint using
_OrderManagerInterface::updateShippingAddress()_ service contract:
 
webapi.xml
```xml
<route url="/V1/cart/mine/shippingAddress" method="POST">
    <service class="Magento\Sales\Api\OrderManagerInterface" method="updateShippingAddress"/>
    <resources>
        <resource ref="anonymous"/>
    </resources>
    <processors>
        <before class="Magento\Sales\Presentation\WebApi\CartProcessor" method="beforePostAddress" />
    </processors>
</route>
```
 
And then we would create the processor to call the validator:
```php
namespace Magento\Sales\Presentation\WebApi;

class CartProcessor
{

    /**
     * @var OrderShippingAddressValidator
     */
    private $addressValidator;
    
    ....
    
    public function beforePostAddress(string $cartId, CustomerAddressInterface $shippingAddress): array
    {
        ....
        
        $this->addressValidator->validateAddressForOrder($this->userContext, $shippingAddress);
        
        return [$cartId, $shippingAddress];
    }
}
```
Notice how _beforePostAddress()_ arguments declaration fits _OrderManagerInterface::updateShippingAddress()_
arguments declaration.
 
###### How to introduce it to Magento?
* Add devdocs article regarding these new user-input-processors
* Update technical guidelines
* Implement processors/webAPI middleware in existing modules
  Suggested refactoring points:
  * Price permissions
  * Custom layout permissions
  * Unify customer address ID to customer ID validation across HTML/REST/SOAP/GraphQL presentations
  * Gift card attempts validator
  * Move non-API presentation layer/context aware classes into the new namespace in couple of modules
