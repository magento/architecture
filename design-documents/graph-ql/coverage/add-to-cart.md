

Add simple/virtual/downloadable/giftcard product to cart
```graphql
type Mutation {
    addToCart(cartItems: [CartItem]): Cart
}

input CartItem {
    cartItemId: ID!
    sku: String!
    qty: Float!
    options : [ProductOption!]
}

input ProductOption {
    optionId: ID
    valueId: ID
}
```

Global registry of options excluding configurable and bundle options

Option(Value) Id desired state is UUID 
backward compatible way to achieve the same behaviour
`base_64_encode('option_type', option_id)`


Add configurable product to cart
```graphql
type Mutation {
    addToCart(cartId: ID, cartItems: [CartItem]): Cart
}

input CartItem {
    cartItemId: ID!
    sku: String!
    qty: Float!
    parentSku: String!
    options : [ProductOption!]
}

input ProductOption {
    optionId: ID
    valueId: ID
}
```
Add bundle product to cart
```graphql
type Mutation {
    addToCart(cartId: ID, cartItems: [CartItem]): Cart
}

input CartItem {
    cartItemId: ID!
    sku: String!
    qty: Float!
    parentSku: String!
    options : [ProductOption!]
}

input ProductOption {
    optionId: ID
    valueId: ID
}
```


Unified approach to add any product to cart
```graphql
type Mutation {
    addToCart(cartId: ID, cartItems: [CartItem]): Cart
}

input CartItem {
    cartItemId: ID!
    sku: String!
    qty: Float!
    parentSku: String
    options : [ProductOption!]
}

input ProductOption {
    optionId: ID
    valueId: ID
}
```
