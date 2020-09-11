# Deprecate and replace `Mutation.createEmptyCart`

## What

- Mark `Mutation.createEmptyCart` as deprecated
- Add `Mutation.createGuestCart` as a replacement that includes `Cart` in its return type

## Why

This PR started from a Slack discussion about mutations and client-side caching. At one point [@sirugh](https://github.com/sirugh) mentioned:

> This makes me wonder if the createCart mutation should return a cart type instead of just the id

There are 2 problems that I'm aware of with the current schema returning just an `ID` instead of the `Cart` type:

1. Requires manual cache handling with libraries like Apollo, to link the `ID` back to a `Cart` object
2. Requires 2 round trips for the client to get cart data for a guest. Even though the cart is empty, it still has defaults the pwa-studio components will use to render an empty cart

During discussions, it was also noted that `createEmptyCart` has some behavior that's not clearly described by our schema. That is, If a customer is logged-in and has items in the cart, `createEmptyCart` returns an ID referencing a cart that is _not_ empty, which is extremely unintuitive. It was mentioned that the `pwa-studio` code [aliases this query](https://github.com/magento/pwa-studio/blob/38d652a4fbc797a4ac8ac0c3efa611003152c090/packages/venia-ui/lib/queries/createCart.graphql#L4) because of that.

## Intended Usage
Because the UI knows if it has a token for a customer, `Query.cart` should be used by the UI to get the cart for a logged-in user. If there is _not_ a customer token present, the UI should use `createGuestCart`, which will return a `Cart` object, the same return type as `Query.cart`.


## Proposed Changes

```diff
type Mutation {
+    createGuestCart(
+        input: CreateGuestCartInput
+    ): CreateGuestCartOutput @doc(description: "Create a new shopping cart")
    createEmptyCart(
        input: createEmptyCartInput
+    ): String @deprecated(reason: "Use `Mutation.createGuestCart`, or `Query.cart` for logged-in shoppers")
-    ): String @doc(description:"Creates an empty shopping cart for a guest or logged in user")
}

+input CreateGuestCartInput {
+    cart_uid: ID @doc(description: "Optional client-generated ID")
+}
+
+type CreateGuestCartOutput {
+    cart: Cart
+}

input createEmptyCartInput {
    cart_id: String
}
```
