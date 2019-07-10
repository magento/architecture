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
{{var customer.name }}
```
_Magento\Framework\Filter\Template_ does not allow that right now so
it's _getVariable()_ method must be updated to allow just that.
 
To make these changes easier to adopt
```
{{var customer.getName() }}
```
 
Will be equal to accessing the 'customer' array by 'name' key.

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
 
To preserve backward compatibility instances of _DataObject_ will be accepted as well, then recursively their
_getData()_ methods will be called to get a graph of scalars to use in templates. 

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
     * @param mixed $value Template var, scalar or null if nothing has been passed to the directive.
     * @param string[] $parameters Additional parameters.
     * @param string|null $html HTML inside the directive.
     * @param \Magento\Framework\Filter\Template $template Template object calling the directive
     * @return string
     */
    public function process($value, ?string $html, array $parameters, Template $template): string;
    
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
until there is no directives left. That means that directives' HTML may contain other directives that will be processed
recursively.
 
Consider the example:
```html
<h2>{{currentStoreName}}</h2>
<p>{{trans 'Hello, %name! %text' name=customer.name text='txt'|raw }}</p>
```
 
When processing given template the _Template_ will try to find _DirectiveProcessorInterface_ whose
_getName()_ returns __currentStoreName__ to process the first directive, it will
call it's _process()_ method with _(null, null, [], $this)_ since there is no value or parameters
provided, after it will call it's _getDefaultFilters()_ and it will return _'escape'_
because there are no filters provided in the template. _'escape'_ filter will be applied.
 
Next _Template_ will try to find _'trans'_ directive, it will pass _('Hello, %name! %text',
null, ['name' => 'Karen Smith', 'text' => 'txt])_ to it's _process()_ method. The directive's
_getDefaultFilters()_ will not be called since _'raw'_ filter is specified.
 
While this SPI does not allow introducing all kinds of directives like your own _switch-case_ directive or a _for_ loop
directive it is definitely enough to cover 3rd-party developers needs.
 
Another thing that 3rd party developers might want to extend/add are filters, I propose such SPI:
```php
interface FilterProcessorInterface
{
    /**
     * Process a value.
     *
     * @param string $value HTML returned by a directive.
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
 
###### Old directive processors
Currently directives are being processed by methods of _Template_ class following _\<Directive Name\>Directive()_ pattern.
This forces developers to create child classes of _Template_ in order to introduce new directives or modify existing ones.
This way of working with directives has to be deprecated but preserved in order to keep backward compatibility.
 
We may create a dev-docs page explaining developers how to introduce directives and provide examples by creating new
directives in order to replace _\$this.method()\_ usages inside E-mail templates (like using $this.getUrl() method).

#### Backward compatibility
This would not be a backward compatible change - we use objects in our own
E-mail templates in core Magento and developers are using objects in their own templates
as well.

In order to preserve backward compatibility E-mail templates read from _view/\<area\>/email/\*.html_ files will keep working
just as before allowing developers time to adopt these new changes (e.g. still allow using objects as template variables).
New static tests will be added to notify developers
when they use method calls and _$this_ in E-mail templates, documentation will be updated to walk developers through these
changes.
 
When we update core E-mail templates to use scalars it is important to introduce new scalar variables replacing object
variables with new names so that old templates using objects can remain functional. For instance the _account\_new.html_
expects _\$customer_ variable to be a _Customer_ object so we would add _\$customer_data_ variables containing an _array_
with a customer's data so if a merchant was using customized _account\_new_ template it will still render fine.
 
We have to keep existing user-defined E-mail templates rendering as before to preserve backward compatibility. Newly-added
templates will be marked appropriately in the DB and rendered with new limitation regarding template variables though.
 
These measures aimed to improve backcompat are to be removed in a later minor version disallowing using objects
in E-mail templates for any case.
