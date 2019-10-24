# Product schema 

Product entity described with a pseudo schema.
The purpose of this schema to determine the bare minimum set of data required for rendering all storefront operations.

## Product identity

```json
{
  "type": "Product",
  "fields": [
    {"name": "id", "type": "string"},
    {"name": "sku", "type": "string"}
  ]
}
```
* `id` is an immutable identifier that represent a product in unique way. 
* `sku` - (stock keeping unit) product natural identifier.

## Product meta information

Meta information defines product structure.
```json
{
  "type": "Product",
  "fields": [
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

* `type` - product type defines type specific attributes and options that can be associated with a product.
* `attributeSet` attribute set name.
Attribute set declares schema for product dynamic attributes.
For storefront scenarios attribute set may may specify more precise product taxonomy.


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
* `weight` - weight of physical item, makes sense for *Simple* products only. 
* `weightUnit` - unit for measurements for product weight 
* `visibility` - tells how the product can be shown if enabled.


## Dynamic/EAV Attributes

Product may have user-defined custom attributes.
Set of attributes may vary and depends on an `attributeSet` assigned to a product.

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

* `attributeCode` - attribute code.
* `value` - array of values that assigned to the attribute.
## Product flags

Explains allowed operations. 
```json
{
  "type": "Product",
  "fields": [
    {"name": "displayable", "type": "bool"},
    {"name": "buyable", "type": "bool"}
  ]
}
```
* `displayable` answers on question can this product be displayed?
* `buyable` answers on question can this product be purchased?

## SEO attributes

### Product URL
Each product has URL.
URL is created as an function on product data and store configuration.
This function can be described as `{domain}/{storeSuffix/}{urlKey}{urlSuffix}`.
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

### Meta attributes
```json
{
  "type": "Product",
  "fields": [
    {"name": "metaDescription", "type": "string"},
    {"name": "metaKeyword", "type": "string"},
    {"name": "metaTitle", "type": "string"}
  ]
}
```
* `metaTitle` *PDP* meta title
* `metaDescription` *PDP* meta description  
* `metaKeyword` *PDP* meta keyword


## Products and Categories

```json
{
  "type": "Product",
  "fields": [
    {"name": "categories", "type": "string", "repeated": "true"}
  ]
}
```
* `categories` - list of categories(category IDs) which contain the product.

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
* `taxClass` - information about product tax class.
* `currency` - currency code.
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


## Product Option & Values
Product options explain how product can be configured before adding a product to cart.

```json
{
  "type": "ProductOption",
  "fields": [
    {"name": "id", "type": "uuid"},
    {"name": "isRequired", "type": "bool"},
    {"name": "isMulti", "type": "bool"},
    {"name": "name", "type": "string"},
    {"name": "values", "type": "ProductOptionValue", "repeated": "true"}
  ]
}
```
* `id` - option identifier.
* `isRequired` - is this options required.
* `isMulti` - is the option allows multiple choices.
* `name` - option name.
* `values` - set of option values.

```json
{
  "type": "ProductOptionValue",
  "fields": [
    {"name": "id", "type": "uuid"},
    {"name": "value", "type": "string"},
    {"name": "minimalPrice", "type": "Price"}
  ]
}
```
* `id` - value id.
* `value` - option value.
* `minimalPrice` - option real price. 

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

## [TBD]Product Videos
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

## Product Variations

* *Product variation* is a representation of one or many product options that represent a dedicated simple product.

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
    {"name": "productId", "type": "string"},
    {"name": "minimalPrice", "type": "Price"},
    {"name": "buyable", "type": "bool"},
    {"name": "inStock", "type": "bool"},
    {"name": "lowStock", "type": "bool"},
    {"name": "defaultQty", "type": "float"},
    {"name": "selections", "type": "uuid", "repeated": "true"}
  ]
}
```
## [TBD]Type specific system attributes

Some products may have attributes specific to type.

### Downloadable Product

```json
{
  "type": "Product",
  "fields": [
    {"name": "linksExist", "type": "bool"},
    {"name": "linksPurchasedSeparately", "type": "bool"},
    {"name": "samples", "type": "DownloadableSample", "repeated": "true"}
  ]
}
```
```json
{
  "type": "DownloadableSample",
  "fields": [
    {"name": "link", "type": "string"},
    {"name": "name", "type": "string"}
  ]
}
```

### GiftCart

