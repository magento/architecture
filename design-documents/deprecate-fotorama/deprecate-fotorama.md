# Overview

On Magento 2 product pages, product images are displayed using a forked version of the [`Fotorama`](https://github.com/artpolikarpov/fotorama) library. Some unfortunate design decisions in this library, coupled with the amount of JS execution on product pages, makes the rendering of product photos a very obvious bottleneck for shoppers.

Speaking with Magento SIs and Partners at various conferences, it sounds like a good number of agencies already rip out `Fotorama` as one of their default tasks when building out a new store for a client. This should be a key signal for us that this area needs improvement, and we should lean on them for their suggestions and overall expertise when going through this implementation.

In this document, we'll propose deprecating `Fotorama`, and replacing it with an alternative that has better performance characteristics.

## Discussion/Complaints about Fotorama Performance

- https://github.com/magento/magento2/issues/14667
- https://github.com/magento/magento2/issues/14667#issuecomment-415813263 (Comment from issue linked above diving into specific bottlenecks)
- https://github.com/magento/magento2/issues/6018

## Alternatives Proposed by Community
[Rendered](proposed-alternatives.md)

## Performance Problems with Fotorama

### Library Size

The [minified version of `Fotorama`](https://github.com/magento/magento2ce/blob/8a7f23e95/lib/web/fotorama/fotorama.min.js) in the 2.3 branch is `71.9 KB`. This is _massive_ for a photo gallery script. There are two problems related to the size of the library:

- With gzipping turned on, the library is still roughly `21 KB` that has to be sent over the network. This isn't huge for those on desktops with reliable connections, but could take 1 second or more on a slower 3g connection.
- [JavaScript parse time on slower smart phones is a massive bottleneck](https://medium.com/@addyosmani/the-cost-of-javascript-in-2018-7d8950fbb5d4). Because parsing happens against the uncompressed (but minified) JavaScript, the parse time is based on the `71.9 KB` figure. Using [`WebPageTest`](http://webpagetest.com/easy) to test on a [Moto 4G](https://www.gsmarena.com/motorola_moto_g4-8103.php), the parse time in [v8](https://en.wikipedia.org/wiki/Chrome_V8) took `398.6 ms`.

### Execution Time

Fotorama takes a significant amount of time to execute once the script has been loaded. In my most recent trace on [`WebPageTest`](http://webpagetest.com/easy) with a [Moto 4G](https://www.gsmarena.com/motorola_moto_g4-8103.php), it took around `336 ms` to execute. In the wild on real Magento stores, I've seen this take as long as `650 ms`, which is all time spent blocking the main thread.

### Resource Discoverability

Modern web browsers all have their own version of a [preload scanner](https://andydavies.me/blog/2013/10/22/how-the-browser-pre-loader-makes-pages-load-faster/). While the browser is spending time on script execution or other blocking tasks, a parser works in the background to discover all external resources, and then starts fetching them to get a head start.

Unfortunately, Magento 2 uses a feature of `Fotorama` that works by being fed an array of images during instantiation of the widget on the client-side. [Magento renders this data from the server in an `x-magento-init` tag](https://github.com/magento/magento2ce/blob/8a7f23e9/app/code/Magento/Catalog/view/frontend/templates/product/view/gallery.phtml#L47), which the browser sees as nothing more than a string. Because no `<img />` tag is rendered from the server, the browser is unable to start preloading images early. The browser only begins fetching the main product photo after `Fotorama` has appended the first `<img />` tag to the DOM.

Injecting the `<img />` tag(s) with JavaScript also has the unfortunate side-effect that, even if the images are already cached in a user's browser (repeat visit), they will still see the loading spinner for several seconds.

### Late Execution

The `Fotorama` library is initalized using an [`x-magento-init`](https://devdocs.magento.com/guides/v2.3/javascript-dev-guide/javascript/js_init.html) configuration. This poses a problem, because Magento "JS Components" do not have a way of expressing priority, and their execution order is non-deterministic. In practice, their execution order typically comes down to what external scripts finish loading first (the core loops over widgets on pageload and loads their JS assets in parallel). Because of the size of `Fotorama`, the gallery is often one of the last scripts to execute.

## Proposed Solution

The proposed solution here is to deprecate usage of `Fotorama` in Magento 2, and replace its usage with a library that does not have the same pitfalls as `Fotorama`.

### Requirements for a replacement

- Library _must_ allow easily server rendering the first `<img />` tag, to ensure the resource is picked up by the preload scanner
- Library should _not_ be more than `5 KB` minified and gzipped
- Library _must_ support both desktop and mobile interactions
- Library must _not_ be written in-house. Sliders have been around for a _long_ time, and there is no need to reinvent the wheel
- Should _not_ be initialized using the `data-mage-init` or `x-magento-init` functionality in core

### Non-Goals

- It is not a goal to pick a library that has every feature a store owner would want. To keep the core slim and fast, the goal is to choose the simplest library that covers all _basic_ use-cases for shoppers. For those that want all the bells and whistles, there are [a signficant number of options in the Magento Marketplace](https://marketplace.magento.com/catalogsearch/result/?q=slider).

## Outstanding Work

- [ ] Decide on a new library to replace Fotorama
- [ ] Decide/document what to do with product videos (also uses `Fotorama` currently)
- [ ] Verify `Fotorama` is not used in core anywhere but Product pages
- [ ] Decide/document the deprecation path