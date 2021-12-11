### Rule for binding HTML values using knockout/Magento syntax
#### Description
Templates allow binding javascript variables/expressions using knockout/Magento to DOM elements' HTML
like this:
```html
<!-- Knockout syntax -->
<div data-bind="html: variable"></div>
<!-- Magento syntax -->
<div html="variable"></div>
```
Developers may use this binding instead of text binding for all values leaving XSS
vulnerabilities.

#### Proposing
Create static test to require developers to use variables/functions with _'UnsanitizedHtml'_ suffix
to bind HTML (like _variableUnsanitizedHtml_ or _widget.generateButtonUnsanitizedHtml()_). It's not a 100% guaranty that properly sanitized values will be used for dynamic HTML
but it will notify developers to think whether they really need HTML binding for a value or
maybe text binding will be enough. This rule would be more-or-less consistent with the rule we have for _.phtml_ templates.
