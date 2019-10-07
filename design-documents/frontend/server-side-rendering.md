# Server-Side Rendering (SSR)

SSR can be a challenge to implement correctly, but the benefits have the potential to provide the unparalleled performance you seek to decrease bounce rate and drive more business.

## Pros

- Faster, first meaningful paint.
- No need to render initial content via JavaScript on the client.
- SEO/bot support.
- Better SEO performance.
- JavaScript code can be shared universally (full-stack).

## Cons

- User may see some content before it's interactive.
- Difficult or tedious to implement.
- May require additional configuration and refactoring.
- Some features [not yet supported](#not-supported-yet).

## How it works

The basic idea is that we don't want the client browser to be responsible for rendering content via JavaScript, because JavaScript is notoriously slow, especially at rendering DOM and especially on older or mobile devices. Unnecessary rendering also consumes more energy, which has the potential to suck a mobile device's battery dry. :battery:

If we leverage the server to do as much heavy lifting as we can, some of this heavy lifting (non-sensitive pages) can be cached as well, resulting in a faster, better user experience.

### Without SSR

1. Client (browser) requests a URL for a particular route (e.g., `/`).
1. The server runs the appropriate code for that route.
   - HTML is partially generated with empty containers that need to be filled and rendered by JavaScript after initial page load (bad for SEO).
1. The client receives and processes the HTML.
   - The user might see some empty content that hasn't been filled in yet.
   - JavaScript is loaded and processed, filling specified empty containers with content (slow).
   - Once content is seen, it becomes immediately interactive.

### With SSR

1. Client (browser) requests a URL for a particular route (e.g., `/`).
1. The [Node.js][] server runs the appropriate code for that route.
   - HTML is fully generated via React libraries with no empty containers. This HTML can be streamed and cached.
1. The client receives and processes the HTML.
   - No additional HTML needs to be rendered into containers. As a result, the user perceives faster performance.
   - At this point, some content might appear to be interactive (e.g., buttons), but won't actually work yet.
   - JavaScript libraries (including React) are loaded along with any application code.
   - React hydrates the application.
   - The page is now interactive.

## Implementation

Some solutions provide out-of-the-box SSR, like [Next.js](https://nextjs.org/) and [Razzle][], but if you want more tight control of your application, you might want to set it up yourself. If you go this route, you would still do well to use these projects as inspiration. In fact, [Razzle][] has [a great development setup](https://github.com/jaredpalmer/razzle#how-razzle-works-the-secret-sauce).

To render a React application on a server, the server needs to understand how to process JavaScript. This means you'll need a ([Node.js][]) [Express][] server. Ideally, you'll serve the HTML by piping to a writable stream. You can do this with React 16's [`renderToNodeStream()`](https://reactjs.org/docs/react-dom-server.html#rendertonodestream).

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

## Optimizations

- Above the fold only SSR (e.g., [electrode's implementation](https://github.com/electrode-io/above-the-fold-only-server-render)).

## Not supported (yet)

- [Error Boundaries](https://reactjs.org/docs/error-boundaries.html)
- [Portals](https://reactjs.org/docs/portals.html)
- [React.lazy and Suspense](https://reactjs.org/docs/code-splitting.html#reactlazy)
- [Create React App](https://github.com/facebook/create-react-app)

## FAQ

### Why do I need Node.js (Express)?

Other server-side languages can render HTML, but are not designed to process JavaScript. Without a JavaScript engine, you won't be able to render the HTML generated by React components; thus, won't get SSR. [Express][] runs on [Node.js][], which runs Google's open source JavaScript engine, [V8](https://chromium.googlesource.com/v8/v8). This makes [Express][] the ideal server for rendering React applications. Technically, there are non-golden paths you can take to render JavaScript on some non-JavaScript servers, but it's not recommended.

## References

- [React | Update on SSR](https://reactjs.org/blog/2019/08/08/react-v16.9.0.html#an-update-on-server-rendering)
- [React | Suspense for SSR](https://reactjs.org/blog/2018/11/27/react-16-roadmap.html#suspense-for-server-rendering)
- [What’s New With SSR in React 16](https://hackernoon.com/whats-new-with-server-side-rendering-in-react-16-9b0d78585d67)
- [Making CRA apps work with SSR](https://hackernoon.com/making-cra-apps-work-with-ssr-b45f7c23d8db)

[express]: http://expressjs.com/
[node.js]: https://nodejs.org/
[razzle]: https://www.npmjs.com/package/razzle
