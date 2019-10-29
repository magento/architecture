# Product prices

# Definitions

* Simple products should have a price.
* Complex products do not have a price but they know about prices of included simple products.
* All products have a range that represents borders of prices in which this product can be purchased.
 
```json
{
  "type": "Product",
  "fields": [
    {"name": "priceRange", "type": "PriceRange"}
  ]
}
```
```json
{
  "type": "PriceRange",
  "fields": [
    {"name": "minimumPrice", "type": "Price"},
    {"name": "maximumPrice", "type": "Price"}
  ]
}
```
```json
{
  "type": "Price",
  "fields": [
    {"name": "regularPrice", "type": "float"},
    {"name": "finalPrice", "type": "float"}
  ]
}
```

* `regularPrice` - original product price.
* `finalPruce` - price after discounts.

## Simple product
* `minimumPricice` - the lowest price a simple product can be purchased including required options.
* `maximumPrice` - the most expensive price including all options.

## Complex product
* `minimumPricice` - price of the cheapest product in group or products combination. 
* `maximumPrice ` - price of the most expensive product in group or products combination. 
