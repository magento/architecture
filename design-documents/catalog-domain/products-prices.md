# Product prices

# Definitions

* Simple products should have a price.
* Complex products does not have a price but they know about prices of included simple products.
* All products have a range that represents borders of prices in which this product can be purchased.
* 
 
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
