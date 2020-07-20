# Configuration Propagation from Admin Panel to Storefront

## Problem Statement

Existing Magento monolith is responsible for the whole cycle of data and user workflow.
Magento Admin Panel contains configuration settings for all areas of the application, including Admin Panel behavior, storefront UI, etc.

With Storefront being a separate service (or set of services), should configuration settings in Magento Admin Panel be propagated to storefront service?
If yes, in what way?

## Configuration Propagation Guidelines

This document covers general agreements for decisions about configuration propagation from Admin Panel to storefront. 
A separate design decision should be made and documented for specific use cases.

There are three use cases for the configuration, based on its impact.

### 1. Configuration that impacts Admin Panel behavior

Examples: admin ACLs that allow or deny parts of functionality for the admin user, reindex on update or by schedule.

This type of configuration has nothing to do with Storefront and should not be propagated to storefront, as well as should not be exposed via export API. 

### 2. Configuration that impacts data provided by Storefront

Example: Base Media URL is used for calculation of media image URLs returned by Storefront API.

This type of configuration should be propagated to Storefront in one of the following ways:

1. Pre-calculate final data on back office side and provide final result to the Storefront during synchronization.
   1. Pros: simpler implementation of Storefront due to eliminated necessity for Storefront to keep knowledge about additional configuration, including data calculation algorithms.
   2. Cons: massive (up to full) reindexation necessary in case the configuration is changed.
2. Move/duplicate calculation logic in the Storefront service based on original data is synced from the back office.
    1. Pros and cons are opposite to option 1. So this option is valuable in case of expected frequent change of configuration that impacts a lot of data.

To choose the right approach in a specific case, consider the following:

1. How frequently the configuration is expected to change?
   1. Frequent configuration changes leading every time to massive reindexation may be unacceptable.
   2. Configuration changes expected a few times in the store lifespan may not be worth additional complexity on the Storefront side and full reindexation may be better in this case.
2. Is it acceptable to have significant delay in data propagation after the configuration change?
   1. Changing Base URL may be not a big issue, especially if a redirect can be setup. So it may be acceptable to have URLs to be fully updated in a few hours.
   2. Changes in prices, on the other side, may not stand long delays.

### 3. Configuration that impacts UI representation of Storefront data

Example: include or exclude taxes in displayed prices. 

This kind of configuration has nothing to do with Storefront data itself, but is still necessary for the client to know how to display the data.

This section describes possible implementation options.

#### 3.1. Configuration as Part of Domain Service

UI Configuration is synced to the Storefront domain service, similarly to how the data is synced.
This may be done by means of a separate data flow as configuration usually doesn't change together with the data.

Is UI config included in the data (on the level of entities)?

![UI Configuration Propagation as Part of Domain Service](https://app.lucidchart.com/publicSegments/view/4805a8df-abe8-4605-96e2-20266b2f2876/image.png)

Pros:

1. Simplicity in case of Magento as back office. Same/similar data sync can be implemented for configuration
2. In-process call to storage for UI configuration

Cons:

1. Potential difficulties with UI configuration exclusion from deployment in case it is not needed, as it is part of the service 

#### 3.2. Domain-Specific Configuration Service 

Each domain service that requires UI configuration has a companion UI Configuration service.
Client application may call the specialized configuration service to get configuration it needs.
Some clients may not need all or part of UI configuration.

GraphQL serves as a single entry point for both data and UI requests to simplify client implementation and have more control of which APIs are publicly exposed.

![UI Configuration Propagation as Domain-Specific Configuration Service](https://app.lucidchart.com/publicSegments/view/c3c9f0a7-1780-416f-ab3f-caeeebb15680/image.png)

Pros:

1. More control of the deployment, scalability, technologies for the domain and config services.
2. More flexibility in the future
   1. Easier path for UI configuration disablement in case it becomes unnecessary. For example, if it turnes out that client apps want to have full ownership of UI configuration

Cons: 

1. Additional request to configuration service instead of in-process request to the storage.

#### 3.3. Single UI Configuration Service for All Domain Services

UI configuration settings are fed into a single UI Configuration service and read from it when necessary.

As, most likelly, all configuration options will look like key-value pairs, it might be reasonable to unify all of them under umpbrella of a single serfvice and so provide more uniformity of working with such configuration service.
This is a variation of option 3.2.

![UI Configuration Propagation as Single UI Configuration Service](https://app.lucidchart.com/publicSegments/view/cc77dc21-77ac-4eb4-b766-0f9afc8c11d5/image.png)

Pros:

1. More control of the deployment, scalability, technologies for the domain and config services. Assuming, all config services has the same technical requirements.

Cons: 

1. Additional request to configuration service instead of in-process request to the storage.

#### Integration with 3rd-party Data-Provider System

Data-prover system is a system that provides domain-specific data. For example, [PIM (product information management)](https://en.wikipedia.org/wiki/Product_information_management) system. 

In case of integration with 3rd-party data-provider systems, a separate data flow process is setup to sync configuration when it's changed.
If PIM doesn't provide all necessary configuration settings, a different management system may be used as the source.

Additional configuration may be provided by a separate client application dedicated to the configuration management.
Also, the additional configuration may be, in reality, the only source of configuration, in case PIM system doesn't provide such configuration.

In case of UI Configuration provided as part of the domain service:

![3rd-party PIM integration - UI Configuration as part of domain service](https://app.lucidchart.com/publicSegments/view/0e81c178-7480-41ca-8e7d-a37e6bdb6b6d/image.png)

In case of UI Configuration provided by a separate service:

![3rd-party PIM integration - UI Configuration as separate service](https://app.lucidchart.com/publicSegments/view/6051ef66-4b8f-44aa-9211-c6e7399bc9e2/image.png)
