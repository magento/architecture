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
   
To choose the right approach in a specific case, consider the following:

1. How frequently the configuration is expected to change?
   1. Frequent configuration changes leading every time to massive reindexation may be unacceptable.
   2. Configuration changes expected a few times in the store lifespan may not be worth additional complexity on the Storefront side and full reindexation may be better in this case.
2. Is it acceptable to have significant delay in data propagation after the configuration change?
   1. Changing Base URL may be not a big issue, especially if a redirect can be setup. So it may be acceptable to have URLs to be fully updated in a few hours.
   2. Changes in prices, on the other side, may not stand long delays.

### 3. Configuration that impacts UI representation of Storefront data

Example: include or exclude taxes in displayed prices. 

This kind of configuration has nothing to do with Storefront data itself, and so it is suggested to have a separate service that provides this configuration.
Client application may call the specialized configuration service to get configuration it needs.
Some clients may not need all or part of UI configuration.
GraphQL serves as a single entry point for both data and UI requests to simplify client implementation and have more control of which APIs are publicly exposed.

![UI Configuration Propagation](https://app.lucidchart.com/publicSegments/view/b7ec5763-eb23-48ac-9092-6b92821040fb/image.png)

Pros and cons of a separate service compared to configuration being part of the domain service are described bellow.

Pros:

1. More control of the deployment, scalability, technologies for the domain and config services.
2. Ability to exclude config service from deployment in case it becomes unnecessary for the specific use case.
   1. In case of multi-tenant deployment, of course it can't be done based on specific customer needs, but still may allow resources optimization in case specific merchant does not need UI configuration propagated from Magento.
   1. Even in case of multi-tenant deployment, it is easier to eliminate configuration feature (and free resources needed for it) if it's represented by a separate service.

Cons: 

1. Additional request to configuration service instead of in-process request to the storage.
