## Content Security Policies
CSP allows a server to define a list of trusted sources of static files (scripts, styles, fonts, media files etc.)
along with disabling certain unsafe HTML features like inline JavaScript. It's a powerful tool
in the fight with XSS attacks.
## Modes
CSP supports two modes:
* restrict mode `Content-Security-Policy`
* report violations mode `Content-Security-Policy-Report-Only`
 
1st one would follow defined rules and prevent certain scenarios from happening
(like not loading a font from an untrusted address).
 
Second one will only report violations
to the server but proceed as it would without these restrictions. This mode would be convenient to merchants
just adopting CSP on their existing stores to see what kind of legit resources/features they need to enable
via CSP.
## Backward compatibility
In order to remind merchants that having CSP defined is important it will be enabled by default. But since adopting it
would require manually whitelisting trusted resource/features by merchants and extension developers CSP
will be functioning in the report-only mode. This will allow us to introduce CSP in a patch release.
 
## API
At the beginning Magento would not manage all of the possible CSP policies, that's
why we need an extensible SPI to allow 3rd party developers to implement and control new
policies.
 
CSP API/SPI will consist of the following interfaces:
 
* Policy DTO. The interface has only policy ID because not all CSP policies look like _\<policy name\> \<value\> \<value\>_,
  some may consist from multiple headers and have different syntax
```php
namespace Magento\Security\Model\CSP\Data;

interface PolicyInterface
{
    //Policy ID, like 'default-src' or 'referrer'
    public function getId(): string;
}
```
 
* Policy collector, will collect policies from different sources, will be composed from multiple collectors.
```php
namespace Magento\Security\Model\CSP;

interface PolicyCollectorInterface
{
    //Keys - policy IDs.
    public function collect(PolicyInterface[] $defaultPolicies = []): PolicyInterface[];
}
```
 
* Headers generator, will create headers based on a policy provided
```php
namespace Magento\Security\Model\CSP;

interface PolicyHeadersGeneratorInterface
{
    //With keys as header names and values as header values.
    public function generateHeaders(PolicyInterface $policy): array
}
```
 
* A violation recorded by a browser
```php
namespace Magento\Security\Model\CSP\Data;

interface ViolationInterface
{
    public function getDocumentUri(): string;
    
    public function getViolatedDirective(): string;
    
    public function getOriginalPolicy(): string;
    
    public function getReferrer(): ?string;
    
    public function getBlockedUri(): ?string;
    
    public function getSourceJsFileLine(): ?string;
    
    public function getReported(): \DateTime
}
```
 
* A repository of violations logs collected when CSP reporting is enabled
```php
namespace Magento\Security\Model\CSP;

interface ViolationsRepositoryInterface
{
    public function getList(SearchCriteriaInterface $search): ViolationSearchResultInterface;
    
    public function create(string $reportJson): void;
}
```
 
## CSP configuration
Security settings like CSP should be only manageable with physical files and
CLI requiring highest level of access.
 
CSP mode will be set with a config path not appearing in the admin configuration page so merchants
would have to execute a CLI command in order to change it.
 
#### Merchants
Merchants without dedicated development teams should be able to manage CSP.
Configuration for policies will be stored in a config path not exposed to admin _Configuration_ page
and managed by a CLI command `security:csp:add <policy> <value> <value> --replace`.
#### Theme developers
Themes may introduce new static files on Magento pages loaded from various resource so they need a
way to add those resources to the CSP whitelist. This can be done by introducing new _.xml_ file called
_csp_whitelist.xml_ which will allow to __add__ new sources to the whitelist but not to modify existing ones.
This sources will be required to have a URL or a URI with/without a wildcard but not containing
reserved words like 'self' or just a wildcard. _nonce-\*\*\*_ sources will also be allowed to be whitelisted here for
static inline javascripts. Policies that can be configured through this file
will be limited to _*-src/form-action_ policies.
 
An example of _csp\_whitelist.xml_:
```xml
<csp_whitelist xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:Security/etc/csp_whitelist.xsd">
    <policies>
        <policy id="font-src">
            <values>
                <value>fonts.mywebsite.com</value>
            </values>
        </policy>
    </policies>
</csp_whitelist>
```
 
#### Extension Developers
Extension developers will be able to place _csp_whitelist.xml_ inside their modules just as theme developers. For more intricate
cases they will be able to create a custom _PolicyCollectorInterface_ implementations to edit policies in the runtime.
 
#### Per-page configuration
It should be possible to configure CSP for a single page to allow access to certain resources/features
for just that one document. To achieve this developers will be able to implement a special interface by the
controller for the page they want custom CSP for.
```php
namespace Magento\Security\Model\CSP\PolicyCollector;

interface CSPAwareActionInterface extends ActionInterface
{
    public function modifyCsp(PolicyInterface[] $appliedPolicies): PolicyInterface[];
}
```
 
Thus controllers will be able to exclude/add policies to configured CSPs.

## Reporting
Browsers can report CSP violations in both modes. Magento will provide a default endpoint to receive these reports.
Since there is no way to authenticate a genuine report and on a live store they can fill up quickly the number of
reports stored in the database will be limited to 10000 deleting old ones when the limit is reached.
 
Merchants will see new violations attention message in the admin panel.
 
![alt text](img/csp_violations.png "CSP Violations")
 
Extension developers will be able to redefine default report-uri via config and implement a
custom _ViolationsRepositoryInterface_ to show them in the admin panel.
 
Violations report should be accessible through web API. Both CSP related admin pages and the web API endpoints
must require _Magento_Backend::system_ resource access.
 
## Default CSP
Magento uses some JavaScript libraries but usually has them provided with Magento itself without including
them from 3rd party resources so for most cases having *-src directives with 'self' value will be enough. In some .phtmls
we set event listener via attributes and we use eval in template.js so we'd have to also allow
'unsafe-inline' and 'unsafe-eval'. Magento has integrations with multiple 3rd party services like vimeo, youtube,
google analytics and various payment systems which we must whitelist additionally.
That will be the default whitelist Magento provides out of the box packaged alongside related modules via _csp\_whitelist.xml_.
