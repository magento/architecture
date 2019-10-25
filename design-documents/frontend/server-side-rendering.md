# Server-Side Rendering (SSR)

_This document refers to SSR in the context of a React web application (e.g., PWA). The concept of SSR does not apply to native apps._

_SSR belongs to the BFF layer of [service isolation](https://github.com/magento/architecture/tree/master/design-documents/service-isolation)._

SSR can be a challenge to implement correctly, but the benefits have the potential to provide the unparalleled performance you seek to decrease bounce rate and drive more business.

## Pros

- Faster, first meaningful paint.
- No need to render initial content via JavaScript on the client.
- SEO/bot support.
- JavaScript code can be shared universally (full-stack).

## Cons

- User may see some content before it's interactive.
- Difficult or tedious to implement.
- May require additional configuration and refactoring.
- Some features [not yet supported](#not-supported-yet).

## How it works

The basic idea is that we don't want the web browser (client) to be responsible for rendering initial page content via JavaScript, because JavaScript is notoriously slow, especially at rendering DOM and especially on older or mobile devices. Unnecessary rendering also consumes more energy, which has the potential to suck a mobile device's battery dry. :battery:

If we leverage the HTTP server to do as much heavy lifting as we can, some of this heavy lifting (non-sensitive pages) can be cached as well, resulting in a faster, better user experience.

### Without SSR

1. A web browser (client) requests a URL for a particular route (e.g., `/`).
1. The HTTP server runs the appropriate code for that route.
   - HTML is partially generated with empty containers that need to be filled and rendered by JavaScript after the initial page load (bad for SEO).
1. The web browser (client) receives and processes the HTML, but the user may see some empty content that hasn't been filled in yet.
1. JavaScript is loaded and processed, including the React library.
1. Client-side JavaScript makes any number of requests to various APIs for the initial data needed to render the page; thus, filling in the empty containers with content.
1. React renders the final HTML into the above-mentioned empty containers by combining the data received with the HTML defined in the React components.
1. Once content is seen, it becomes immediately interactive.

### With SSR

1. A web browser (client) requests a URL for a particular route (e.g., `/`).
1. The [JavaScript-capable HTTP server](#requirements) runs the appropriate code for that route.
   - Ideally, a single GraphQL API request is made for from this HTTP server, collecting all the data that's required to render [critical content](https://www.abtasty.com/blog/above-the-fold/) on page.
   - HTML is fully generated [via React methods](https://reactjs.org/docs/react-dom-server.html) with no empty containers. This HTML can be [streamed](https://reactjs.org/docs/react-dom-server.html#rendertonodestream) and cached.
1. The web browser (client) receives and processes the HTML.
   - No additional HTML needs to be rendered into containers. As a result, the user perceives faster performance.
   - At this point, some content might appear to be interactive (e.g., buttons), but won't actually work yet.
1. JavaScript is loaded and processed, including the React library.
1. React [hydrates](https://reactjs.org/docs/react-dom.html#hydrate) any containers whose HTML contents were rendered by [ReactDOMServer](https://reactjs.org/docs/react-dom-server.html) in step 2 (above). Nothing needs to be re-rendered here.
1. The page is now fully interactive.
1. Subsequent API requests are made directly to the GraphQL API.

## Requirements

- A server with access to _something_ that can execute JavaScript, typically a [Node.js][] server (e.g., [Express][]).

## Implementation

Some solutions provide out-of-the-box SSR, like [Next.js][] (comprehensive), [Razzle][] (lacks [data fetching](#data-fetching)) or the light-weight [Preact](https://preactjs.com/guide/v10/server-side-rendering). You don't have to use any of these, but they might serve as inspiration for your custom implementation.

### Data fetching

A comprehensive SSR strategy requires data fetching on the server.

- If you go the [Next.js][] route, read [GraphQL with Next.js and Apollo](https://nec.is/writing/graphql-with-next-js-and-apollo/).
- If not, you might want to make a choice between [Apollo Client](https://www.apollographql.com/docs/react/) and Facebook's [Relay](https://graphql-code-generator.com).
- [GraphQL Code Generator](https://graphql-code-generator.com) can generate code from your GraphQL schema with a single function call and [integrates with Apollo](https://graphql-code-generator.com/docs/plugins/typescript-react-apollo) as well as [with Relay](https://graphql-code-generator.com/docs/plugins/relay-operation-optimizer).

Whatever you decide, the most important thing is that [critical content](https://www.abtasty.com/blog/above-the-fold/) is rendered with all of its data on the server before it's served to the client. Note that this might require a 2-pass rendering strategy.

### Rendering

To render a React application on a server, the server needs access to [_something_ that processes JavaScript](#requirements). Ideally, you'll serve the HTML by piping to a writable stream. You can do this with React 16's [`renderToNodeStream()`](https://reactjs.org/docs/react-dom-server.html#rendertonodestream).

```jsx
import { renderToNodeStream } from 'react-dom/server'
import MyPage from './MyPage'

app.get('/', (req, res) => {
  res.write('<!DOCTYPE html><html><head><title>My Page</title></head><body>')
  res.write('<div id="content">')
  const stream = renderToNodeStream(<MyPage/>)
  stream.pipe(res, { end: false })
  stream.on('end', () => {
    res.write('</div></body></html>')
    res.end()
  })
})
```

_Note that this has to be done for every route, so depending on your routing strategy, you will have to tweak it to support SSR._

Next, you need to tell the client how to hydrate the app.

```jsx
import { hydrate } from 'react-dom'
import MyPage from './MyPage'

hydrate(<MyPage/>, document.getElementById('content'));
```

That's it &ndash; you technically now have SSR! But don't stop here. It's time to look for [optimization opportunities](#optimizations), of which there may be many.

Furthermore, some libraries that you are using might have specific SSR requirements (e.g., [Loadable Components](https://www.smooth-code.com/open-source/loadable-components/docs/server-side-rendering/)). Please consult their documentation to ensure you are adhering to those requirements.

## Extensibility

Market research is required to assess the extent at which extensibility is desired and where the interception points should be.

We have an opportunity here to take a fresh look at the extensibility model of Magento, where it has served us well and where it can be improved.

## Optimizations

- Above the fold only SSR (e.g., [electrode's implementation](https://github.com/electrode-io/above-the-fold-only-server-render)).

## Not supported (yet)

- [Error Boundaries](https://reactjs.org/docs/error-boundaries.html)
- [Portals](https://reactjs.org/docs/portals.html)
- [React.lazy and Suspense](https://reactjs.org/docs/code-splitting.html#reactlazy)
- [Create React App](https://github.com/facebook/create-react-app)

## FAQ

### Why do I need a JavaScript engine?

React components are written or transpiled into JavaScript. In order to execute JavaScript code, you need a JavaScript engine. Without one, you won't be able to render the HTML generated by React components; thus, won't get SSR. [Express][] runs on [Node.js][], which runs Google's open source JavaScript engine, [V8](https://chromium.googlesource.com/v8/v8). This makes [Express][] a typical choice for rendering React applications. Technically, there are non-golden paths you can take to render JavaScript on some non-JavaScript servers, but it's not recommended.

## References

- [React | Update on SSR](https://reactjs.org/blog/2019/08/08/react-v16.9.0.html#an-update-on-server-rendering)
- [React | Suspense for SSR](https://reactjs.org/blog/2018/11/27/react-16-roadmap.html#suspense-for-server-rendering)
- [What’s New With SSR in React 16](https://hackernoon.com/whats-new-with-server-side-rendering-in-react-16-9b0d78585d67)
- [Making CRA apps work with SSR](https://hackernoon.com/making-cra-apps-work-with-ssr-b45f7c23d8db)
- [Best PWA Framework, UI kit and starter? React Preferred](https://www.reddit.com/r/PWA/comments/dcsdys/best_pwa_framework_ui_kit_and_starter_react/f378bko/)

[express]: http://expressjs.com/
[next.js]: https://nextjs.org/
[node.js]: https://nodejs.org/
[razzle]: https://www.npmjs.com/package/razzle
