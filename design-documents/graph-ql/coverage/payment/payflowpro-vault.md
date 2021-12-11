# Overview

In the scope of extending the GraphQL coverage, we need to add Vault support to PayPal Payflow Pro payment integration.

## Mutations

We need to extend existing `PayflowProInput` by adding a possibility to store a Vault token:

```graphql
input PayflowProInput @doc(description:"Required input for Payflow Pro and Payments Pro payment methods.") {
    cc_details: CreditCardDetailsInput! @doc(description: "Required input for credit card related information")
    is_active_payment_token_enabler: Boolean! @doc(description:"States whether an entered by a customer credit/debit card should be tokenized for later usage. Required only if Vault is enabled for PayPal Payflow Pro payment integration.")
}
```

The `is_active_payment_token_enabler` would specify that credit card details should be tokenized by a payment gateway, and a payment token can be used for further purchases.

The next needed modification is extend of the existing `PaymentMethodInput` by adding Vault payment method for PayPal Payflow Pro.

```graphql
input PaymentMethodInput {
    payflowpro_cc_vault: VaultTokenInput
}
```

The `VaultInput` provides a generic type for all payment integrations with the Vault support.

```graphql
input VaultTokenInput @doc(description:"Required input for payment methods with Vault support.") {
    public_hash: String! @doc(description: "The public hash of the payment token")
}
```