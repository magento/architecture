### Magento\Framework\Session\SessionManagerInterface should be used only in controllers and blocks
#### Why?
Web APIs (REST (by definition), SOAP, GraphQL) do not employ sessions,
relying on sessions in classes that are not directly linked to preparing HTML
may cause sessions being used during Web API requests unintentionally causing
unexpected behaviour during and negative performance impact.
 
#### Current situation
Sessions are being used in model classes.
 
#### What has to be done
Add PHPMD rule which will be triggered when a class has a constructor parameter of type
SessionManagerInterface (and any of the implementations) and is not an instance of
ActionInterface or AbstractBlock.
The rule's message should explain that UserContextInterface should be used to get current
user instead of sessions.
 
Removing all existing session usages from non-controller and non-block classes would
prove a difficult process so we just have to warn developers not to use them
in the future with the PHPMD rule.
