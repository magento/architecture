## Current caching
For GraphQL, we create a full query cache, which has results stored under unique keys as described in the [dev docs](https://devdocs.magento.com/guides/v2.4/graphql/caching.html).

These unique keys are a combination of several factors which define the scope of a request. Currently, this is the formula used for GraphQL:

````
[Store Header Value] + [Content-Currency Header Value]
````

All three types of cache (FPC, Varnish, and Fastly) know about these headers and use them to compute their cache keys.

## The problem
This strategy only allows for all the components of the cache key to have public values, and for a user knowing those values to not pose any security concerns.
There is, however, what we will call "private data" which should not be exposed, or at the very least it should not be able to be reproduced/guessed easily if it is. 

Take Customer Group as an example: At no point in Luma do we expose Customer Group. Instead, we compute a hash using Store and Currency but also including Customer Group and a salt:

````
hash([Store] + [Currency] + [CustomerGroup] + [Salt])
````

Using a salted hash hides the Customer Group from the user but still allows it to be considered as a cache key.
This works in Luma because we put that value into a cookie called `X-Magento-Vary`, which Luma sends with each request for the VCL code to use to do cache lookups.
In this case it is the server that computes the cache key for Varnish/Fastly.
Luma also stores Store and Currency in cookies, but never the value of Customer Group.

PWA and GraphQL, however, use a cookieless approach.
PWA currently knows about Store and Currency, but not about the private components of the cache key. Because of this, we explicitly bypass the cache for logged-in customers in order to hit Magento and retrieve correct results.
Also, each time we change what is used for the cache key (such as adding Customer Group), we have to change the VCL in Fastly and Varnish.

Additionally, we missed the fact that the cache node, since it uses these values as components of its own cache key, should respond with this `Vary` header for the browser cache:
````
Vary: Store, Content-Currency
````

The Fastly VCL code for GraphQL currently has a bug where it returns the same `Vary` header as it does for non-GraphQL requests (which use a cookie to store `X-Magento-Vary`):
````
Vary: Accept-Encoding, Cookie
````

All components/headers that compose the cache key should be included in the `Vary` header read by the browser cache.

## The Solution
On every request, our GraphQL framework will compute a salted and hashed cache key using the same factors as the Luma `X-Magento-Vary` cookie and return it in a header on the response.
````
X-Magento-Cache-Id: 85a0b196524c60eaeb7c87d1aa4708a3fb20c6a1
````

PWA will capture this header and send it back on following GraphQL requests, which the cache node will then use as the key.
This requires a VCL change, but only once; the VCL code will not care about the method used to generate the header, so even if that changes no further updates would be required.

PWA will continue to capture the header on every response, and when it makes a new request it will send the most recent value it has received.
This way, if something happens that changes a customer's cache key such as updating their shipping address, PWA will immediately pick it up and use the correct key on the next request.

A different value for `X-Magento-Cache-Id` can be sent by PWA than Magento calculates for the response, such as when a user changes their currency (which does not use a mutation).
When this happens, we cannot cache the response under the `X-Magento-Cache-Id` that was on the request, or else incorrect results will be returned for other requests also using that initial value.
To avoid this issue, the VCL code will compare the `X-Magento-Cache-Id` values on the request and the response and not store the result in the cache if they do not match.

The cache node will also respond with a proper `Vary` header for the browser cache to use:
````
Vary: Store, Content-Currency, Authorization, X-Magento-Cache-Id
````
There is no mutation for changing Store or Currency, so they need to remain in `Vary` for the browser cache to know to use them as keys.
Likewise, a customer logging out does not use a mutation; instead, PWA just stops sending the `Authorization` header. The browser cache needs to know that this could cause a change in result values, so `Authorization` also needs to be in `Vary`. Of note: The VCL code will not consider the full bearer token when doing a cache lookup, just whether it exists on the request at all.

Like `X-Magento-Vary`, the hash calculation for the `X-Magento-Cache-Id` header will use an unpredictably random salt. To ensure this salt is consistent between requests, we will store it in the environment configuration.
However, as this will happen as part of the first GraphQL request without a pre-existing salt, multiple attempts to update the config could occur simultaneously before the first finishes.
To avoid issues with concurrent writes to the same file, we will use a lock when writing to `app/etc/env.php` while adding the salt.
Generating this value automatically instead of through an admin interaction will avoid adding a step to the upgrade process and allow us to ensure the salt is sufficiently random.

For built-in FPC, there will be no changes other than outputting `X-Magento-Cache-Id` and the proper `Vary`.

This should account for all issues listed above except for a major possible security flaw explained below.

### The Solution's major flaw
We're caching the whole query, requests hit the caching server first, and we don't (yet) have an auth session service, nor is there a differentiator between auth token and session token. This means we can't currently validate the bearer token before the cache node checks for a hit.

This is not a problem in Luma because it has blocks, and blocks marked as private aren't cached. We use separate ajax to populate those after we retrieve the page from the cache.
Unfortunately, GraphQL doesn't have any equivalent to Luma's blocks. We could say that a node has a set of blocks (Example: `product {block { subblock }}`), but our current use cases are catalog and prices, which we could populate and not cache.
For category permissions and shared catalogs, suddenly the whole response stops being cacheable because the products and categories might be different.

We couldn't find a viable solution for this without having some high availability service that checks session token, which will only be possible after having full oauth2 protocol in 2.5.
