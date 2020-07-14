## Use cases

### Render configurable option values available for selection on the product page

User navigates to the configurable product page. Option values available for selection are rendered on the page.

```graphql
{
  products(filter: {sku: {eq: "configurable-sku"}}) {
    items {
      description {
        html
      }
      name
      ... on ConfigurableProduct {
        configurable_options {
          attribute_code
          label
          values {
            id
            is_available_for_selection
            value_index
            label
            swatch_data {
              value
            }
            use_default_value
          }
        }
        configurable_options_selection_metadata {
          options_available_for_selection {
            attribute_code
            available_value_ids
          }
          media_gallery {
            url
            label
            position
            disabled
          }
        }
      }
    }
  }
}

```

The user makes a selection for the first option and the list of option values available for selection is updated for the remaining options. 
The images and videos relevant for the selection are also updated.

```graphql
{
  products(filter: {sku: {eq: "configurable-sku"}}) {
    items {
      ... on ConfigurableProduct {
        configurable_options_selection_metadata(
          selectedConfigurableOptionValues: ["hash from selected option value"]
        ) {
          options_available_for_selection {
            attribute_code
            available_value_ids
          }
          media_gallery {
            url
            label
            position
            disabled
          }
        }
      }
    }
  }
}
```

### User opens URL leading to configurable product page and configurable option selections are specified in the URL

In this case URL will have to be resolved first:

```graphql
{
  urlResolver(url: "http://magento.instance/configurable_product.html?configurable_options[0]=first-selection-hash&configurable_options[1]=second-selection-hash") {
    id
    type
  }
}
```

Then the product data along with available selections can be requested in a single query:

```graphql
{
  products(filter: {sku: {eq: "resolved-sku"}}) {
    items {
      description {
        html
      }
      name
      ... on ConfigurableProduct {
        configurable_options {
          attribute_code
          label
          values {
            id
            is_available_for_selection    
            value_index
            label
            swatch_data {
              value
            }
            use_default_value
          }
        }
        configurable_options_selection_metadata(
          selectedConfigurableOptionValues: ["hash from selected option value", "hash from another option value"]
        ) {
          options_available_for_selection {
            attribute_code
            available_value_ids
          }
          media_gallery {
            url
            label
            position
            disabled
          }
          variant {
            sku
          }
        }
      }
    }
  }
}
```

### Add to cart

After the user makes final selection, the corresponding simple product data becomes available and the product can now be added to cart.

```graphql
{
  products(filter: {sku: {eq: "configurable-sku"}}) {
    items {
      ... on ConfigurableProduct {
        configurable_options_selection_metadata(
          selectedConfigurableOptionValues: ["hash from selected option value", "hash from another option value"]
        ) {
          options_available_for_selection {
            attribute_code
            available_value_ids
          }
          media_gallery {
            url
            label
            position
            disabled
          }
          variant {
            sku
          }
        }
      }
    }
  }
}
```

Information about variant is taken from previous query result and used to add configurable product to cart.

### Render configurable option values available for selection on the category page

In case when the facet filter was used on the category page, for example to search "Red" shorts, it would be a good idea to display available sizes in "Red" for each product on the page. This can be achieved with the following query:

```graphql
{
  products(filter: {category_id: {eq: "shorts category ID"}}) {
    items {
      name
			sku
      ... on ConfigurableProduct {
        configurable_options_selection_metadata(
          selectedConfigurableOptionValues: ["hash from selected red color option"]
        ) {
          options_available_for_selection {
            attribute_code
            available_value_ids
          }
          media_gallery {
            url
            label
            position
            disabled
          }
        }
      }
    }
  }
}
```

### Extension points

`ConfigurableOptionsSelectionMetadata` type can be extended to support additional use cases, which are not currently supported by Magento like:
 - Price range for the variants based on configurable options selection
 - Low stock notification based on configurable options selection
