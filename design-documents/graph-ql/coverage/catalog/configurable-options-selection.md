## Use cases

### Render configurable option values available for selection on the product page

User navigates to the configurable product page. Option values available for selection are rendered on the page.

```graphql
{
  configurableOptionsSelectionMetadata(configurableProductSku: "configurable-sku") {
    configurable_options_available_for_selection {
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
```

The user makes a selection for the first option and the list of option values available for selection is updated for the remaining options. 
The images and videos relevant for the selection are also updated.

```graphql
{
  configurableOptionsSelectionMetadata(
    configurableProductSku: "configurable-sku", 
    selectedConfigurableOptionValues: ["hash from selected option value"]
  ) {
    configurable_options_available_for_selection {
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
            value_index
            label
            swatch_data {
              value
            }
            use_default_value
          }
        }
      }
    }
  }
  configurableOptionsSelectionMetadata(
    configurableProductSku: "configurable-sku", 
    selectedConfigurableOptionValues: ["hash from selected option value", "hash from another option value"]
  ) {
    configurable_options_available_for_selection {
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
```

### Add to cart

After the user makes final selection, the corresponding simple product data becomes available and the product can now be added to cart.

```graphql
{
  configurableOptionsSelectionMetadata(
    configurableProductSku: "configurable-sku", 
    selectedConfigurableOptionValues: ["hash from selected option value", "hash from another option value"]
  ) {
    configurable_options_available_for_selection {
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
```

Information about variant is taken from previous query result and used to add configurable product to cart.

### Render configurable option values available for selection on the category page

Category and search result pages do not require the list of configurable option values available for selection. A list of all available options is enough.

It is still possible to make selection on the category page directly after it was renedered for the first time. The `configurableOptionsSelectionMetadata` query can be used in the same way as on product page.

### Extension points

`ConfigurableOptionsSelectionMetadata` type can be extended to support additional use cases, which are not currently supported by Magento like:
 - Price range for the variants based on configurable options selection
 - Low stock notification based on configurable options selection
