# Catalog Service

## Purpose of Catalog Service
Catalog has a single purpose of returning product information that was intended to be shown at the storefront, both product listing or PDP.
Basically saying Catalog will answer on questions:
* How the product should look like to satisfy storefront requirements.
* How client of the Catalog can request products.

## Usage examples

### Direct call
Catalog service can be involved directly through the gateway.
![Dirrect call](images/catalog-usage-01-01.png)

A good example of such services is sales channels integrations that have to synchronize across connected channels.

### Routing
Clients who want to retrieve product data by URL.
![Routing](images/catalog-usage-01-02.png)

Literally any application.


### Listings and search

Category page, search results, layered navigation.

![Routing](images/catalog-usage-01-03.png)


## Performance requirements

Different scenarios may require different sets of attributes.


## Catalog Service DSL

See [Product Data Object](product-data-object.md)
```json
{
  "query": "products",
  "fields": [
    {
      "field": "id",
      "filter": [
        "1652b2fe-f758-11e9-8f0b-362b9e155667",
        "1652b588-f758-11e9-8f0b-362b9e155667",
        "1652b7c2-f758-11e9-8f0b-362b9e155667",
        "1652b8f8-f758-11e9-8f0b-362b9e155667",
        "1652bb00-f758-11e9-8f0b-362b9e155667",
        "1652bc40-f758-11e9-8f0b-362b9e155667",
        "1652be5c-f758-11e9-8f0b-362b9e155667",
        "1652bf88-f758-11e9-8f0b-362b9e155667",
        "1652c186-f758-11e9-8f0b-362b9e155667"
      ]
    },
    {"field": "name"},
    {"field": "sku"},
    {
      "field": "priceRange",
      "fields": [
        {
          "field": "minimumPrice",
          "fields": [
            {"field": "finalPrice"}
          ]
        }
      ]
    },
    {
      "field": "attributes",
      "fields": [
        {
          "field": "attributeCode",
          "filter": [
            "attributeCode1",
            "attributeCode2",
            "attributeCode3"
          ]
        },
        {"field": "values"}
      ]
    },
    {"field": "options"}
  ]
}
```
alternative formats
```xml
<query name="products">
    <field name="id" />
    <filter name="id">
        <filter>1652b2fe-f758-11e9-8f0b-362b9e155667</filter>
        <filter>1652b2fe-f758-11e9-8f0b-362b9e155667</filter>
        <filter>1652b2fe-f758-11e9-8f0b-362b9e155667</filter>
        <filter>1652b2fe-f758-11e9-8f0b-362b9e155667</filter>
        <filter>1652b2fe-f758-11e9-8f0b-362b9e155667</filter>
    </filter>
    <field name="name" />
    <field name="sku" />
    <document name="priceRange" >
        <document name="minimumPrice">
            <field name="finalPrice" />
        </document>
    </document>
    <document name="attributes">
        <filter name="attributeCode">
            <filter>attributeCode1</filter>
            <filter>attributeCode2</filter>
            <filter>attributeCode2</filter>
        </filter>
    </document>
</query>
```
or
```graphql
query(name: product, filter: { id: [1,2,3], name: 'sfd' }) {
    id,
    name,
    sku,
    priceRange: {
        minimumPrice: {
                finalPrice
         }
        },
        attributes(attributeCode: [attributeCode1, attributeCode2, attributeCode2])
        {
        }
}
```
## Example DSL to code transformation

```php
<?php

/**
 * Class Field
 */
class Field
{
    /**
     * @var string
     */
    private $name;

    /**
     * @var array
     */
    private $fields;

    /**
     * @var array
     */
    private $filter;

    /**
     * Field constructor.
     *
     * @param string $name
     * @param array $fields
     * @param array $filter
     */
    public function __construct(
        string $name,
        array $fields = [],
        array $filter = []
    ) {
        $this->name = $name;
        $this->fields = $fields;
        $this->filter = $filter;
    }

    /**
     * @return string
     */
    public function getName(): string
    {
        return $this->name;
    }

    /**
     * @return array
     */
    public function getFields(): array
    {
        return $this->fields;
    }

    /**
     * @return array
     */
    public function getFilter(): array
    {
        return $this->filter;
    }
}

/**
 * Class ProductDataObject
 */
class ProductDataObject {}
/**
 * CatalogInterface
 */
interface CatalogInterface
{
    public function getProducts(\Field $field) : \ProductDataObject;
}

class Catalog implements CatalogInterface {}
```


```php
<?php
$query = new \Field(
    'products',
    [
        new \Field(
            'id',
            [],
            [
                "1652b2fe-f758-11e9-8f0b-362b9e155667",
                "1652b588-f758-11e9-8f0b-362b9e155667",
                "1652b7c2-f758-11e9-8f0b-362b9e155667",
                "1652b8f8-f758-11e9-8f0b-362b9e155667",
                "1652bb00-f758-11e9-8f0b-362b9e155667",
                "1652bc40-f758-11e9-8f0b-362b9e155667",
                "1652be5c-f758-11e9-8f0b-362b9e155667",
                "1652bf88-f758-11e9-8f0b-362b9e155667",
                "1652c186-f758-11e9-8f0b-362b9e155667"
            ]
        ),
        new \Field(
            'priceRange',
            [
                new \Field(
                    'minimumPrice',
                    [
                        new \Field('finalPrice')
                    ]
                )
            ]
        ),
        new \Field('sku'),
        new \Field(
            'attributes',
            [
                new \Field(
                    'attributeCode',
                    [
                        "attributeCode1",
                        "attributeCode2",
                        "attributeCode3"
                    ]
                ),
                new \Field('values')
            ]
        ),
        new \Field('options')
    ]
);

$catalog = new \Catalog();
$products = $catalog->getProducts($currency, $locale, $store, $query);
```
