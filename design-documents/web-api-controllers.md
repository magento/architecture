# Controllers for web APIs
## RESTful and SOAP API
#### Current situation
Service contracts, their arguments and return types are supposed to be used directly
as web API endpoints. The problem is service contracts are also used as API
and are supposed to be a part of business layer. By using them for presentation developers may encounter various
problems like having to put additional logic related to specifically processing web API requests (fields validation
for instance), DTOs used as arguments and return types may have sensitive data that is fine to use programmatically but
should not be exposed to web APIs or simply having properties needed for implicit business logic that would confuse web
API consumers.
#### Presentation layer processors.
The solution to the issues described above would be to introduce web APIs processors that would be a part of the
presentation layer, DTOs used for presentation and additional presentation-specific configuration.
##### Controllers
Right now any interfaces can be used as web API processors and DTOs, this is a convenient way of representing operations
and data used for web API - it should not be changed. But we have to leave the _Api_ namespace for module APIs. Web API
processors should have their own space - _WebAPI\Controller_ for processors and _WebAPI\Data_ for DTOs used for view.
 
###### Example
For instance let's take a look at creating a customer account:
 
```php
    AccountManagementInterface::createAccount(
        \Magento\Customer\Api\Data\CustomerInterface $customer,
        $password = null,
        $redirectUrl = ''
    );
```
 
The service contract responsible for it has this method signature, right of the bat we can see an argument that should
not be accessible to web API - _redirectUrl_ which is used to set redirect URL inside E-mail confirmation E-mail.
And what about password? From web API client's perspective it should be the $customer's property but it's a separate
account because _CustomerInterface_ is meant to represent business layer entity which doesn't store original password
and only has the password's hash. Also _CustomerInterface_ has _defaultShipping/Billing_ properties, but then
_AddressInterface_ also has _isDefaultShipping/Billing_ properties - which one should a web API client use to set a
customer's default billing/shipping address? Another problem - customer's addresses have _customerId_ property which
doesn't make sense in context of a customer managing itself since a customer can manage only their own addresses.
All of these problems stem from having business layer contracts/entities exposed as presentation layer operations/data.
 
Now let's see how we could improve this situation by having separate web API presentation related processors and view data:
 
###### View data
First we would have DTO that describes a customer the way it's convenient for web API clients:
```php
namespace Magento\Customer\WebAPI\Data;

interface CustomerUpdateInterface
{
    //No getId() since web API clients must not be able to control their customer's ID
    ...
    
    //For setting new password
    public function getPassword(): ?string;
    
    public function getAddresses(): CustomerAddress[];
}
```
 
Then we would have DTO for customer's addresses:
```php
namespace Magento\Customer\WebAPI\Data;

interface CustomerAddressUpdateInterface
{
    public function getId(): ?string;
    ....
    
    //No getCustomerId() because we don't give control over that
    ....
    
    public function getPostcode(): string
}
```

Since we don't want original password in customer related web API responses we could have separate view DTO for reading
a customer:
```php
namespace Magento\Customer\WebAPI\Data;

interface CustomerReadInterface
{
    //Now we have getId() since it's OK for web API clients to know their customer's ID.
    public function getId(): string;
    
    ....
    //No getPassword()
    ....
}
```
 
And now the processor for creating a storefront account:
```php
namespace Magento\Customer\WebAPI\Controller;

interface CustomerControllerInterface
{
    public function create(CustomerUpdateInterface $customer): CustomerReadInterface;
}
```

##### Validation
Web API clients displaying forms should be able to get a list of invalid properties sent to display to end users or
even have validation rules provided to them. Right now entities in Magento are being validated manually and errors
are being displayed one at the time. Having validation rules provided with the endpoints configuration would solve these
issues.
 
We could reuse existing validation mechanism that is partially being used for validating customer provided in
Magento framework in _Magento\Framework\Validator_ namespace
_(see app/code/Magento/Customer/etc/validation.xml)_.
 
_webapi.xml_ schema could be updated to allow following configuration:
```xml
<route url="/V1/customers" method="POST">
    <service class="Magento\Customer\WebAPI\Controller\CustomerControllerInterface" method="create"/>
    <resources>
        <resource ref="anonymous"/>
    </resources>
    <validation>
        <parameters>
            <parameter name="customer" entity="customer" group="web_api_create">
                <properties>
                    <!-- Individual address validators -->
                    <property name="addresses" entity="customer_address" group="web_api_create_address" walk="true" />
                    <!-- Validators for the whole collection -->
                    <property name="addresses" entity="customer_addresses" group="web_api_create" walk="false" />
                </properties>
            </parameter>
        </parameters>
    </validation>
</route>
```
 
REST/SOAP framework then would create validator for entity _"customer"_ and group _"web_api_create"_, perform validation
and generate standard validation error related response to web API clients. To allow displaying messages related only to
certain properties of object arguments of a web API processor the web API framework will also try to create validators
for them by generating an entity name and reusing the same group. For instance the framework will attempt to create
validators for entity with the name _"customer.email"_ and group = _"web_api_create"_ for the
_CustomerUpdateInterface::getEmail()_ property. Validation entity identifiers and groups can be specified explicitly
in the configuration. Customer creation API also allows creating sub-entities - addresses, we would want to validate
each one and the collection as a whole (like not allowing having more than 10 addresses). That's why _parameter_ elements
can have _property_ elements (which can have child _properties_ as well) to specify entity IDs and groups for those
properties. The ability to specify these entity IDs/groups for properties explicitly instead of relying on auto-generated
ones (like having _customer.addresses_ for addresses) is needed to reuse rules for different endpoints.
 
This validation configuration potentially can be reused for HTML controllers and graphQL resolvers as well.
 
This rules lists are extensible by 3rd party developers allowing them to easily add validation for new properties
they introduce or change defaults.
 
When validation fails web API framework will throw ValidationException. This exception has to be introduced since existing
Magento\Framework\Validator\Exception cannot describe validation errors properly. The exception will be rendered as following:
 
```json
{
    "errorMessage": "Validation failed",
    "validation_errors": {
        "arguments": {
            "customer": {
                "general": {
                    "customer_pwd_as_login": {"message": "Cannot have password being equal to login"}
                },
                "properties": {
                    "email": {
                        "general": {
                          "string_length_min": {"message": "Length must be greater then 10", "min": 10},
                          "customer_unique_email": {"message": "E-mail is already in use"}
                        },
                        "properties": {}
                    },
                    "addresses": {
                        "general": {
                            "collection_max": {"message": "Cannot contain more then 10 items", "max": 10}
                        },
                        "properties": {
                            "0": {
                                "general": {},
                                "properties": {
                                    "postcode": {
                                        "general": {
                                            "string_length_min": {"message": "Length must be greater then 5", "min": 5}
                                        },
                                        "properties": {}
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
```
