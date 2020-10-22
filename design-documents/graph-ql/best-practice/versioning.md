## Problem 

GraphQL APIs are constantly evolving and require changes to be made to existing queries, mutations and data types.

Some GraphQL schema changes can be covered using versionless strategy, which [is considered best practice](https://graphql.org/learn/best-practices/#versioning) for GraphQL APIs. Examples of such changes include:
 - new fields in return types
 - deprecation and subsequent removal of fields in return types
 - new optional arguments of the fields, queries and mutations
 - replacement of existing query or mutation with a new semantically different one. Example is replacing a set of mutations for adding products to cart with a single unified mutation which can add products of any type to cart. Semantics of the operations is different in this case, that is why it is not difficult to come up with a new meaningful name. The old mutations and queries should be deprecated and eventually removed.
 
Sometimes a new query which is meant to replace existing one has the same semantics and thus it is difficult to give the new query meaningful name. Example is new `products` query, which has a set of filters incompatible with existing implementations, thus it is impossible to comply with backward compatibility and deprecation policies.

## Proposed solution

It is proposed to suffix names of the new fields/mutations/queries which are semantically close to existing ones with  `v2`, `v3`, ..., `vX`. In the example above, a new query will be named `productsV2`, while existing `products` should be deprecated. Note that version suffixes should start from `v2`, while suffixless fields will be considered `v1` by default.

The main benefit of the proposed solution is simplicity for server and client side developers. Nothing has to be implemented on the framework level to support this strategy and maintenance cost will be close to zero (we still need to remove deprecated fields when the time comes).


## Alternatives considered

 1. Versioning of the GraphQL endpoint itself: `http://magento.host/graphql/v2`
    1. Uncommon and considered [bad practice](https://graphql.org/learn/best-practices/#versioning)
    1. Another issue is that we will have to support multiple versions of the schema and third parties will have to extend all of them
    1. Custom implementation will be required to avoid schema duplication and versioning
 1. Versioning of all fields based on the version of the module they belong to
    1. Clients will have to be modified with every upgrade of the Magento (as it will most likely lead to bump in the API version)
    1. Such strategy will require complicated infrastructure for bumping API versions, similar to the one used for bumping module versions in composer 
