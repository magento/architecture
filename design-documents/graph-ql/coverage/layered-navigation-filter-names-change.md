**Overview**

As a Magento developer, I need to render layered navigation via GraphQL and have an understandable schema and field names.

Currently the schema for layered navigation is very specific to how you would render it in Luma and magento internal attributes. It doesn't make a lot of sense for GraphQl.

![Layered Navigation](layered-navigation-filter-names-change/layered_navigation.png)

**Use cases:**
- Reading relevant filters after a product search and displaying them
- The UI logic has to be able to loop through attributes and list them as sections
- Each attribute has multiple and at least one value to be rendered. A value of an attribute can then be used to be filtered by in a product search, by using it's ID value where it's label is only used for display purposes.

**Current schema:**

- Query and return value:
```graphql
filters {
      filter_items_count
      name
      request_var
      filter_items {
        items_count
        label
        value_string
      }
    }
```

**Problems in the current schema:**

- `filters->name` it's actually the filter label intended for display and rendering 
- `filters->request_var` it's actually the filter name used in product filtering. this is not a HTTP request anymore, it's graphql.
- `filters->filter_items->value_string` it's actually the comparison ID value that we use in product filtering. Indeed is a string type for now because all attributes are. We don't make that distinction and when we will the 'value_string' won't make any sense.

    It is used as:
    ```graphql
    products(
        filter: {
          request_var: {eq: "value_string"}
        }
    }
    ```

**Proposed schema:**
```graphql
filters {
      filter_items_count @deprecated
      options_count
      
      name @deprecated
      label
      
      request_var @deprecated
      field_name
      
      filter_items @deprecated {
        items_count
        label
        value_string
      }
      
      options {
        results_count
        label
        value
      }
    }
```

Response: 

```graphql
"filters": [
        {
          "options_count": 2,
          "label": "Price",
          "field_name": "price",
          "options": [
            {
              "results_count": 3,
              "label": "*-100",
              "value": "*_100"
            },
            {
              "results_count": 2,
              "label": "100-*",
              "value": "100_*"
            }
          ]
        },
        {
          "options_count": 6,
          "label": "Category",
          "field_name": "category_id",
          "options": [
            {
              "results_count": 5,
              "label": "Category 1",
              "value": "3"
            },
            {
              "results_count": 1,
              "label": "Category 1.1",
              "value": "4"
            },
            {
              "results_count": 1,
              "label": "Category 1.1.2",
              "value": "6"
            },
          ]
        },
      ],
```

**ProductInterface filters field area is impacted:** 

Example Query:
```graphql
{
  products(
    filter: {
      category_id: {eq:"3" }
      mysize: {eq:"17" }
    }
  	pageSize:10
    currentPage:1
    sort: {
      name :ASC
    }
  ) {
    items {
      sku
      name
    }
    filters {
      filter_items_count
      name
      request_var
      filter_items {
        items_count
        label
        value_string
      }
    }
    page_info {
      current_page
      page_size
      total_pages
    }
    total_count
    items {
      sku
      url_key
    }
  }
}

```
