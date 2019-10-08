# Understanding of complex product types

## 1. Definitions
See [Simple Products](simple-products.md)
* Complex product is a virtual representation of multiple real SKUs at the storefront.
* To simplicity and data consistency, Magento gives such product ability to obtain virtual SKU.
There is a trick.
Configurable product creation wizard creates SKUs of Simple products as a concatenation of Configurable SKU and attributes values which were used its creation.
In fact, real purchases will use these generated SKUs because they are material products.
* Complex product may have similar attributes like name or URL to a simple product except for price and quantity.
* Complex product may have images or inherit images from the included products. 
* Currently, Magento does support three complex product types - Bundle, Grouped and Configurable.

## 2. Inventory Management

* Complex products are uncountable.
* Complex products do not have individual inventory records.
* But this does not mean that complex products do not have inventory statuses.
Depends on a product type and selected inventory system complex product inventory status will be represented as the function.

```
Example 1. Configurable product is in stock if exists at least one child product with "in stock" status.
```

### 2.1 Managing complex products thresholds.
* A merchant may want to restrict the selling of SKUs in the scope of the configurable product by specifying thresholds.
* This threshold does not have anything in common with warehouse SKU quantity but introduces a rule that limits the number of purchases.
* The threshold is optional for the complex products.

```
Example 2. Bundle product is in stock if all required child products have "in stock" status and a cumulative number of the bundle purchases did not reached a threshold number.
```

### 2.2 Exception/Limitation

* A merchant may want to specify a complex product which contains simple products with predefined quantity but sell this bundle with a single price.
This particular example should be implemented as a specific case of a simple product with options.  For this case, the inventory decrement has to be handled by the inventory management system. If an option has a quantity,
it always will be treated as a simple product by the order management system.

## 3. Product prices
See [Product Prices](products-prices.md)

* Complex products do not have a final prices instead they have price ranges.
* Price range describes minimum and maximum price of included SKUs or their combinations.
* Complex products could override prices for Simple SKUs if they assumed selling together in the scope of a bundle.
Such __bundle price__ will be accounted during option final price calculation.
If this price will lowest from the list it would be treated as the option final price.

## 4. ProductVariations and ProductOptions
See [Product options](product-options.md)

* Complex product has options and variations.
* The complex product contains variations which explain the relationship between the product and included SKUs.
* Each variation represents an option or options set which connects a real SKUs with buyer selection.
* Options explain how the complex product can be configured to chose a simple product or products.

```
Example 1. Selection of all options of the configurable products represents a single variation which represents a simple product with the selected attributes.
Example 2. Selection of an option of bundle product represents a real SKU.
Example 3. Grouped product can be represented as a set of options where each option may have a single value only.
```

### 4.1 Product variations examples

### 4.2 Mixing complex product options and simple product options
* The complex product can not have product customizable options because the complex product is not a real product, so this is uncertain in which way customizable options should be applied.  But the complex product may declare customizable options which will be applied to a simple product which was selected through the complex product if this applicable.  For instance, the configurable product that represents T-Shirt may declare a customizable option - print which will be applied to a particular simple product with colour and size.
TBD

## 5. Complex products in operations

* Complex product is more limited in storefront operations than a simple product.
* Complex product may appear in listings and has PDP.
* For checkout, we have to manipulate with a real SKU, which has an inventory record, certain price etc.
* Some of the scenarios may require mixed input, for instance, when a buyer not confident that he is going to purchase a product.
Example of such behaviour is wishlist functionality.
Such operations should expect to work not only with certain SKU but support virtual SKUs as well.

We can figure out a rule - with moving close to completing purchase our API should be more and more specific regarding the SKU we are passing as an argument.

See: [Add to cart/Add to wishlist documentation]()