## Use cases

### Admin user obtains customer token

Admin user is expected to be authenticated using admin token. While there is no GraphQL Mutation for retrieving the admin token, REST should be used (see [example](https://devdocs.magento.com/guides/v2.4/graphql/queries/index.html#staging)).

Admin bearer token must be provided in the `Authorization` header along with the following mutation. The admin user associated with that token must have `Login as Customer` (`Magento_LoginAsCustomer::login`) permissions. As usual, the store context is determined based on `Store` header in the request.

```graphql
mutation {
  generateCustomerTokenAsAdmin(
    input: {
      customer_email: "customer@example.com"
    }
  ) {
    customer_token
  }
}
```

### Customer can manage and view "Allow remote shopping assistance" flag

This flag should be added to `Customer`, `CustomerCreateInput` and `CustomerUpdateInput`.

## Additional requirements
 1.  A new `LoginAsCustomerGraphQL` module should be created. 
 2. `Magento_LoginAsCustomer::login` permission must be declared in `LoginAsCustomer` module.
 3. The following store config settings must be honored, but should not be exposed as part of `StoreConfig` GraphQL query:
```xml
<login_as_customer>
    <general>
        <enabled>0</enabled>
    </general>
</login_as_customer>
```
 4. Customer field `allow_remote_shopping_assistance` must be taken into account
