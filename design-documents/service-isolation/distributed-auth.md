### Authentication and authorization for remote services
#### Key attributes of proposed authentication mechanism
* Local validation of tokens, which is great for performance
* Token generation delegated to remote services which allows modules responsible for authentication logic
to be remote and auth logic to be extendable
* Minimal changes to existing mechanisms and preservation of existing functionality

#### Current situation
_Authorization_, _Webapi_ and _Integration_ modules are responsible for issuing and validating tokens for customer, integration
and admin users. There's _Token_ entity which holds token, user ID, user type, issue datetime and revoked
status information for all types of users. Issuing a token goes through integration module which uses customer/admin
service contracts to actually find a user and then integration module generates token and saves it to _oauth_token_
table. Authenticating via token is performed only by _Authorization_ and _Integration_ modules without having
to use and code from _Customer/User_ modules. Implementation of _UserContextInterface_ finds token in the HTTP request
data, looks it up in the _oauth_token_ table and decides whether a token is valid. Authorization is performed by
_Authorization_ module which uses ACL resources list read from config and roles found by locators to decide whether a
user can perform an action. The only other module modifying this behavior and providing additional roles source is
_Company_. Authorization is most often than not is performed only once for the whole endpoint and rarely called
explicitly in service contracts.

#### What do we need for isolated services?
Authenticating a user is an operation which will be performed for every client request. All client requests will be
directed to _front node_ which will be calling remote service contracts who are actually doing all the work. Considering
the circumstances it makes sense to try to avoid for front node to have to make remote requests every time we want to
authenticate a user and since all HTTP requests will be going through the _front node_ it should be able to perform
authenticating operations as independent as possible. 

#### What needs to be done for isolated services?
With current situation it's already pretty easy to prepare auth mechanism for microservices - most work is done in a
couple modules (Authorization, Webapi and Integration) which only delegate locating users to other modules when issuing a token.
What we need to do is to allow front node to perform authentication and authorization locally by keeping _Authorization_
, _Webapi_ and _Integration_ modules locally installed.
In more detail:
* Keep _Authorization_ and _Isolation_ modules installed on front node for them to perform token/acl role lookup locally
* Demand from developers who modify auth mechanism in their modules to separate code actually modifying auth and other
logic separately; For instance Company adds company roles and adjusts ACL permissions logic - the code responsible for
that should be separated into CompanyAcl module and everything else left in Company module
* _Authorization_, _Integration_, _CompanyAcl_ modules will be local for the _front node_ and remote for isolated
services for them to be able to perform ACL permissions validations additionally when needed
(Proxy versions of these modules will be installed on isolated services nodes)
* Magento\Framework\AuthorizationInterface is to be treated as a service contract and a proxy implementation is to be
provided in _AuthorizationProxy_ module for deployment on isolated services instances
* A service contract to be added to _Magento\User_ module to be able to validate administrators' credentials in
the _Integration_ module remotely

#### Flow
Example flow of a customer editing their info considering described above

* Customer sends POST HTTP request to /rest/V1/integration/customer/token to retrieve token
  * Request is sent to the _front node_ on http(s)://<front node domain>/rest/V1/integration/customer/token
  * Magento\Integration\Api\CustomerTokenServiceInterface::createCustomerAccessToken() is matched according to
  webapi.xml from Integration module installed on the front node
  * CustomerTokenService uses Magento\Customer\Api\AccountManagementInterface::authenticate() to find the customer
  based on provided username and password
  * Magento\CustomerProxy\Api\AccountManagementProxy is actually initiated by the object manager which sends remote call
  to Customer node to perform the login since CustomerProxy module is installed on the front node
  * CustomerTokenService generates token and stores it in oauth_token table for further use
  * token is returned in HTTP response to the client
* Customer sends PUT HTTP request to /rest/V1/customers/me to update their information
  * the HTTP request contains _Authorization_ headers with _Bearer \<token\>_ from the response to the previous HTTP
  request
  * Magento\Webapi\Model\Authorization\TokenUserContext initiated since _Webapi_ module is installed locally on the
  front node
  * TokenUserContext reads token from the _Authorization_ headers
  * TokenUserContext finds the token in oauth_token table, sets user type and ID
  * Magento\Webapi\Controller\Rest\RequestValidator (from the locally installed _Webapi_ module) validates permissions
  * RequestValidator uses Magento\Framework\Authorization which uses roles, found locally in _authorization_role_ table
  declared in locally installed _Authorization_ module and resources, declared in _acl.xml_'s of *Api modules installed
  on the front node to determine whether the user can perform the action
