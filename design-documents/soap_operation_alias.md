### SOAP Operation Alias
#### Current situation
While multiple RESTful endpoints using the same class/method can be declared by using different URLs/HTTP methods the
same cannot be done for SOAP operations - there's always 1 per class method.
 
#### What's the problem?
The reason multiple RESTful endpoints are created for the same service method is to cater for different users -
some are meant for admins, others for authenticated customers, others for guests. They may differ by authorization
criteria and accepted arguments/arguments' properties. While not found in core Magento it is possible to declare
different ACL resources assigned for endpoints to provide access for different admins, like an endpoint for admins
that have access to _Magento_Catalog::catalog_inventory_ and another, using the same service method, for admins that only
have _Magento_Catalog::categories_.
 
Same cannot be done for SOAP operations - when request comes for a SOAP operations it is required for a user to have
access to all ACL resources assigned to all different endpoints using the same requested service method. Example:
there is an endpoint for customers updating their own profile it uses _CustomerRepositoryInterface::save()_ and
requires "self" ACL resource access, there is another endpoint for admins updating any customers' information that also
uses _CustomerRepositoryInterface::save()_ and requires "Magento_Customer::manage" ACL resources access; When a request
for _customerCustomerRepositoryV1Save_ comes in the user is required to have both "self" and "Magento_Customer::manage"
ACL resources access. This creates situation when 1 declared endpoint (customers updating themselves)
can cancel out other endpoint exclusively for SOAP gateway. Worse situation would be if core Magento had an endpoint
for customers with "self" resources, and then an extension introduced an endpoint for admin with more severe
authorization - that would make original endpoint from Magento core inaccessible.
 
Another potential issue - different endpoints for admins. Let's say a 3rd-party developer wanted to create multiple
endpoints - one with more general resource access required - like "Magento_Catalog::catalog_inventory", and another
with more limited resource access like "Magento_Catalog::categories" and enforce a certain parameter for the latter
endpoint. This would work fine for RESTful gateway but would only allow access to admins with
"Magento_Catalog::catalog_inventory" access for SOAP.
 
To summarize - endpoints for SOAP represent partially aggregated variant of declared RESTful endpoints, which may
be confusing for developers and unintentionally create functional differences between RESTful and SOAP APIs.
 
#### What can be done?
The reason users required aggregated resource access for SOAP is because since there can be only 1 operation per service
method there's no way to know which endpoint's declared resources and other parameters to use. This can be solved by
allowing to declare multiple operations per class method and assigning them aliases so their name could be based
not only on actual class' methods.
 
For instance for editing customers it would look like this:
```xml
<?xml version="1.0"?>
<routes xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:module:Magento_Webapi:etc/webapi.xsd">
    <route url="/V1/customers/:customerId" method="PUT">
        <service class="Magento\Customer\Api\CustomerRepositoryInterface" method="save"/>
        <resources>
            <resource ref="Magento_Customer::manage"/>
        </resources>
    </route>
    <route url="/V1/customers/me" method="PUT"  soap-operation="saveSelf">
        <service class="Magento\Customer\Api\CustomerRepositoryInterface" method="save"/>
        <resources>
            <resource ref="self"/>
        </resources>
        <data>
            <parameter name="customer.id" force="true">%customer_id%</parameter>
        </data>
    </route>
</routes>
```
This way when _customerCustomerRepositoryV1SaveSelf_ operation is requested Magento would not to use second
(/V1/customers/me) endpoint's configurations to process it.
 
To achieve this:
* webapi.xsd needs to be altered to allow _soap-operation_ attribute for _route_ element
* _Magento\Webapi\Model\Config\Converter_ altered not to aggregate endpoints' config, but to create different methods
(operations) for SOAP gateway
* _\Magento\Webapi\Model\ServiceMetadata_ altered to work with virtual methods (aliased operations)
