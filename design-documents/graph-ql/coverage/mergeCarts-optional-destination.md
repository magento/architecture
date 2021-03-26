# `mergeCarts` - Make `destination_cart_id` optional

## What

- Make the `destination_cart_id` argument optional in the `mergeCarts` mutation

## Why

This came up in a discussion with [@sirugh](https://github.com/sirugh) from the [`pwa-studio`](https://github.com/magento/pwa-studio) team.

When a user logs in (creates a new token), one of the first things the UI needs to do is merge the current guest cart (if items are present) into the customer account's cart.

If `destination_cart_id` is required, this requires 3 round trips:

1. Call for `Mutation.generateCustomerToken`
2. Call for `Query.cart` to get customer cart ID
3. Call for `Mutation.mergeCarts` to merge guest cart ID into customer cart

Because a customer can only have a single cart, and this API only works for authenticated users, `destination_cart_id` is an unnecessary requirement here. If we make it optional, the login + cart upgrade for UI can happen in a single request:

```graphql
# Mutations run serially, in-order. So `mergeCarts` will only execute
# if `generateCustomerToken` succeeds
mutation LoginAndMergeCarts($email: String!, $password: String!, $guestCartID: String!) {
    generateCustomerToken(email: $email, password: $password) {
        token
    }
    mergeCarts(source_cart_id: $guestCartID) {
        ID
    }
}
```

## Proposed Change
```diff
type Mutation {
-    mergeCarts(source_cart_id: String!, destination_cart_id: String!): Cart!
+    mergeCarts(source_cart_id: String!, destination_cart_id: String): Cart!
}
```