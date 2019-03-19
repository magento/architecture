# The Goal

The goal of this proposal is to find ways of improving Semantic Version Checker (SVC) build to catch more backward incompatible changes and reduce number of false positive failures.

Some of the following checks should be integrated into SVC build, while should be implemented as standalone tools.


# GraphQL Schema validation

GraphQL is designed to support continuous API schema evolution. There must be checks integrated into our CI process to guarantee schema backward compatibility.

To perform backward compatibility check a base version of the schema is required for each release line. 
 - Base schema can be generated manually and then stored along with the tests. There is no endpoint for retrieving SDL, the schema can be generated using some hacks after schema stitching process is complete.
 - It can also be taken from demo instance running on the latest version of the target release line. There are no demo instances maintained by the engineering team and it may be challenging to support them for GraphQL only. There are demo instances used by sales, but their state and readiness cannot be guaranteed (e.g. new EAV attributes can be created and affect the schema).

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

# Miscellaneous improvements

1. SVC must ignore removal of private constants like in [this PR](https://github.com/magento/magento2ce/pull/3875)
