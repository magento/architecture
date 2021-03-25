## Current caching
Like described in [dev docs](https://devdocs.magento.com/guides/v2.4/graphql/caching.html) we create a full query cache which we identify through a unique key

The unique cache key is usually a hash based on several factors which define the type of request, scope etc.

Currently that formula is:

````
hash([FULL URL + GET PARAMS] + [Store Header Value] + [Content-Currency Header Value])
````

All three types  of cache (FPC, Varnish and Fastly) now know about those headers and they compute different caches

## The problem
This strategy only supposes that all components of cache have public values, and knowing their values doesn't pose any security concern.
There is however what we can call private data that should not really be exposed or at the very least guessed easily if exposed.

Take customer-group as an example. At no point in Luma would we expose customer group. Instead we compute the same hash but including customer group and salt it:

````
hash([FULL URL + GET PARAMS] + [Store] + [Currency] + [CustomerGroup] + [Salt])
````

This works In Luma we put that value into a cookie called X-Magento-Vary which luma will send with each request.
In this case is the server who computes a cache key on which varnish or fastly can add.
Luma also stores in cookies the Store and Currency but never the value of customer group.

PWA and Graphql uses a cookieless approach. PWA has to know now about Store and Currency but not about the private components of cache.
Also each time we change something on private components like in this case with customer-group we have to add something to the VCL in fastly and varnish.

We also missed the fact that the server, since it has these components as variations of cache, should respond with:
````
Vary: Store, Content-currency
````

There is a bug in GraphQL, as it doesn't construct properly the Vary header:
````
Vary: Accept-Encoding, Cookie
````

All components/headers that compose cache key should be included in the Vary.

## The Solution
Graphql would compute the cache key and return a header with the same function as the Luma cookie (`X-Magento-Vary`), except we won't name it that way. The `Vary` header is not meant to have a hash.
````
X-Magento-Request-Id: 85a0b196524c60eaeb7c87d1aa4708a3fb20c6a1
````

PWA would send `X-Magento-Request-Id` back to GraphQL. So it will act like the cookie in Luma. 
This way we solve the public-private without adding more haders than we have. Just this one time.

This will require a change in VCL but only once. With all the other changes if we change the way cache key is computed, no VCL changes would be required nor in PWA.

VCL/Fastly would process this and turn it into a real cache key processed internally but the edge cache.
Currently, Fastly has this:

````
X-Request-Id: chn45aznc4feogj23ssw3y7k
````

The `X-Magento-Request-Id` will take into account `X-Magento-Request-Id` and respond with a propper Vary header:
````
Vary: X-Magento-Request-Id
````

This should suffice all explained. We don't need to add Store and Content-Currency to Vary anymore, though if we want to be transparent for the public headers we still can.
For FPC built in cache there would be no changes besides outputing X-Magento-Request-Id and the proper Vary. We already have the code that computes the cookie hash value.
