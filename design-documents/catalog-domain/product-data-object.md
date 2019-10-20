# Product schema 

Product entity described with a pseudo schema.
The purpose of this schema to determine the bare minimum set of data required for rendering all storefront operations.

## Product meta information

```json
{
  "type": "Product",
  "fields": [
    {"name": "sku", "type": "string"},
    {"name": "type", "type": "ProductType"},
    {"name": "attributeSet", "type": "string"} 
  ]
}
```
```json
{
  "enum": "ProductType",
  "values": [
    "simple",
    "virtual",
    "downloadable",
    "giftcard",
    "grouped",
    "configurable",
    "bundle"
  ]
}
```
* `sku` - (stock keeping unit) product natural identifier.
* `type` - product type
* `attributeSet` attribute set name.
Attribute set declares schema for product dynamic attributes.
For storefront scenarios attribute set may may specify more precise product taxonomy.

## Product identity

```json
{
  "type": "Product",
  "fields": [
    {"name": "id", "type": "uuid"} 
  ]
}
```
* `id` is a string representation of UUID.
`id` is the guaranteed unique product identifier for storefront scenarios.

## System attributes

Product has a set of default out of the box attributes.
This set present across all system installations.
```json
{
  "type": "Product",
  "fields": [
    {"name": "name", "type": "string"},
    {"name": "description", "type": "string"},
    {"name": "shortDescription", "type": "string"},
    {"name": "metaDescription", "type": "string"},
    {"name": "metaKeyword", "type": "string"},
    {"name": "metaTitle", "type": "string"},
    {"name": "weight", "type": "float"},
    {"name": "weightUnit", "type": "string"},
    {"name": "visibility", "type": "Visibility"}
  ]
}
```
```json
{
  "enum": "Visibility",
  "values": ["Not Visible Individually", "Catalog", "Search", "Catalog & Search"]
}
```
* `name` product name.
* `description` product description.
* `shortDescription` product short description.
* `metaTitle` *PDP* meta title
* `metaDescription` *PDP* meta description  
* `metaKeyword` *PDP* meta keyword
* `weight` - weight of physical item, makes sense for *Simple* products only. 
* `weightUnit` - unit for measurements for product weight 

## Products and Categories

```json
{
  "type": "Product",
  "fields": [
    {"name": "categories", "type": "string", "repeated": "true"}
  ]
}
```

## Url

Each product has URL.
URL is created as an function on product data and store configuration.
Basically this function can be described as
{domain}/{storeSuffix/}{urlKey}{. urlSuffix}.
 
```json
{
  "type": "Product",
  "fields": [
    {"name": "urlKey", "type": "string"},
    {"name": "url", "type": "string"}
  ]
}
```
* `urlKey` - root part of a canonical URL.
* `url` - product canonical URL.


## Calculated fields

Some of product attributes are stored as a result of function.
 
```json
{
  "type": "Product",
  "fields": [
    {"name": "displayable", "type": "bool"},
    {"name": "buyable", "type": "string"}
  ]
}
```

* `displayable` - is a calculated field that 
* `buyable` - 

## Prices and taxes
```json
{
  "type": "Product",
  "fields": [
    {"name": "taxClass", "type": "string"},
    {"name": "currency", "type": "string"},
    {"name": "priceRange", "type": "PriceRange"}
  ]
}
```
* `taxClass` - information about product tax class
* `currency` - information 
* `priceRange` - range of prices from minimum to maximum.

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
    {"name": "finalPrice", "type": "float"},
    {"name": "fixedTaxes", "type": "FixedTax", "repeated": "true"}
  ]
}
```
* `regularPrice` - base product price.
* `finalPrice` - final product price, lower or equal to `regularPrice`. 
* `fixedTaxes` - fixed product taxes. Example WEEE tax.
```json
{
  "type": "FixedTax",
  "fields": [
    {"name": "title", "type": "string"},
    {"name": "amount", "type": "float"}
  ]
}
```

## Product Option & Values
Product options explain how product can be configured before adding a product to cart.

```json
{
  "type": "ProductOption",
  "fields": [
    {"name": "id", "type": "uuid"},
    {"name": "required", "type": "bool"},
    {"name": "isMulti", "type": "bool"},
    {"name": "name", "type": "string"},
    {"name": "values", "type": "ProductOptionValue", "repeated": "true"}
  ]
}
```
```json
{
  "type": "ProductOptionValue",
  "fields": [
    {"name": "id", "type": "uuid"},
    {"name": "value", "type": "bool"},
    {"name": "minimalPrice", "type": "Price"}
  ]
}
```


## Inventory attributes

Catalog knows the bare minimum information about inventory,
only data that we need to render product.

```json
{
  "type": "Product",
  "fields": [
    {"name": "inStock", "type": "bool"},
    {"name": "lowStock", "type": "bool"}
  ]
}
```
* `isStock` - is product in stock.
* `lowStock` - is product in low stock.

## Type specific system attributes

```json
{
  "type": "Product",
  "fields": [
    {"name": "linksExist", "type": "bool"},
    {"name": "linksPurchasedSeparately", "type": "bool"},
    {"name": "linksTitle", "type": "string"}
  ]
}
```


## Product Images 

```json
{
  "type": "Product",
  "fields": [
    {"name": "image", "type": "Image"},
    {"name": "smallImage", "type": "Image"},
    {"name": "thumbnailImage", "type": "Image"},
    {"name": "swatchImage", "type": "Image"},
    {"name": "mediaGallery", "type": "Image", "repeated": "true"}
  ]
}
```
* `image` - base product image.
* `smallImage` - small product image.
* `thumbnailImage` - thumbnail image.
* `swatchImage` - image to represent a product with swatch functionality.
* `mediaGallery` - set of images to be shown at the *PDP*.

```json
{
  "type": "Image",
  "fields": [
    {"name": "url", "type": "string"},
    {"name": "label", "type": "string"}
  ]
}
```

## Product Videos
```json
{
  "type": "Product",
  "fields": [
    {"name": "videos", "type": "Video", "repeated": "true"}
  ]
}
```
* `videos` - set of product videos.
```json
{
  "type": "Video",
  "fields": [
    {"name": "url", "type": "string"},
    {"name": "label", "type": "string"}
  ]
}
```

## Dynamic Attributes

```json
{
  "type": "Product",
  "fields": [
    {"name": "attributes", "type": "Attribute", "repeated": "true"}
  ]
}
```
```json
{
  "type": "Attribute",
  "fields": [
    {"name": "attributeCode", "type": "string"},
    {"name": "value", "type": "string", "repeated": "true"}
  ]
}
```

## Product Variations

```json
{
  "type": "Product",
  "fields": [
    {"name": "variations", "type": "ProductVariation", "repeated": "true"}
  ]
}
```
```json
{
  "type": "ProductVariation",
  "fields": [
    {"name": "id", "type": "string"},
    {"name": "sku", "type": "string"},
    {"name": "minimalPrice", "type": "Price"},
    {"name": "buyable", "type": "bool"},
    {"name": "inStock", "type": "bool"},
    {"name": "lowStock", "type": "bool"},
    {"name": "selections", "type": "uuid", "repeated": "true"}
  ]
}
```




@@@@ where is store //    ?{"name": "storeViewCode", "type": "string"}
