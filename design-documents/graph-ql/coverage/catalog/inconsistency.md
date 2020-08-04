## Problem 
There are a lot of inconsistency in current GraphQL schema for different entities.

To follow  the principle of least astonishment it's good to have one approach for similar components.

The goal of this document to highlight the problem and discuss possible solutions

## Examples

Here are few examples:

1. Data Filtering and return structure
```graphql

query {
  products(filter: {category_id: {eq:"3"}}) { # used filter object for provide different filters
    items {
      id
      name
    }
  }
}


query {
  category(id: 3) { # can be filtered only by id
      id
      name
  }
}
```

2. Complex type

```graphql
query {
  products(filter: {category_id: {eq:"3"}}) {
    items {
      description { 
         html     # Return description as complex object
      }
    }
  }
}



query {
  category(id: 3) {
    description   # Return description as plain value
  }
}


query {
  products(filter: {category_id: {eq:"3"}}) {
    items {
      small_image {
        url          # Return small_image as complex object
        label
      }
      swatch_image   # Return swatch_image as plain value
    }
  }
}


```

3. Data duplication


```graphql

query {
  products(filter: {category_id: {eq:"3"}}) {
    items {
      small_image {
        url                   # Return "small_image" attribute as coplex object
        label
      }
      media_gallery_entries {
        id                    # Return all images related to product
        label
        file
        media_type
        types
      }
    }
  }
}


```


## Solution

Design a uniform approach for the same schema component, deprecate old implementations and provide new one in a major release 
