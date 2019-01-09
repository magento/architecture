# Services decomposition guidelines

## About

Document contains general recommendations and principles that need be considered and followed during services decomposition.

## Guidelines

Services should have it's own core_config_data table that would contain settings specific to the service. Services should expose API to modify settings in core_config_data. Admin panel should support editing settings in the local database (settings for BFF) and for individual services via remote calls.

Services should not have knowledge about stores/websites. As some services store information about stores/websites with it's data stores/websites may need to be validated. If validation is critical services can request information about stores/website from websites service to use for validation. If not, validation can be omitted.

Modules use directory data (countries, cities, states). This data need to be reused between different services. As it almost never changes, it doesn't make much sense to add directory service. Directory data can be moved to configuration, so it can be shared across different services.

Modules have dependency on session. Handling session should be responsibility of BFF. BFF can pass user data to the service if needed.

For reporting purposes, services should publish event with data for reporting services (BFF after first iteration). Reporting service then can consume the data and aggregate it for reporting.

Need to consider code reuse disadvantage for deployment when applying decomposition. Sharing code between different services means that when updating one service we would need to redeploy another service.

Techniques to reduce number of calls
* Calls can be grouped. Related data can be requested in one call and used in different places by the service.
* Part of the logic can be moved in a separate scenario. For instance, on the shopping cart page we need to validate QTY for each item to make sure we have enough items in stock and product is in correct QTY. This operation can be executed on it’s own instead on each quote load.
* Some of the operations became not very useful in multi service environment. For instance, websites/stores validation may not be very useful in checkout service and these calls can be eliminated.
* Making service own the data. Even though we expose editing configuration on the BFF, each service should own it’s own configuration.
* In some cases number of calls can be reduced by creating new service. For instance, creating service to calculate totals for cart allows to reduce number of calls compare to having totals calculation logic in checkout service and having individual collectors send requests to total collectors.

Data interfaces can be reused across different services. For instance, ProductInterface can be used in Catalog and Checkout service. But it also has several disadvantages:
* Entities may contain data that is not needed in other services
* Reusing the same interfaces may lead to overhead in hydration/extraction and increase amount of data sent over network
* Changes to interface for one service may require redeployment of other services

Services can have separate interfaces. For instance, checkout service may have it's own ProductInterface that will contain only fields needed in checkout. What to do if checkout need more fields? It can easily add them to interface, but it will be breaking change to the customizations in checkout service that use this interface. To make possible extend data interface of another service, we can defined fields needed by service in the configuration and generate interface and DTO. This, however, doesn't resolve the problem with unnecessary data being sent from Catalog service. To avoid this, checkout service can request fields it needs.

Web API of the BFF should not change. BFF can expose composite operations (facade that performs calls to multiple services APIs) or proxy operations to services. API of the services may change.

Resources should not be shared between different services. Multiple Checkout instances can use the same database but Checkout and Catalog can't use the same database.

## Other documents
[Communication between services](https://github.com/magento/architecture/pull/50)
[Authentication and authorization](https://github.com/magento/architecture/pull/48)
[Caching](https://github.com/magento/architecture/pull/52)
