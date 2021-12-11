# The Goal

The goal of this proposal is to find ways of improving Semantic Version Checker (SVC) build to catch more backward incompatible changes and reduce number of false positive failures.

Some of the following checks should be implemented as standalone tools and integrated into SVC build.

It makes sense to update 3rd party libraries before starting work on any improvements. For example [SVC library](https://github.com/tomzx/php-semver-checker) is more than a year-old and should be updated.


# GraphQL Schema validation

GraphQL is designed to support continuous API schema evolution. There must be checks integrated into our CI process to guarantee schema backward compatibility.

To perform backward compatibility check a base version of the schema is required for each release line. 
 - Base schema can be generated manually and then stored along with the tests. There is no endpoint for retrieving SDL, the schema can be generated using some hacks after schema stitching process is complete.
 - It can also be taken from demo instance running on the latest version of the target release line. There are no demo instances maintained by the engineering team and it may be challenging to support them for GraphQL only. There are demo instances used by sales, but their state and readiness cannot be guaranteed (e.g. new EAV attributes can be created and affect the schema). 
 - It should be possible to install two instances during the build and compare the resulting schemas, both of them must be accessible via HTTP.

Suggested tools:

1. [Backward compatibility checker](https://github.com/rodionovp/graphql-schema-compatibility-checker) can verify schemas using introspection queries or local schema files


### The following options have also been considered
1. [Validation with Apollo Engine](https://blog.apollographql.com/schema-validation-with-apollo-engine-4032456425ba) looks promising, but is not suitable since requires Apollo Engine subscription

# Message queue

Message queue topics and data type changes can break backward compatibility and must be tracked by SVC builds. According to [Magento versioning policy](https://devdocs.magento.com/guides/v2.3/extension-dev-guide/versioning/codebase-changes.html):
 - Topic removal or argument modification should trigger Major version bump
 - Consumer removal and new topic must trigger Minor version bump
 
It is possible to get information about message queue configuration from the XML configs, however it might not reflect the full picture since some of the configuration is performed by the framework automatically, e.g. by async bulk API.

Suggested approach is to compare actual configuration of the RabbitMQ after Magento installation, which is similar to the approach for DB comparison test. [RabbitMQ management CLI](https://www.rabbitmq.com/management-cli.html) can be used to get necessary info directly from the RabbitMQ instance.

# Skipped DB compare checks

1. There is a huge tech debt in a form of skipped tables and fields during DB compare check. The blacklists can be found here `tools/Magento/Tools/DatabaseCompare/etc/ce/config.php`.
1. The tool should ignore the order of the fields during comparison since the order can change when module installation order changes.
1. PHP errors are ignored, but must cause build failure.

# Missing analysis for BIC-relevant PHP code changes

The following categories of code changes impact backward compatibility but are missing from the analysis performed by the existing SVC tool and must be added to the ruleset.

### Type declaration annotations

1. The current SVC code handles changes in type declarations (adding/removing/changing) for API method parameters and return types only if the type declarations are done in-line, but `@param` and `@returns` annotations are widely used across the codebase in place of in-line type declarations and must checked for BIC.
1. API class properties must be checked for typing additions, removals and changes using the same change level rules as method parameters.  In-line property typing will not be supported [until PHP 7.4](https://wiki.php.net/rfc/typed_properties_v2), but we use `@var` annotations in our codebase to declare property types so they must be checked along with the `@param` and `@return` annotations.

[MC-16327](https://jira.corp.magento.com/browse/MC-16327) has been created to address these issues.

### Traits

Traits are used occasionally in our API (example: [Magento\Payment\Helper\Formatter](https://github.com/magento/magento2/blob/2.3-develop/app/code/Magento/Payment/Helper/Formatter.php)) and must be analyzed by the SVC tool.  Traits must be treated the same as abstract classes for analysis except private members must have the same change level as protected members; private members of traits are accessible by extending classes and thus have BIC considerations.

### Visibility changes

Visibility changes across all class members must be considered for BIC; currently, a public method moving to protected or even private does not register in the tool.  More-restrictive visibility adjustments must be treated as removals (MAJOR), less-restrictive adjustments must be treated as additions (MINOR).

### Class inheritance

1. Overriding a parent method in a child API class must be treated as a changed method (PATCH), currently it is treated as an added method (MINOR) even though the method already existed on the class.  Note: moving a method to a parent class is already handled as move a move operation instead of a removal and does not need to be changed.
1. Changing the parents of an API class (such as making the class extend a different abstract class or implement a different interface) must be a MAJOR-level change, as the class can have different methods and other members or it could become incompatible with method parameters that expect an object of the previous parent's type.
1. Adding a parent (implemented interface or extended class or trait) to a previously parent-less API class must be considered for BIC, as it adds methods and other members to the class and creates type compatibility constraints that must be considered for future changes.  This should be a MINOR-level change.
1. Removing a parent (implemented interface or extended class or trait) from an API class must be considered for BIC, as it removes methods and other members from the class and can cause type compatibility issues with code expecting the class to belong to the removed parent.  This should be treated the same as method removal and changes in return types (MAJOR-level).
1. API classes must be checked for changes to parent classes or traits they extend even if those parents aren't also marked as API, as a parent's methods and other members affect BIC for all classes that extend them.

# Other backward incompatible changes

There is a [document](https://devdocs.magento.com/guides/v2.3/extension-dev-guide/versioning/codebase-changes.html) listing possible Minor/Major changes. 
Most of PHP class and DB schema changes are covered with SVC and DB compare tools, the coverage may be incomplete and needs to be verified.

The rest of the BiC changes listed in the [document](https://devdocs.magento.com/guides/v2.3/extension-dev-guide/versioning/codebase-changes.html) must be prioritized and groomed separately. Custom tools will be required for 
- XML configs
- JS
- CLI interface
- Less
- Events

# False positive failures

### SVC tool
1. SVC must ignore removal of private constants like in [this PR](https://github.com/magento/magento2ce/pull/3875)
1. Adding optional parameter to the constructor must be detected as Patch change for all classes except for the explicitly defined list of classes used for extension. Examples of such classes include legacy layer supertypes like `AbstractExtensibleModel`

### DB comparison tool
1. In some cases module installation order changes values some fields for specific entities (like `position`). There are two possible solutions: ignore all `position` fields in the specific table OR ignore the specific entity. Having such blacklists may prevent identification of the future BiC changes. Proposed solution is to introduce field-level blacklist for specific records.