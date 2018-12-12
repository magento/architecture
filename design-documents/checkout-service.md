# Checkout Service

## Overview

This document describes design and communication of monolith application and checkout service.

## Design

Service should have it's own core_config_data table that would contain settings specific to the service. Service should expose API to modify settings in core_config_data. Admin panel should support editing settings in the local database (settings for BFF) and for individual services via remote calls.

Service should not have knowledge about stores/websites. However as service stores information about stores/websites within the quote stores/websites need to be validated. Service can request information about stores/website to use use for validation. Or validation can be omitted (recommended).

Directory data (countries, cities, states) should be moved to configuration (preferable) or moved to a separate service/exposed on BFF.

Operations that are currently per item (inventory requests) need to be done for multiple items to reduce number of calls.

Dependency on session should be removed. All methods should receive customer with customer groups as an argument.

Shared interfaces (product interfaces) need to be moved into API modules. For backwards compatibility we can export interfaces in the new API modules under old name. Shared configuration (configuration for extension attributes) need to be moved to a separate modules as well.

BFF should expose all quote (and other services we separate from monolith) API and work like proxy for all calls to maintain backwards compatibility for web API.

## Communication Between Monolith and Checkout Service

### Add to Cart Operation

Current communication for add to cart operation (has one of the largest amount of communication).

![Current state](checkout-service/add-to-cart-current-state.png)

Total 23 calls on Open Source. Commerce will have one more call. Customizations could increase number of calls.

#### Call Analysis

Catalog
* 2 calls to get list of products with different criteria
* 1 call to get final price for QTY
* 1 call to get product info
* 1 call to prepare item for cart
Possible optimizations: remove prepare item for cart call, 2 product list calls into 1 call, we will get 3 calls.

Inventory
* 2 calls to get different stock data
* 1 call to validate 
Possible optimizations: group stock data calls into 1, we will get 2 calls as a result.

Customer
* 7 calls to get customer data
Possible optimizations: pass as an argument of operation.

Quote/shipment
* 2 calls
Possible optimizations: need to investigate if we can eliminate 1 call, as a result we can potentially get 1 call.

Tax
* 2 calls
Possible optimizations: need to investigate if we can eliminate 1 call, as a result we will get 1 call.

Integration
* 1 call that can be removed.

#### Additional Optimizations

* Pass customer/group/addresses as an argument.
* Remove inventory checks, and check inventory to display whether product is stock when user gets to cart page. There are might be checks on qty increment to make sure that product being added with correct qty increment
* Totals may be calculated after adding product to cart in a separate operation
* Totals should not be recalculated on each cart load page, only when cart changes. It will should be recalculated when placing the order to make sure product data didn't change.
* Create service for totals calculation to reduce number of calls to BFF
* Group calls to the same service (request all needed data in the beginning); can lead to shared state

After we applying the following optimizations, we will have communication with checkout service as shown below
* Pass customer as an argument of add to cart operation
* Aggregate calls to catalog and inventory services Magento\CatalogInventory\Api\StockRegistryInterface::getStockItemBySku, Magento\CatalogInventory\Api\StockRegistryInterface::getStockStatusBySku need to combine into one call
* Get product info and get final price can be combined into one call
* Prepare item for cart logic should be eliminated, service should receive cart ready to add
* Create API that would allow to calculate totals for the quote

![First iteration](checkout-service/add-to-cart-first-iteration.png)


If we apply these additional optimizations, communication between services will look like this
* Have tax service return aggregated information that later can be used to display tax on product page or totals. Question: wouldn't this data be redundant in some cases?
* Move totals in it's own service: 1) reduce number of calls to monolith, 2) cache of requested data from tax, quote and sales service and as a result reduce amount of communication.


![Apply more decomposition](checkout-service/add-to-cart-apply-more-decomposition.png)

Cons
* More work for initial iteration as we would have to decouple more services with checkout


Pros
* Allows reduce calls to BFF
* Allows to avoid intermediate state of calculating totals in BFF and reduce amount of backwards incompatible changes

### Place Order Operation

Communication for place order operation looks similar. The only major differences are that we need to save payment, load quote multiple times and save quote. This flow potentially can be optimized.


![Apply more decomposition](checkout-service/place-order.png)


## Transactions
Currently Quote module is responsible for creating an order. Need to move this responsibility to order service (BFF after first iteration).

BFF will be responsible for loading quote and managing the transaction. If order placing failed, no additional transactions need to be rolled back after first iteration as all transactions being performed on BFF. If order placing successful, but  removing quote failed, we would need to retry removing quote instead of rolling back. Mechanism of retrying of failed operations is a general concern and should be discussed separately.


## Reporting
For reporting purposes, service should data to reporting service (BFF after first iteration).


## Open Questions
* Format of the data API receives is different from the format service contracts use in some cases
* Hydration/extraction for certain entities triggers initialization of resource models/queries to database, ProductInterface is one of the examples
* Code reuse (reusing the same modules). It's anti pattern, but maybe for us it's ok?
* When adding product to cart for product option of type file we need to upload the file. Should we upload this file to monolith and save reference with the quote? And in the future have shared file system for catalog services?
* Quote uses \Magento\Catalog\Model\Product\Type\AbstractType::prepareForCartAdvanced to prepare product for cart. BFF should pass product that already ready to be added to cart. Later can consider exposing this API on the catalog service.
* Quote need to get price for a product in certain QTY. We can't pass product with final price as price can depend on tier price. BFF need to expose API that would allow to retrieve final price for product in certain QTY.
* Product model is very heavy. It has dependency on resource model. In quote we don't need all of the data product data (we don't need images for instance). Quote should have different product interface with only fields it needs.
* A lot of fields are not part of the data interfaces 
* User should be able to upload file with the quote. Possible solutions (upload file on BFF and save link with the quote. In the future we could have a service that will be response/quote responsible on storing files)
