# GraphQL Wishlist Functionality

## Open Source edition
### Get customer wishlist id
In order to get customer wishlist data need to call `customer` query.
``` graphql
query {
  customer {
    wishlist {
      id
      items {
        id
        product {
          sku
      }
    }
  }
}
```

If wishlist for customer does not exist, empty one will be created under the hood i.e. we assume that empty wishlist always exists for customer.

### Add product(s) to wishlist
Use `addProductsToWishlist` mutation in order to add products to wishlist with or without options. Need to specify `wishlist_id` and the `wishlist_items`.

### Update product(s) in wishlist
Use `updateProductsInWishlist` mutation in order to update products in wishlist with or without options. Need to specify `wishlist_id` and the `wishlist_items`.

### Remove product(s) from wishlist
Use `updateProductsInWishlist` mutation in order to remove products from wishlist by ID. Need to specify `wishlist_id` and the array of `wishlist_items_ids`.

### Remove wishlist
Use `removeWishlist` mutation in order to remove wishlist by `id`.

## Commerce edition
Use `createWishlist(name: String!):` in order to create named wishlist.

### Get multiple customer wishlists
``` graphql
query {
  customer {
    wishlists {
      id
      name
    }
  }
}
```
`customer` query will return array of all available customer wishlists.

### Get customer wishlist items by `id`
``` graphql
query {
  customer {
    wishlist(id: '42') {
      items {
        id
        product {
          sku
      }
    }
  }
}
```

## Solutions comparison

|  | Proposed | Alternative 1 | Alternative 2 |
| ------------- | ------------- | -------------| -------------|
| Open Source  | `wishlist` | `wishlists` | `wishlists`|
| Commerce  | `wishlists` and `wishlist(id: ID!)` | `wishlists(ids: [ID])` | `wishlists`|

### Proposed
`wishlist` field located under `Customer` type. Commerce edition will introduce new field `wishlists` to get an array of customer wishlists. `wishlist` field will be extended with `id` argument in order to able to retrieve info for single wishlist by ID.
#### Pros
- Semantically correct - in Open Source one wishlist will be returned, in Commerce - array of wishlist.
- It is possible to get info for the single wishlist by `ID` in Commerce edition.
#### Cons
- Client code has to be different for Open Source and Commerce editions. Missing `id` in Commerce will cause an error.
It is not a big problem because client has to know about Wishlist Name in Commerce edition anyway and has to adjust respectively.

### Alternative 1
`wishlists` field located under `Customer` type. In Commerce edition will be extended with `ids` argument in order to not to over-fetch and be able to retrieve info only for needed wishlists.

#### Pros
- Solves over-fetching problem.
- Client could work with both - Open Source and Commerce editions without changing the code.
#### Cons
- In Open Source edition array with one element will be returned instead of single item.
- Need to call `wishlists` at least once to get available wishlists `ids`.

### Alternative 2
`wishlists` field located under `Customer` type. In Open Source will be array with one element, in Commerce - multiple elements.
#### Proc
- Client could work with both - Open Source and Commerce editions without changing the code.
#### Cons
- In Commerce edition all wishlists will be returned, not just needed (over-fetching).
- In Open Source edition array with one element will be returned instead of single item.

## Manipulation with IDs

In order to perform operations with wishlist and wishlist items ID is required, because there is no other unique identifier:
 - wishlist `name` is available only for Commerce Edition
 - product `sku` in `WishlistItem` can be duplicated with different set of options

Since UUID for entities is not implemented yet, it is proposed to use casted to string `entity_id` for now in order to use `ID` type.
Need to make BC change to WishlistItem.
``` graphql
type WishlistItem {
    id: ID #was Int
}
```
This value was previously used for display only, other operations like update or delete are not implemented yet.
