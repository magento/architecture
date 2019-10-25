# GraphQL Wishlist Functionality

## Open Source edition
### Get customer wishlist id
In order to get customer wishlist data need to call `customer` query.
``` graphql
query {
  customer {
    wishlists {
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

### Get customer wishlist ids
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

## Manipulation with IDs

In order to perform operations with wishlist and wishlist items ID is required, because there is no other unique identifier:
 - wishlist `name` is available only for Commerce Edition
 - product `sku` in `WishlistItem` can be duplicated with different set of options

Since UUID for entities is not implemented yet, it is proposed  to use `base64_encode` of `entity_id` for now in order to use `ID` type.
Need to make BC change to WishlistItem.
``` graphql
type WishlistItem {
    id: ID #was Int
}
```
This value was previously used for display only, other operations like update or delete are not implemented yet.
