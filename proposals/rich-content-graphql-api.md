# Common GraphQL Representation of Rich Content

A content-authoring user of a Magento PWA store should be able to use PageBuilder to create and edit as much of the storefront content as possible, without drastically reducing PWA performance.
An integrator should be able to convert PageBuilder storage markup and Adobe Experience Manager UI languages, such as [Coral UI](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/reference-materials/coral-ui/coralui3/documentation.html) and [Granite UI](https://helpx.adobe.com/experience-manager/6-3/sites/developing/using/reference-materials/granite-ui/api/jcr_root/libs/granite/ui/index.html).

:warning: **Addendum 2019/1/9: Added description of [MVP workaround for upcoming PWA Studio releases](#pwa-studio-mvp-workaround).**

## Vision

PageBuilder builds its content in a tree of rows and columns of variable depth, whose leaf nodes are PageBuilder Blocks of various subtypes. Likewise, Coral UI and Granite UI, and most other rich content storage formats, use a tree structure of some kind, though it may not use rows, columns, and containers in the same manner. The GraphQL schema represents this with a recursively defined interface which all block types must implement. The interface consists of a short placeholder content string and enough type metadata to help the PWA query for the next layer of content. **The PWA loads PageBuilder content progressively**, by first requesting only the first two to three degrees of the content tree, then loading more content and functionality based on the metadata. As the PWA traverses PageBuilder content during a session, it caches type definitions and component implementations, and its content queries become more efficient over time. **The effect for the user is that the UI loads "from the outside in", with increasing speed as she navigates. The effect for the content author is that complex (deep) PageBuilder content will load in more "steps", incentivizing the content author to simplify layouts.**

## Proposal

Add an interface to the GraphQL schema representing a recursive tree of PageBuilder content nodes. Each PageBuilder block type should add a concrete type to the GraphQL schema which implements this interface and adds additional fields for any required configuration. PWAs initially query only against the interface, and can therefore only receive the metadata. In the first few content loads, the PWA must make a "followup" request with more concrete query depth. As its cache grows, it does fewer and fewer of those requests.

All GraphQL fields whose contents are meant to be rendered as HTML markup in the browser (for example, [the Product Description field defined here](https://github.com/magento/magento2ce/blob/2.3-develop/app/code/Magento/CatalogGraphQl/etc/schema.graphqls#L251)) should implement an interface type `VisualContentNode` (name TBD):

```graphql
interface VisualContentNode {
  role: String!
  depth: Int
  preload: [String]
  descendentRoles: [String]
  children: [VisualContentNode]
  html: String
}
```

(ID arguments would be necessary here but are omitted for simplicity).

PageBuilder content should consist of trees of these `VisualContent` nodes, with the `role` field containing a "type tag" string equal to the concrete type name of that block instance.

### Examples

#### Container (intermediate node)

Containers can have descendants. Rows, Columns, Tabs, and Accordions are examples of Containers.

A Column might declare these schema additions:

```graphql
enum ColumnAppearance {
  TOP_ALIGNED
  CENTERED
  BOTTOM_ALIGNED
  FULL_HEIGHT
}

type PageBuilder_Column implements VisualContentNode {
  role: String!
  depth: Int
  preload: [String]
  descendentRoles: [String]
  children: [VisualContentNode]
  html: String
  appearance: ColumnAppearance!
  backgroundColor: String
  backgroundImage: String
  backgroundPosition: String
  backgroundSize: String
  backgroundRepeat: String
  backgroundAttachment: String
  minHeight: Int
  verticalAlign: String
}
```

In this case, `ColumnAppearance` is a subtype declared for the use of `PageBuilder_Column`, but it has no meaning in the `VisualContentNode` render cycle otherwise.

A query against the `VisualContentNode` interface might look like:

```graphql
fragment NodeMetadata on VisualContentNode {
  role
  depth
  preload
  descendentRoles
}

content {
  ...NodeMetadata
  html
  children {
    ...NodeMetadata
  }
}
```

A particular instance of `PageBuilder_Column` in response to a query against `VisualContentNode` only might look like:

```json
{
  "content": {
    "role": "Column",
    "depth": 4,
    "preload": [
      "banner1.jpg",
      "diagram.svg",
      "thumbnail001.jpg",
      "thumbnail002.jpg",
      "video.mp4"
    ],
    "descendentRoles": ["Banner", "Accordion", "Video", "Image", "Row", "Text"],
    "children": [
      {
        "role": "Banner",
        "depth": 0,
        "preload": ["banner1.jpg"],
        "descendentRoles": []
      },
      {
        "role": "Accordion",
        "depth": 3,
        "preload": [
          "diagram.svg",
          "thumbnail001.jpg",
          "thumbnail002.jpg",
          "video.mp4"
        ],
        "descendentRoles": ["Video", "Image", "Row"]
      },
      {
        "role": "Text",
        "depth": 0,
        "preload": [],
        "descendentRoles": []
      }
    ],
    "html": "Hello world! Text has already loaded."
  }
}
```

The PWA would handle this response:

1.  In parallel:

    1.  Render the `html` content by injecting it into a `RawHTML` component.

    2.  Initiate preload requests for all the URLs in the top-level `preload` field. *Note that the responses "roll up" descendent preloads to a topmost list, as well as descendentRoles.*

    3.  In series:

        1.  In parallel, dynamically load and inject the React Components for:

            - `PageBuilder_Column`
            - `PageBuilder_Banner`
            - `PageBuilder_Accordion`
            - `PageBuilder_Video`
            - `PageBuilder_Image`
            - `PageBuilder_Row`
            - `PageBuilder_Text`

            (The bundle system and service worker will load from their caches if possible.)

        2.  Run a subsequent query provided by the components' respective implementations:

            ```graphql
            fragment PageBuilder_Column_Config on PageBuilder_Column {
              appearance
              backgroundColor
              backgroundImage
              backgroundPosition
              backgroundSize
              backgroundRepeat
              backgroundAttachment
              minHeight
              verticalAlign
            }

            fragment PageBuilder_Banner_Config on PageBuilder_Banner {
              appearance
              backgroundType
              "More banner config here."
            }

            """
            Additional fragments here.
            """

            content {
              ...PageBuilder_Column_Config
              children {
                ... on PageBuilder_Banner {
                  ...PageBuilder_Banner_Config
                }
                ... on PageBuilder_Accordion {
            	  ...PageBuilder_Accordion_Config
                }
                ... on PageBuilder_Text {
                  ...PageBuilder_Text_Config
                }
            	children {
                  ...NodeMetadata
                }
              }
            }
            ```

        3.  Re-render the fully hydrated second level of component when that query returns results.

Subsequent requests for any content which lists any of those components in its `descendentRoles` will automatically include those fragment and clauses in the `content` query. Therefore, subsequent requests will contain the necessary configurations on the first pass.

#### Leaf Node

A "leaf node" Slider widget might declare these schema additions:

```graphql
type SliderSlide {
  alt: String!
  src: String!
}

type PageBuilder_Slider implements VisualContentNode {
  role: String!
  depth: Int
  preload: [String]
  descendentRoles: [String]
  children: [VisualContentNode]
  html: String
  autoStart: Boolean
  slides: [SliderSlide]
}]]>
```

A particular instance of `Slider` in response to a query for its ID against the `VisualContentNode` interface would look like:

```json
{
  "content": {
    "role": "Slider",
    "preload": ["slide1.jpg", "slide2.jpg", "slide3.jpg"],
    "depth": 0,
    "descendentRoles": [],
    "children": [],
    "html": ""
  }
}
```

The PWA would handle this response:

1.  In parallel:

    1.  Render the `html` content by injecting it into a `RawHTML` component.

    2.  Initiate preload requests for all the URLs in the `preload` field.

    3.  In series:

    4.  Dynamically load and inject the `PageBuilder_Slider` React Component. (The bundle system and service worker will load from their caches if possible.)
    5.  Run a subsequent query provided by the component:

        ```graphql
        fragment PageBuilder_Slider_Config on PageBuilder_Slider {
          autoStart
          slides {
            node {
              alt
              src
            }
          }
        }

        content {
          ... on PageBuilder_Slider {
            ...PageBuilder_Slider_Config
          }
        }
        ```


        3.  Re-render the fully hydrated Slider component when that query returns results.

Subsequent requests for any content which lists `PageBuilder_Slider` in its `descendentRoles` will automatically include that fragment and clause in the `content` query. Therefore, subsequent requests will contain the necessary slider configuration on the first pass.

Fields which contain non-PageBuilder content (for example, TinyMCE WYSIWYG-generated or imported content) should be of a type `RawHtml` , instances of which have null `children` and `init` fields, a 0 depth field, and a `role` of `RawHTML` or some other signal value, and raw HTML in the `html` field, which the PWA runtime may sanitize and render. In this way, all content in GraphQL is of the same type.

## Challenges and FAQs

- **Why do more network traffic on the first pageload?**

  - Block implementations can provide placeholder content of arbitrary size and complexity; the only constraint is that this content must be "static", with no JS-driven behaviors. Blocks optimized for use on landing pages should have acceptable fallbacks implemented in the \`html\` field resolver.

* **Why not use GraphQL introspection for this?**

  - Per the GraphQL specification, the introspection query must provide only metadata, and not any data from resolvers. Implementing "partial introspection" via type tag names is a more efficient way of resolving recursive content nodes.

- **Why increase the number and size of queries, and add complexity to PageBuilder implementations, by forcing them to lift their configuration into the GraphQL type system?**

  - The intent of GraphQL is to provide customizable depth and detail in queries and responses.
  - The intent of PWA is to "progressively load" content and functionality.
  - The intent of PageBuilder is to provide a way to make content of arbitrary complexity and appearance.
  - Best practices for PWAs indicate that most pages should be as simple as possible, with little "depth" of implementation.
  - The progressive recursion method is the best compromise between these needs. The initial query provides immediate TTFMP, and the subsequent queries fulfill TTI. PageBuilder authors are naturally incentivized to keep their implementations simple, but they are not forbidden from making things complicated, especially on interior pages. They simply accept that complex content will load in layers.

## Recap

A unified transmission format is needed for rich content created by business users to flow into PWAs.

PageBuilder storefront code in 2.3 is built with Magento 2 JS components and their RequireJS-jQuery-KnockoutJS architecture. When displaying PageBuilder content on the storefront, Magento renders all server-side dynamically linked content into HTML, and outputs plain HTML, CSS, and JavaScript to the page.

Adobe Experience Manager Coral-UI and Granite-UI components are implemented as Web Components and Angular directives.

PWA Studio creates an alternative storefront rendering system using ReactJS, which cannot easily integrate these components. ReactJS expects content in the form of React Components and server data as "props". It cannot easily consume and display raw HTML and JavaScript, while maintaining its PWA performance and other features as a rendering system.

Therefore, for a store to support both the premier content authoring of PageBuilder and the high-performance storefront of PWA, we must provide an adapter layer that integrates PageBuilder content into the PWA render pipeline. This proposal does so by organizing PageBuilder content into a tree of nodes, which implement a known interface and then expand upon that node interface for different node types. Each query provides static HTML fallback content as metadata using the existing templates (or new HTML templates optimized for PWA) and then enough metadata for the PWA to load live React components, to upgrade progressively to dynamic PageBuilder functionality.

## PWA Studio MVP workaround

As of the release of Magento 2.3, this proposal has not been approved, and @melnikovi and I have agreed on a temporary workaround that we plan to use for the first iteration of PageBuilder support. This proposal should not be withdrawn, though; our workaround is for PageBuilder alone, whereas the Rich Content GraphQL API is meant to apply to content from *any CMS*, such as Adobe Experience Manager, and help us unify syndicated content into a single graph. I still think it's a good idea!

### Workaround summary

Currently, the GraphQL API for CMS blocks and other rich content fields simply returns the HTML contents of the entity as a string. The PageBuilder storage format is HTML with data attributes indicating structure and widget functionality. Using the native DOM parser in the web browser, the PWA can read this data directly and transform it into a data structure similar to the VisualContentNode described above.

### Workaround Process

1. Browser renders a `<RichContent id={id} contents={contents} />` element inside a component like `CmsBlock` or `ProductFullDetail`.
2. After mount, `RichContent` component parses `props.contents` as a [DocumentFragment](https://www.google.com/search?q=documentfragment&oq=documentfragment&aqs=chrome.0.0l6.1759j0j7&sourceid=chrome&ie=UTF-8).
3. Before initial render, `RichContent` component strips the fragment of dangerous HTML.
4. For initial render, `RichContent` injects the sanitized fragment.
5. After initial render, `RichContent` visits the DOM in the fragment to build a tree of PageBuilder rows, columns, widgets, and configurations.
6. When the content tree is finished, `RichContent` dynamically imports the React components corresponding to all the PageBuilder widget types present in the content.
7. When the runtime has loaded the components, `RichContent` re-renders the content tree as instances of live React components instead of HTML.

### Considerations

- The workaround involves DOM parsing and large string comparisons for update filters, which are both expensive operations which PWAs try to avoid.
- If possible, expensive operations should take place off the main thread, so this implementation may want to parse and visit the content in a Web Worker or ServiceWorker.
- React Components for each widget type are still necessary.
