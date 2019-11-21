## Evaluation of GraphQL subscriptions

Subscriptions in GraphQL allow clients to subscribe to the specific events on the server and receive real-time updates. Subscriptions are not implemented in the [grqaphql-php library](https://webonyx.github.io/graphql-php/type-system/schema/) used by Magento. Common implementation approach is to use [web sockets](https://en.wikipedia.org/wiki/WebSocket) as communication protocol. PHP does not natively support web sockets, however third-party libraries exist.


Here are some use cases that can be covered with subscriptions in Magento:
- Real-time stock and cart updates on storefront
- Real-time analytics, admin notification, order management, inventory updates in the admin panel


There are no plans to implement GraphQL coverage for admin panel scenarios. On the storefront the problem is a separate persistent connection will be required per user, which will increase requirements to the servers running Magento instances. Additionally, web-server will require custom configuration to support web-sockets.

## Conclusion

Subscriptions are not critical for any storefront scenarios. Potential improvement in user experience due to real-time updates for the customers does not justify the effort for implementation of custom GraphQL subscriptions framework, especially considering potential configuration and performance problems.

