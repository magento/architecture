# Mutation Error Design

Mutations are _the_ single way to modify application state in a GraphQL API. Mutations are where the large majority of our errors are going to happen, as a result of changing some underlying data.

Today, the Magento 2 GraphQL schema does not do a great job of describing what could go wrong as a result of running a mutation. We should fix that!

## Example of the problem

Take this simplified schema to add an item to a shopper's cart:

```graphql
type Mutation {
    addItemToCart(sku: String!, quantity: Float): Cart
}
```

This schema only describes the happy path of adding an item to the cart. But some things can go wrong when adding an item to the cart, and we know what many of those things are:

- Item went out of stock since the product page was loaded
- Merchant disabled the product since the product page was loaded
- Cart hit the max quantity allowed of item per customer. Full qty selected was not added, but some were
- Extensions can add additional rules that would limit adding an item to the cart

Now put yourself in the shoes of a UI developer: how do you know all the error states your UI needs to cover? For most of the Magento 2 mutations today, you would need run the mutation itself (or dig through PHP) to see what will be returned in the `errors` key of the GraphQL response:

```js
{
    "data": {
        "cart": {
            // cart data here
        }
    },
    "errors": [{
        "message": "Item 'cool-hat' not added to cart (Out of Stock)",
        "path": ["addItemToCart"]
    }]
}
```

This has some problems:

1. It's unreasonable to expect a UI developer to exercise every possible input to a resolver to find the error states manually
2. These errors are not enforced by the schema, so they're not versioned with the schema.
3. If the UI wants to customize the message for a specific error state, they'd have to match on the exact response string, because there's no concept of error codes
4. Internationalization becomes challenging because the error states/strings are not known ahead of time

In the world of headless UIs, it's going to be critical for our errors to be discoverable, translatable, and versioned

## Solution: Design Mutation errors into the schema

If we move our known error states for various mutations into the schema design itself, we'll get a few benefits:

1. Versioning of errors
2. Discoverability of error states for UI devs

### Example: Fixing our `addItemToCart` mutation

In the previous example of `addItemToCart`, we had the following schema:

```graphql
type Mutation {
    addItemToCart(sku: String!, quantity: Float): Cart
}
```

Let's design some known error states directly into the API

```graphql
type Mutation {
    addItemToCart(sku: String!, quantity: Float): AddItemToCartOutput
}

type AddItemToCartOutput {
    # can still get the entire state of the cart post-mutation
    cart: Cart
    # can directly access errors that a shopper should be notified about
    add_item_user_errors: [AddItemUserError!]!
}

type AddItemUserError {
    # if a UI has their own text for message, they can just
    # not use this field, but serves as a descriptive default
    message: string!
    type: AddItemUserErrorType!
}

# This enum is just an example. Non-exhaustive, and of course
# not all errors we'd want a specific type for
enum AddItemUserErrorType {
    OUT_OF_STOCK
    MAX_QTY_FOR_USER
    NOT_AVAILABLE
}
```

With this new schema, we get the following query and response:

```graphql
mutation {
    addItemToCart(sku: "abc", quantity: 1) {
        cart {
            items {
                # select item fields
            }
        }

        add_item_user_errors {
            message
            type
        }
    }
}
```

```js
{
    "data": {
        "cart": {
            // cart data here
        },
        "add_item_user_errors": [{
            "message": "Item 'cool-hat' not added to cart (Out of Stock)",
            "type": "OUT_OF_STOCK"
        }]
    }
}
```
## Should we still use the `errors` key at all?

Yes! _But_, we should consider the `errors` key to be of similar use to the `catch` clause when using `try/catch` in a programming language.

The `errors` key should be for _exceptional_ circumstances, not for describing well-known states in an ecommerce app. It's unlikely you'd write this application code on the backend:

```javascript
try {
    var product = addItemToCart('sku');
    return product;
} catch (error) {
    if (error.code === 'OUT_OF_STOCK') {
        //
    } else if (error.code === 'MAX_QTY_FOR_USER') {
        //
    } else {
        // something else failed, maybe the DB connection?
    }
}
```

Instead, you would likely have your `addItemToCart` function return something that describes the various possible results of the operation. 

## Open Questions

1. Should we be consistent with the field name that represents these first-class errors? `user_errors` vs something like `add_to_cart_user_errors`
2. Should there be a basic interface that mutations should implement to enforce the pattern of returning a list of user errors?
