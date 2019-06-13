### Use scalars and maps of scalars only as E-mail template variables
#### The problem
Developers can use any object and access any method of available objects when
rendering an E-mail template, usually those objects would be models which methods
can often alter database, file system and cache storages. Using those methods in
templates can lead to unexpected behavior when during template rendering value will
be written to database, files changed etc. That should not happen in presentation layer
and E-mails are presentation. To make matters worse - administrators can make their own templates
through admin area and thus being able to execute any code on the server.

#### The solution
To prevent these problems from happening we must force using prepared data in
E-mail templates - scalars received by calling certain model layer services.
Thus templates will only receive data without the possibility to execute any code.

#### What needs to be done
###### Allow access to arrays' values
We need to be able to use hashmaps in templates just as we use objects right now e.g.
```
{{ var customer.name }}
```
_Magento\Framework\Filter\Template_ does not allow that right now so
it's _getVariable()_ method must be updated to allow just that.

###### Remove access to $this
Since $this use in E-mail templates refers to _Magento\Email\Model\Template_
it has to be removed as a default variable passed to templates.
 
Instead of using methods of _Magento\Email\Model\Template_ accessed with _$this_
like
```
$this.getUrl()
```
introduce new directives.

###### Do not accept objects as template variables
_Magento\Framework\Filter\Template::setVariables()_ needs to filter out all objects and
leaves only scalars and arrays of scalars.
 
Also it's _getVariable()_ methods needs to updated to provide access only to scalar variables.

###### Update core E-mail templates
All E-mail templates present in core Magento must be updated to use values instead
of calling methods.
 
When providing data to templates we need not to forget about extensibility and
pass redundant data - for instance if a 'new account' email uses only customer's
full name pass the whole customer info retrieved from _Customer::getData()_.

###### Make introducing new template directives easy for developers
Right now one can only add a directive by extending the _Template_ class and introducing
a new _\<directive name\>Directive()_ method - that is not a good way for developers
to introduce new directives - what if we have 2 separate extensions that want to introduce
2 different directives?
 
To solve this next SPI will be introduced:
```php
DirectiveProcessorInterface
{
    /**
     * Unique name of this directive.
     *
     * @return string
     */
    public function getName(): string;

    /**
     * Process values given to the directory and return rendered result.
     *
     * @param string|null Main parameter.
     * @param string[] Additional parameters.
     * @param string|null $html
     * @return string
     */
    public function process(?string $value, ?string $html, array $parameters): string;
    
    /**
     * Subdirectories that can be used within this directory's HTML.
     * 
     * @return DirectiveProcessorInterface[]|null
     */
    public function getSubDirectives(): ?array;
    
    /**
     * Default filters to apply if none provided in a template.
     *
     * @return string[]|null
     */
    public function getDefaultFilters(): ?array;
}
```
 
_Magento\Framework\Filter\Template_ will receive the list of such directive processors
via di.xml, parse all directives in given templates and give them to processors
until there is no directives left. That means that directives' HTML may contain
directives as well which will not be rendered by it's directive.
 
Consider the example:
```html
<h2>{{currentStoreName}}</h2>
<p>{{trans 'Hello, %name! %text' name=customer.name text='txt'|raw }}</p>
{{if show}}
<div>Showing</div>
{{else}}
<div>No show</div>
{{/if}}
```
 
When processing given template the _Template_ will try to find _DirectiveProcessorInterface_ whose
_getName()_ returns __currentStoreName__ to process the first directive, it will
call it's _process()_ method with _(null, null, [])_ since there is no value or parameters
provided, after it will call it's _getDefaultFilters()_ and it will return _'escape'_
because there are no filters provided in the template. _'escape'_ filter will be applied.
 
Next _Template_ will try to find _'trans'_ directive, it will pass _('Hello, %name! %text',
null, ['name' => 'Karen Smith', 'text' => 'txt])_ to it's _process()_ method. The directive's
_getDefaultFilters()_ will not be called since _'raw'_ filter is stated.
 
Then _Template_ will find __if__ directive, it's _getSubDirectives()_ will be called
since the directive contains HTML
and it will return __else__ directive processor. It's _process()_ method will
receive _(false, '<div>Showing</div>', [])_ and then the __else__ directive will
receive _(null, '<div>No show</div>', [])_. Both __if's__ and __else's__ _getDefaultFilters()_ will return nulls.
 
Another thing that 3rd party developers might want to extend/add are filters, I propose such SPI:
```php
interface FilterProcessorInterface
{
    /**
     * Process a value.
     *
     * @return string
     */
    public function filterValue(string $value): string;
    
    /**
     * This filter's unique name.
     *
     * @return string
     */
    public function getName(): string;
}
```
 
_Template_ class would receive a list of such filters and use them accordingly.

####### Alternatively:
Use twig, it's an established and well known template engine and will solve this problems, we would just have
to limit template variables to scalars.

#### Backward compatibility
This would not be a backward compatible change - we use objects in our own
E-mail templates in core Magento and developers are using objects in their own templates
as well.
 
But there are couple of steps we can take in order to make it easier for merchants to adopt this change:
###### Allow partial access to DataObject instances
_Template::setVariables()_ method to accept instances of _DataObject_ but in a template all property access instructions
and getters calls will be converted to _getData(\<name\>)_ calls
 
e.g.
```
{{var customer.name }} => $customer.getData('name')
{{var customer.getName() }} => $customer.getData('name')
```
Every other method call will result in an exception.
 
If the result of _getData()_ call on a _DataObject_ is and object - throw an exception.
e.g.
```
{{var cart.getItem() }} => Exception
{{var cart.getItem().getSku() }} => $cart.getData('item').getData('sku')
```
 
This way if a developer has used models just to access their data then the templates would keep rendering just fine.
 
###### Keep access to $this
_Magento\Email\Model\Template_ methods that are not meant for templates are already blacklisted, we can keep allowing
use the _Template_ class and all of it's methods in the templates and remove it in a later version.

###### Allow 3rd-party developers to whitelist more object/method pairs
By default we will have such pairs:
* DataObject,getData
* Magento\Email\Model\Template,getUrl
 
3rd-party developers will be able to extend this list via di.xml
 
###### Core templates
Both of these features are meant to make it easy on merchants to adopt these new changes and we should not use them in
our core E-mail templates to make it clear that we only want data passed to E-mail templates.
