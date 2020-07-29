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

Magento configuration is divided into three categories, based on impact on Storefront services:

1. Admin configuration - impacts Admin Panel behavior
2. Data configuration - impacts data provided by Storefront
3. UI Configuration - impacts UI representation of Storefront data

### 1. Configuration that impacts Admin Panel behavior

Examples: admin ACLs that allow or deny parts of functionality for the admin user, reindex on update or by schedule.

This type of configuration has nothing to do with Storefront and should not be propagated to storefront, as well as should not be exposed via export API. 

### 2. Configuration that impacts data provided by Storefront

Example: Base Media URL is used for calculation of media image URLs returned by Storefront API.

This type of configuration should be propagated to Storefront in one of the following ways:

1. Pre-calculate final data on back office side and provide final result to the Storefront during synchronization. Minimize synchronization by updating only entities that have been really affected.
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
3. Do 3rd-party systems provide similar configuration?
   1. If 3rd-party systems don't have equivalent configuration, how will it be populated in the Storefront service? It might be better to avoid Magento-specific concepts to simplify integrations, and instead provide indexed data to the Storefront service.
   1. If a configuration option is pretty common among 3rd-party systems, it may make sense to reflect it in the Storefront service.

Expected consequences are described in the decision document for each case.
For example, time for configuration propagation in case reindexation is chosen, logic duplication/complexity in case configuration is propagated to Storefront application, performance impact for Storefront read API or for synchronization.

### 3. Configuration that impacts UI representation of Storefront data

Example: number of products on products listing page. 

This kind of configuration has nothing to do with Storefront data itself, but is still necessary for the client to know how to display the data.

This section describes possible implementation options.
Both options are possible and acceptable. The correct option should be selected based on the specific use case and client requirements. 
For other considered options see [older revision](https://github.com/magento/architecture/blob/48f2db6cc9f18a50b181b6a6b76cb0dbc81722cb/design-documents/storefront/configuraiton-propagation.md) of the document.

#### 3.1. Configuration is Responsibility of the Client

Client application (such as PWA) is responsible for the UI configuration, either hard-coded or by means of a service.

Justification: Storefront service is responsible for providing data, client applications may vary significantly and may just want to hard-code many of the options or take settings from different sources.
Until it is confirmed by the client developers (PWA, AEM, other teams) that UI Configuration API backed by Magento system configuration is necessary, Storefront efforts should not focus on supporting such API.

UI configuration is not synced from Magento Admin Panel to Storefront.

#### 3.2. Configuration is Provided by Magento Back Office API

Rely on current GraphQL API for providing UI Configuration.
No additional Store Front service is created to serve such configuration.
GraphQL entry point can proxy to the Magento Back Office GraphQL for simplicity in API usage.

Two options are possible, second expands on top of the first one.

Client holds knowledge about two sources (one for data and one for config) and handles requests.
This can be the first step.

![Service Configuration - Direct Back Office API](https://app.lucidchart.com/publicSegments/view/aeae7ddc-7a1f-4c94-88aa-2309dca63c05/image.png)

GraphQL handles requests routing to either Storefront domain service (for data) or to Magento Back Office GraphQL (for UI Config).

![Service Configuration - GraphQL Proxy](https://app.lucidchart.com/publicSegments/view/775a580e-fdb0-4bda-a532-eef04767396a/image.png)
