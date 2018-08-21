**Problem statement**

Current way of declaring queries and mutations limits extensibility and deprecation capabilities. 

```$graphqls
type Mutation {
    generateCustomerToken(
        email: String!,
        password: String!
     ): String!
}
```

Such declaration has several issues:
- It is not possible to extend or modify the list of arguments from 3rd party extension. Schema stitching mechanism performs merge on types level only
- There is no way to add extra data to the output of the mutation in a backward compatible way
- It is not possible to evolve arguments list and output type of our API in the future. Even if we decide to do breaking changes, there is no way to deprecate existing arguments and output data first


**Proposed solution: Wrappers for output and merger for arguments**

Wrappers for output type along with merging capabilities for arguments can solve extensibility and deprecation issues.

```$graphqls
type Mutation {
    generateCustomerToken(
            email: String!,
            password: String!
    ): GenerateCustomerTokenOutput!
}

type GenerateCustomerTokenOutput {
    token: String!
}
```

With such schema it is possible to extend the list of arguments (not reduce, however). For example, if system integrator got a new requirement to enable multi-factor authentication, the schema can be extended from 3rd party module to support this requirement as follows. All arguments from different modules will be merged and the resulting schema will contain all of them.

```$graphqls
type Mutation {
    generateCustomerToken(
            email: String!,
            password: String!,
            multi_factor_auth_token: String!
    ): GenerateCustomerTokenOutput!
}
```
Additionally, plugin can be added for the `generateCustomerToken` mutation resolver to implement additional verification step.

Now let's assume that client app needs to know when to request a new token. One of the solutions could be to return token TTL along with its value.
The output data can be extended in this case as follows:
```$graphqls
input GenerateCustomerTokenOutput {
    token_ttl: String!
}
```
The `token_ttl` value can be populated via new resolver for this field or from the plugin on existing `generateCustomerToken` mutation resolver.


**Action items**

1. Modify schema for all existing queries and mutations to use wrappers as output types
1. Make sure that this requirement is documented and followed for the new GraphQL coverage
