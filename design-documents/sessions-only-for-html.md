### Magento\Framework\Session\SessionManagerInterface and Magento\Framework\Stdlib\Cookie\CookieReaderInterface should be used only in HTML related presentation level
#### Why?
Web APIs (REST (by definition), SOAP, GraphQL) do not employ sessions and cookies,
relying on sessions and cookies in classes that are not directly linked to preparing HTML
may cause sessions being used during Web API requests unintentionally causing
unexpected behaviour during and negative performance impact.
 
#### Current situation
Sessions and cookies are being used in model classes.
 
#### What has to be done
Add PHPMD rule which will be triggered when a class has a constructor parameter of type
SessionManagerInterface/CookieReaderInterface (and any of the implementations) and are not a part of HTML presentation level
(instances of ActionInterface, AbstractBlock, DataProviderInterface, Document, controller and block plugins).
The rule's message should explain that UserContextInterface should be used to get current
user instead of sessions and cookie usage should be avoided.
 
Removing all existing session and cookie usages from classes that are not a part of HTML presentation level would
prove to be a difficult process so we just have to warn developers not to use them
in the future with the PHPMD rule.
