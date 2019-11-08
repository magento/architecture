## Separate mutations for updating customer email and password

### Problem overview
`createCustomer` and `updateCustomer` operations are currently sharing `CustomerInput` object. Among other fields `CustomerInput` contains `email` and `password`.

`password` has different semantics in these two mutations:
 - In scope of `createCustomer` it is a password of the newly created user
 - In scope of `updateCustomer` it serves for verification of the user's current password, when changing `email`

### Proposed solution

**updateCustomerV2**

Deprecate `updateCustomer` mutation in favor of `updateCustomerV2`. `CustomerUpdateInput` is similar to `CustomerInput` but excludes `password` and `email` fields.

 ```graphql
 mutation {
    updateCustomerV2(input: CustomerUpdateInput!): Customer
 }
 
 type CustomerUpdateInput {
     date_of_birth: String
     dob: String
     firstname: String
     gender: Int
     is_subscribed: Boolean
     lastname: String
     middlename: String
     prefix: String
     suffix: String
     taxvat: String
 }
 ```
 
 **updateCustomerEmail**
 
 ```graphql
  mutation {
     updateCustomerEmail(email: String!, password: String!): Customer
  }
 ```
 
 **updateCustomerPassword**
 
 ```graphql
  mutation {
     updateCustomerPassword(password: String!, old_password: String!): Customer
  }
 ```
 
 ### Alternative solution
 
 Alternative solution is to use existing field consistently for setting new customer password and introduce additional input argument for current password verification:

 - `password` field in `updateCustomer` must be used to set a new customer password
 - Introduce new optional argument for `updateCustomer` called `currentPassword: String`. This argument must be verified on the attempt of changing `email` or `password` fields
 - Existing web API tests must be adjusted and new flow for changing the password must be covered with tests
 
 The main issue with this approach is that API becomes more complex.
