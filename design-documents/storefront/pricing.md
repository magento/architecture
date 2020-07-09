## Problem statement

The final price of the product in Magento monolith depends on multiple variables such as current customer group, current website, qty of items in the shopping cart and current date/time. Magento monolith calculates all possible permutations of prices in advance and store them in `price index`. These calculations are expensive and may not be done in reasonable time for large catalogs.

Few problematic use cases from real merchants:

- Each customer in the system has unique prices for very few products. Per-customer prices are result of a physical contract with pricing included.
The system has 20,000 customer groups, 10,000 products, promotions are excluded from the system. Magento will generate 200,000,000xCOUNT_OF_WEBSITES records to handle all possible combinations for this merchant. For one website it will consume 17,8 GB of space for data and 22.3 GB for index(40 GB total). Due to external promotion system, prices are synchronized periodically. The synchronization process consumes a lot of resources and time and eventually space.

- Customer groups are used for many things, but they are also a multiplier for prices
The system has 56 stores, 58 customer groups, 29,000 products, most customer groups are not global and used on one website only. Existing `price index` contains 25,000,000 records. The reindex process takes more than 7 hours. Customer groups are also used for promotions, cms content, product availability, B2B company, tax status and of course pricing. The real count of the prices is 26 times smaller. Potentially, reindex process may take 16 minutes.

The impact form cases describe above will be doubled(or even tripled) if we introduce a new storefront service with existing index structure inside. Thus, we need some other way to work with prices in storefront.

## Glossary
- Customer group - allocate each customer to a group and control store behaviour(catalog restrictions, pricing, discounts, B2B, CMS, payments, shipping, etc) according to which group a customer belongs to  
- Website, Store View - abstract Magento scopes - https://docs.magento.com/user-guide/configuration/scope.html . Magento modules and third-party extensions may utilize this scope as they want.
- Base price - the initial product price. Different values could be set per website in Magento monolith.
- Tier price - have two meanings `customer-specific price` and `volume based price`.  
- Special price - time based price for the product. Original price is strikedout in the UI. (e.g. was ~~$100.00~~, now $99.00) 
- Catalog Rule - Magento functionality that allows to apply the same discount to multiple products


## Goals

- Support up to 5,000,000 products and 15,000 customer groups
- Establish efficient sync of prices between Magento monolith and Magento storefront
- Reduce the size of pricing index
- Provide reliable support for personalized prices

## Solution

All price dimensions aren't used exclusively for pricing: 
multiple `websites` may have the same price, but represent different web domains; `customer group` could represent the company in b2b scenarios or class of customer service; `product` may have different pricing on one website only.
The solution is to reuse data where possible and make separation of dimensions, so `websites` or `customer groups` which are not a part of pricing will not trigger `price index` explosion.

### Price books

The `price book`(price list) is a new entity that holds a list of product's prices for a sub-set of catalog. 
The `price book` is a non-scoped entity, but it may hold information about linked customers, websites, etc. 
Instead of direct lookup in price index by customer_group, website and product, we will detect the customer's price book first, then we will extract two price books from the index: default and current `price book`. 
The resulting product price will be the value from current price book if it's exists, otherwise the product price will be extracted from default price book:

![Price books diagram](pricing/pricebooks.png)

### Default price book

A `default price book` is a predefined system price book that contains `base prices` for __all__ products in the system. User-defined `price books` may contain only sub-set of products. Default `price book` should be used as fallback storage if the price for a specific product doesn't exist in other resolved pricebook.


### Customer tags instead of customer groups

Magento monolith uses `customer groups` for customer segmentation globally. There is one-to-many relation between customer groups and customers, 
so customer could be a member of excatly one group only. However, in modern world, each customer could be a 
member of different groups based on current behavior. For example, pricing system works with wholesale and regular buyers,
but recommendation system works with different groups of customers which are based on gender, age, ML-generated groups, etc.

In order to provide more flexibility in customer segmentation, we may introduce many-to-many. Also, having `customer groups` 
which are not bound to pricing functionality make them looks like a regular tags. Thus, we may also rename them to tags:

![Price books diagram](pricing/customer-tags.png)

### Complex products support

The `minimum prices` of complex products calculated based on variation's prices, variation's availability and variation's stock.
Having variations as a separate products makes `minimum price` and `maximum price` dependent on products which may not
be visible for the current group or in a current catalog. Example: configurable product contains variation #1 - price $10,
 2 - price $9 and 3 - price $12. Let's imagine that variation #2 is visible for people with "VIP access" only,
then `minimum price` of configurable product for basic access will be $10, for "VIP access" - $9.

This happens because parent product and variation are separate products which could be assigned to different access lists 
and price books. In order to mitigate this issue products should be isolated, so product options fully define complex products.

The case from example above could be handled by two independent configurable products with different set of variations.

Details will be provided in the separate proposal.

### Synchronization with monolith

One of the goals of `price books` is to speedup reindex process. Existing reindex process lives in the monolith and 
prepares the prices for luma storefront exclusively. The data produced by this indexer is useless for the new storefront, 
so old indexer should be disabled for the installation which uses the new storefront exclusively.





## Scenarios

#### Simple

- Admin sets a `base price` for the `simple` product
- `product listing`, `PDP` and `checkout` scenarios contain the `base price`
- Customer or guest are able to buy the product for the `base price`

#### Customer specific price

- Admin sets a `base price` for the `simple` product
- Admin sets a `customer-specific price` for the same product
- `product listing`, `PDP` and `checkout` scenarios contain `base price` for guests and `customer-specific price` for selected customer
- Customer is able to buy the product for the `customer-specific price`

#### Complex product pricing

- Admin creates a `configurable` product and assign different `base prices` to variations
- `product listing` scenario contains the minimum price of the variations
- `PDP` scenario contains the minimum price of the variation and the price of currently selected variation
- `product listing` scenario contains the price of currently selected variation
- Customer or guest are able to buy the product variation for the price of selected variation

#### Special prices

- Admin sets a `base price` for the `simple` product
- Admin sets a `special price` for the same product for the current date
- `product listing`, `PDP` and `checkout` scenarios contain `special price`
- Customers and guests are able to buy the product for the `special price`

