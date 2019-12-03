 # Configurable products options limiting and variations filtering. 
 ## Problem statement   
 
Pulling all configurable products data in one query is a valid use case, for one product, that can result in quite a big payload if we have 2 options with each some hundreds of values.
This problem becomes multiplied by the number of products we're trying to query at a time, with related products, or category page. For simplicity a 9 products at a time can result in a payload of 30mb, consisting of 1.5 million lines of data and some 40 seconds wall time.
The difference on the PDP page and multiple products rendering, is that for multiple, you don't need as much detail.

Only data as a developer needed to display multiple configurable products is two or three sizes/colors and the total count plus the individual simple products with prices, images and minor details about them.

Currently we can't use this Id in Product to filter by, because of how Search API works, also CMS has another unique identifier that can be used.
Also we want not to expose database autoincrement ID in graphql in the future because of replacements with UUID.

 ## Proposed solution

Introduction of limiting the `configurable_options` from the `ConfigurableProduct` type. This can be done by adding an optional parameter to keep schema backward compatible.

```graphql
... on ConfigurableProduct {
        configurable_options {
            id
            attribute_id
            label
            position
            use_default
            attribute_code
            values(limit: 3) {
                value_index # usually value of the dropdown attribute
                label
                store_label
                default_label
                use_default_value
            }
            total_values # new field proposed
            product_id # parent id (configrable product id)
        }
   }
```

```json
{
  # ...
      "configurable_options": [
        {
          "id": 27,
          "attribute_id": "232",
          "label": "Color",
          "position": 0,
          "use_default": false,
          "attribute_code": "color",
          "values": [
            {
              "value_index": 278,       # pair 1.1
              "label": "RED"
              # ...
            },
            {
              "value_index": 280,       # pair 1.2
              "label": "GREEN"
              # ....
            },
            {
              "value_index": 282,       # pair 1.3
              "label": "BLUE"
              # ....
            }
          ],
          "product_id": 1851
        },
        {
          "id": 28,
          "attribute_id": "233",
          "label": "SIZE",
          "position": 1,
          "use_default": false,
          "attribute_code": "size",
          "values": [
            {
              "value_index": 279,       # pair 2.1
              "label": "S"
              # ...
            },
            {
              "value_index": 281,       # pair 2.2
              "label": "M"
              # ....
            },
            {
              "value_index": 282,       # pair 2.3
              "label": "L"
              # .....
            }
          ],
          "product_id": 1851
        }
        # ...
}
```

Note the limit 3, we can pull information as a regular sql limit, or most popular colors, or other data.

 #### Intended functionality
 
 #####1st pass query - querying the options and get the value_index array 
- Limit will cut down query time dramatically and just display -- `RED, BLUE, YELLOW` +200 --
- This query can be done as a first pass so the variants won't be even loaded.

 #####2nd pass query - querying the variants with value_index filtering from the first limit
- Using the `RED, BLUE, YELLOW` value indexes, but not all 200 ones filter the variants and get them all in the second query.
- Each value_index refers to an attribute value that in fact refers to a simple product from which we need basic info like name, price, image.

```graphql
... on ConfigurableProduct {
                variants(value_indexes: {in: [ [278,279], [280,281], [282,283] ... ]}) { #equal to [RED, S] [GREEN, M], [BLUE, L]
                    product { # single product that coresponds to 1 pair of attributes
                       name
                       image
                       price
                       # ....
                    }
                    attributes { #array for the pair of the attributes
                        label
                        code
                        value_index
                    }
                }
   }
```

```json
{
 # ...
          "variants": [
            {
              "product": {
                # ....
              },
              "attributes": [
                {
                  "label": "option 1.1",
                  "code": "attribute1",
                  "value_index": 278
                },
                {
                  "label": "option 1.2",
                  "code": "attribute2",
                  "value_index": 281
                }
              ]
            }
          ]
  # ...
}
```
 ## Alternatives

#### Schema change
Related data is not nested, it is queried separately. If variants would be child of values of options, just limiting would be enough, because the results would be spanned down the chain.
Also same problem is on the variants product that should be inside the attributes. One value_index maps ones to one to the product.


```graphql
... on ConfigurableProduct {
        configurable_options {
            id
            attribute_id
            label
            position
            use_default
            attribute_code
            values(limit: 3) {
                value_index # usually value of the dropdown attribute
                label
                store_label
                default_label
                use_default_value
                attribute_values {
                        label
                        code
                        value_index
                        product {
                           name
                           image
                           price
                           # ....
                        }
                  }
            }
            total_values # new field proposed
        }
   }
```

#### Exposing product id/sku instead of the whole product interface

The fact that product is exposed as an object adds a lot of extra processing, since we already know the id of the child product, and we have to query all other details about it and map it to the attribute value.
Exposing just product id and using a second pass query with those ids would be better. The problem here is that they are not visible so we still have to filter them by id on the variatns

1st query
```graphql
... on ConfigurableProduct {
        variants {
            product_id # not querying the whole product will also reduce query time but can still return quite some data.
            attributes { #array for the pair of the attributes
                label
                code
                value_index
            }
        }
   }
```

Note: An optimization can probably be made in case we query product { id } instead of adding product_id as a new field.

2nd query
```graphql
... on ConfigurableProduct {
        variants(uuid: {in: [1,2,3..]}) {
            product {
               name
               image
               price
               # ....
            }
            attributes { #array for the pair of the attributes
                label
                code
                value_index
            }
        }
   }
```
# Limiting the variants and options at the same time, so it would return unique combinations and pass those combinations automatically from options to variants
```graphql
... on ConfigurableProduct {
     configurable_options(limit:3) {
       # ...
     }
     variants(limit:3) {
       # ...
    }
}
```

This last schema only works if limit is set on both configurable_options and variants. Otherwise we won't know what pairs to pass from one to another.
