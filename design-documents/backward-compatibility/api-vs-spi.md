# API vs. SPI

## Background

Magento codebase consists of pubic and private code.
Public code is an interface for extension developers they can rely on, and core developers should preserve those interfaces for as long as possible.
[Module version dependencies](https://devdocs.magento.com/guides/v2.3/extension-dev-guide/versioning/dependencies.html) describes which version of a module an extension developer should depend on based on the code used in the extension.
The principles for public code are based on whether the core code is used as API (application programming interface) or SPI (service provider interface).
Due to commonly used practice of inheriting classes in Magento, many of them are considered both API and SPI because some extensions may use them in one of another way.
This makes those classes stuck in time.
It is impossible to add new methods to the class because it is a breaking change for an SPI class, and it is impossible to remove methods because it is a breaking change for an API class.

From another side, Magento [Technical Guidelines](https://devdocs.magento.com/guides/v2.3/coding-standards/technical-guidelines.html) discourage usage of inheritance ("2.6. Inheritance SHOULD NOT be used. Composition SHOULD be used for code reuse.").
This means that only interfaces should be considered SPI, while classes should be considered API-only code.  

## Problem Statement

As we're trying to find the balance between keeping extensions as compatible with core as possible, while allowing core developers to make some necessary changes, [Codebase changes](https://devdocs.magento.com/guides/v2.3/extension-dev-guide/versioning/codebase-changes.html) and [Module version dependencies](https://devdocs.magento.com/guides/v2.3/extension-dev-guide/versioning/dependencies.html) describe more and more rules and exceptions.
Also, see "classes explicitly meant for extension" in [Definition of Done](https://devdocs.magento.com/guides/v2.3/contributor-guide/contributing_dod.html?itm_source=devdocs&itm_medium=quick_search&itm_campaign=federated_search&itm_term=defin#review).
It becomes hard to remember and follow all the described recommendations.
It is also more difficult to follow a separate document than codebase itself.
And `@api` annotation used in the codebase is not sufficient now to guide a developer about dependencies.

## Solution

Distinguish API and SPI using different annotation in the code:

* `@api` for API
* `@spi` for SPI

This simplifies rules and helps get rid of exceptional cases.
See [updated Dependencies page](https://github.com/magento/devdocs/pull/5085/files) for possible simplifications of guidelines.

Guidelines should describe:
* API
   * changes in API increase MAJOR version of the module
   * extension developers should depend on MAJOR version if they call API methods
* SPI
   * changes in SPI increase MINOR version of the module 
   * extension developers should depend on MINOR version if they implement/extend SPI
   * XML configuration files are SPI. Special cases for `di.xml`
      * virtual types are private code by default. Can be marked as `@api` and/or `@spi`
      * lists (e.g., `CommandList`) can be marked as `@spi`
   * all classes intended for extension are SPI
 
## Scope of Work

### Initial Demarcation

Initially, mark the code as follows:

* For all interfaces marked with `@api`, also add `@spi`. Interfaces are intended for implementation, so they are SPI. Most of the interfaces are also intended for usage, so they are API. There might be exceptions, but those can be fixed in the future or used for new interfaces.
* For all classes marked with `@api`, keep as is. Classes are API-only by default.
* For classes marked with `@api` and explicitly intended for extension, add `@spi` annotation. List of classes (see [Definition of Done](https://devdocs.magento.com/guides/v2.3/contributor-guide/contributing_dod.html?itm_source=devdocs&itm_medium=quick_search&itm_campaign=federated_search&itm_term=defin#review)):
   * `\Magento\Framework\Model\AbstractExtensibleModel`
   * `\Magento\Framework\Api\AbstractExtensibleObject`
   * `\Magento\Framework\Api\AbstractSimpleObject`
   * `\Magento\Framework\Model\AbstractModel`
   * `\Magento\Framework\App\Action\Action`
   * `\Magento\Backend\App\Action`
   * `\Magento\Backend\App\AbstractAction`
   * `\Magento\Framework\App\Action\AbstractAction`
   * `\Magento\Framework\View\Element\AbstractBlock`
   * `\Magento\Framework\View\Element\Template`

### Update Guidelines

* Describe what is considered API and SPI in Magento. List specific types of PHP/XML/other files where it makes sense.

### Update Tools

Update SVC behavior according to the changes in the guidelines.

Optional: update release tools or add another tool that adds explanation for `@api` and `@spi` annotations:
```php
/* @api Can be called. Changes increase MAJOR version of the module. */
/* @spi Can be implemented/extended. Changes increase MINOR version of the module. */
``` 
