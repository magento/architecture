# Understanding of complex product types

## 1. Definitions
See [Simple Products](simple-products.md)
* Complex product is a virtual representation of multiple real SKUs at the storefront.
* Currently, Magento does support three complex product types - Bundle, Grouped and Configurable.
* For the simplicity and data consistency a complex product has SKU.
In fact this SKU is virtual.
* Complex product may have all attributes that required to render product details page. 
* Complex product does not have attributes that describe physical product characteristics.

## 2. Inventory Management

* Complex products are uncountable.
* If *IMS* is synchronized with Catalog a complex products may have stock status.

## 3. Product prices
* Complex product does not have itself price.
* Complex product has a price range.
Price range has minimum and maximum prices.
Minimum price contains prices of the cheapest included simple product or sum of products combination. 
Maximum price contains prices of the most expensive included simple product or sum of products combination.

## 4. Complex product options 
See [Product options](products-options.md)

* Complex product has to have options which expose included simple products.
Such complex option does not have a price.
Price can be retrieved from the corresponding simple product through the variation.
* *Product variation* is a representation of one or many product complex options that represent a dedicated simple product.

* Complex product may expose simple options of included simple products.
* Complex product does not support simple options by itself.