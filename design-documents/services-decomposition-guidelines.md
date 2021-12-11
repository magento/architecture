# Services decomposition guidelines

## About

Document contains general recommendations and principles that need be considered and followed during services decomposition.

## Guidelines

1. Services should not maintain information about available stores/websites. Services can store scoped information about quote, configuration, reviews, etc. In some cases information about stores/websites that being saved with the data may need to be validated. If validation is critical services can request information about stores/website from websites service to use for validation. If validation is not critical, it can be omitted.

2. Modules use directory data (countries, cities, states). This data need to be reused between different services. As it almost never changes, it doesn't make much sense to add directory service. Directory data can be moved to configuration, so it can be shared across different services.

3. Modules have dependency on session. Handling session should be responsibility of BFF. BFF can pass user data to the service if needed.

4. For reporting purposes, services should publish event in the message queue with data for reporting services (BFF after first iteration). Reporting service then can consume the data and aggregate it for reporting. Service should publish excessive data after sanitation to minimize changes to services that provide data when evolving reports.

5. Need to consider code reuse disadvantage for deployment when applying decomposition. Sharing code between different services means that when updating one service we would need to redeploy another service.

6. Communication between services should be efficient. All network calls need to be evaluated and unnecessary calls should be eliminated. The following techniques can be used to reduce number of calls:
    * Calls can be grouped. Related data can be requested in one call to the service and used in different places by the service.
    * Part of the logic can be moved in a separate scenario. For instance, on the shopping cart page we need to validate QTY for each item to make sure we have enough items in stock and product is in correct QTY. This operation can be executed on it’s own instead on each quote load. If change affects UX, these changes need to be discussed with PO and UX team.
    * Some of the operations became not very useful in multi service environment. For instance, websites/stores validation may not be very useful in checkout service and these calls can be eliminated.
    * Making service own the data. Even though we expose editing configuration on the BFF, each service should own it’s own configuration.
    * In some cases number of calls can be reduced by creating new service. For instance, creating service to calculate totals for cart allows to reduce number of calls compare to having totals calculation logic in checkout service and having individual collectors send requests to total collectors.

7. Services can have separate interfaces. For instance, checkout service may have it's own ProductInterface that will contain only fields needed in checkout. What to do if checkout need more fields? It can easily add them to interface, but it will be breaking change to the customizations in checkout service that use this interface. To make possible extend data interface of another service, we can defined fields needed by service in the configuration and generate interface and DTO. This, however, doesn't resolve the problem with unnecessary data being sent from Catalog service. To avoid this, checkout service can request fields it needs.

8. Web API of the BFF should not change. BFF can expose composite operations (facade that performs calls to multiple services APIs) or proxy operations to services. API of the services may change during the transition (release of the service treated as new code) but after transition need to be kept backwards compatible.

9. Resources should not be shared between different services. Multiple Checkout instances can use the same database but Checkout and Catalog can't use the same database.

10. If interfaces (internal to service and exposed) missing data, they need to be modified to include missing data.

11. DTOs should be made light weight, models should be converted to DTOs.

12. Translation files should be in *Storefront modules deployed on BFF.

13. Extension mechanisms for the service are the same as for monolith application: framework level (modules, DI, plugins, etc) and extension points provided by individual modules.

14. When working with data need to obey the following rules:
    * Consumers should accept the unknown fields. Fields that are not used by the consumer should be ignored.
    * Consumers should validate only against required. Only fields that are used by the consumer need to be validated.
    * Producers should ignore unknown attributes.
    * Returning more data is always backwards compatible, returning less data is braking change.
    
15. Need to design services and APIs in a way that would minimize number of services that need to redeployed when one of the services change:
    * Follow best practices around working with data.
    * Use API versioning.
    * Be cautious about code reuse. As code reuse may lead to a need to redeploy multiple services need to make sure that it will at least not require redeploying them at the same time.
    
16. Framework should not manage retries, timeouts, circuit breaking and load balancing. Trying to implement this behaviour in application is an anti-pattern. Service mesh should be used instead.

## References

* [TolerantReader](https://martinfowler.com/bliki/TolerantReader.html)
