# Proposal for Representing Storefront Routes in GraphQL

## Problem Statement

**URL routing in the PWA must handle all custom URLs maintained by the merchant, installed by extensions, and built in to Magento.**

PWAs have client-side routing logic so they can work offline.
The Magento PWA includes a [router implementation](https://reacttraining.com/react-router/web/guides/quick-start) which receives dispatched URLs and handles them.
Some of these routes are hardcoded into the PWA logic, but most routes are created and controlled by the business user.
The business user can modify the custom URL of any entity at any time.
Therefore, the PWA must query the Magento API to learn how to route a URL and display its associated entity.

This document proposes a GraphQL-driven solution to the routing issue that simplifies the routing scheme, supports business users, and enables extensibility.

## Requirements

As stated above, the PWA must be able to route:

1) SEO URLs for catalog and CMS pages
2) Built-in Magento routes, like `/checkout`
3) Custom routes installed by extensions and other modules

The current PWA implementation has a partial solution for **(1)**, using the [`urlResolver`](https://github.com/magento/graphql-ce/issues/80).
It has a suboptimal solution for **(2)**, in the form of reimplementing Magento routing logic in the client-side app router.
There is no current solution for **(3)**.

To support Magento's SEO and display customization features, the PWA must be able to query the API for the following data:

- **Entity type**, page type or content type (e.g. category, product, or CMS page)
- **Entity** content and data

With that information, the PWA can asynchronously load the view components which handle page display for the appropriate entity.

Additionally, the PWA should be able to query the API for **display metadata** for any entity.
This could be a simple string representing a shared "variant" component label, which would enable PWAs to support the equivalent of per-entity Theme Variants in existing Magento 2.

<!-- diagram: url => { entity_type, entity, metadata } -->

The current `urlResolver` query delivers part of these requirements: entity type in the form of an enum of "magic strings", such as `CATEGORY` and `CMS_PAGE`, and entity data in the form of a unique ID, which functions as a foreign key to be used in a subsequent GraphQL call.

<!-- diagram: url => { type, id } -->

This solution helps us honor SEO URLs, which is a _distinguishing feature_ of PWA Studio.
However, it also has flaws:

- The entity type enum must be hand-maintained and mapped to real types
- Not all routable entities have implemented types: only `CATEGORY`, `CMS_PAGE` and `PRODUCT`.
- The associated entity must be fetched in a subsequent API call
- No metadata exists for page display strategy
- Page types are not discoverable or extensible

## Proposal: GraphQL `RoutableInterface`

Add a `RoutableInterface` to the base schema, and require all entity types which are routable to implement it.

A subset of Magento entities are "routable", in that they have URLs and they serve as the model for a rendered page.
The natural fit for this concept is an interface, and GraphQL provides [multiple interfaces](https://graphql.github.io/graphql-spec/June2018/#sec-Objects) so that a given type can implement more than one.

As described in [Requirements](#requirements), the PWA must have the entity type and entity data for a given route, and it should be able to query for display metadata for that route. A GraphQL interface is a good match for these requirements:

- Get the entity type using `__typename` introspection to get the GraphQL concrete type of the `RoutableInterface`
- Get the entity data from the same `RoutableInterface` using GraphQL fragments on concrete type conditions
- Get optional display metadata by requiring that field in the `RoutableInterface`

### `RoutableInterface` Definition and Implementation

Since the entity type is built in to GraphQL introspection, `RoutableInterface` only needs to require the common field for display metadata.

```graphql
interface RoutableInterface {
    #copied from the deprecated EntityUrl type which will not be used anymore
    relative_url: String @doc(description: "The internal relative URL. If the specified  url is a redirect, the query returns the redirected URL, not the original.")
    redirectCode: Int @doc(description: "301 or 302 HTTP code for url permanent or temporary redirect or 0 for the 200 no redirect")
    display_metadata: String
}
```

Any type which can be a page model must implement `RoutableInterface` in addition to any other interfaces.

```graphql
type CmsPage implements RoutableInterface {
  content: String
  content_heading: String
  meta_description: String
  meta_keywords: String
  meta_title: String
  page_layout: String
  title: String
  url_key: String
  display_metadata: String
}

# multiple interfaces
type SimpleProduct implements ProductInterface & RoutableInterface {
  # [...other property definitions]
  display_metadata: String # not to be confused with meta_description, meta_keywords, meta_title which is defined in each entity like cms
}
```

### Route Query

To enable clients to query based on route, add a root query to the schema.

```graphql
type Query {
    urlResolver(url: String!): EntityUrl @deprecated(reason: "Use `route` instead")
    route(url: String!): RoutableInterface
}
```

This query would replace `urlResolver` in the PWA implementation; `urlResolver` could then be deprecated.

The simplest route query would be:

```graphql
route(url: "/product-foo.html") {
  __typename
}
```

The response to such a query would be:

```json
{
  "data": {
    "route": {
      "__typename": "SimpleProduct"
    }
  }
}
```

This query alone would not be useful without placing a subsequent query for entity data.
Since `RoutableInterface` entities are also page models, a more useful query would contain a typed fragment for supported components:

```graphql
route(url: "/product-foo.html") {
  __typename
  display_metadata
  ...on SimpleProduct {
    name
    description
    media_gallery_entries {
      url
    }
  }
  ...on CmsPage {
    title
    content
  }
}
```

The response would be:

```json
{
  "data": {
    "route": {
      "__typename": "SimpleProduct",
      "display_metadata": "featured",
      "name": "Product Foo",
      "description": "Example product",
      "media_gallery_entries": [
        {
          "url": "/media/catalog/product/foo.jpg"
        }
      ]
    }
  }
}
```

This combines entity type, entity data for two types of concrete entity, and display metadata in a single query.
The PWA uses `__typename` to route to the correct display component, `display_metadata` to optionally use an labeled alternate view (e.g. `featured`, `dark` or `microsite_A`), and the rest of the properties as content to render.

## Use Cases

The `RoutableInterface` covers all the use cases for PWA Studio to support core Magento functionality and familiar Magento extensibility.
With `RoutableInterface`, a Magento instance describes its sitemap in its GraphQL schema.
Extension developers create custom pages by adding an implementation of `RoutableInterface` to the store schema.

### SEO URL for catalog and CMS pages

The above examples demonstrate this idea. A full-featured PWA would ideally have typed fragments for each supported page type. When the page type is unfamiliar to the PWA, the PWA can still handle it by rendering a general purpose "app shell" and displaying a 404 or other custom page.

```graphql
route(url: "/product-foo.html") {
  __typename
  display_metadata
  ...on SimpleProduct {
  }
  ...on ConfigurableProduct {
  }
  ...on DownloadableProduct {
  }
  ...on CmsPage {
  }
  ...on Receipt {
  }
}
```

### Built-in Magento route

To keep the PWA sitemap in sync with the Magento instance configuration, _all valid routes must be represented as `RoutableInterface` implementations._

Even if there is no dynamic data in the requested page, the type must at least be implemented.

Example schema declaration:

```graphql
type CreateAccountRoute implements RoutableInterface {
  display_metadata: String
}
```

Example PWA query to resolve URL:

```graphql
{
  currentPage: route(url: "/create-account") {
    __typename
    display_metadata
  }
}
```

Example result:

```json
{
  "data": {
    "route": {
      "__typename": "CreateAccountRoute",
      "display_metadata": null
    }
  }
}
```

This allows the PWA to use Magento's routing rules.
A PWA implementor may choose to hardcode a URL like `/create-account` so that the above query is not necessary, which would improve performance.
However, the entity must be available in the route query so that a PWA without that route hardcoded will still honor Magento page types and custom URLs.

### Custom route installed by extension

Extension developers would implement custom pages in the same way that Magento modules implement built-in routes described above.
A full-featured extension would include:

- Magento extension code
- Bundled PWA component code, including a `RootComponent` which handles the custom type and/or a GraphQL fragment to be included in the routing query

### Validating and building the PWA against the sitemap

Using a standard introspection query, a PWA can determine the whole list of possible `RoutableInterface` concrete types at build time.

```graphql
{
  __type(name: "RoutableInterface") {
    possibleTypes {
      name
    }
  }
}
```

This query might respond with:

```json
{
  "data": {
    "__type": {
      "possibleTypes": [
        {
          "name": "SimpleProduct"
        },
        {
          "name": "ConfigurableProduct"
        },
        {
          "name": "Category"
        },
        {
          "name": "CmsPage"
        },
        {
          "name": "CreateAccountRoute"
        },
      ]
    }
  }
}
```

The build tooling can then validate that React components tagged with the `RootComponent` directive exist for all possible page types.