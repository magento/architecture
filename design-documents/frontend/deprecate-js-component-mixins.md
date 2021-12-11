# Deprecate JS Component Mixins

## Context

Magento 2 uses the term `mixins` to define 2 different concepts:

1. [RequireJS Mixins](https://devdocs.magento.com/guides/v2.2/javascript-dev-guide/javascript/js_mixins.html)
2. [JavaScript Component](https://devdocs.magento.com/guides/v2.2/javascript-dev-guide/javascript/js_init.html) mixins.

This document advocates for deprecation of #2.

## Problems

### Naming Confusion

Magento 2 developers are very familiar already with the concept of a mixin. It is well-known that the signature for a mixin function is:

```ts
define(function() {
    return function(target: Object): Object {
        // return alternative implementation for module
        // being overwritten
    }
});
```

However, the signature for a JS Component mixin is different:

```ts
define(function() {
    return function(config: Object, selector: string): Object {
        // return alternative config
    }
});
```

This difference will be a source of confusion for any Magento developer familiar with JS mixins.

### Performance

With JS Mixins, the feature was designed in such a way that we should be able to optimize performance over time (such as pre-fetching mixin modules). However, the implementation of JS Component mixins adds a not-insignificant delay to the time it takes for a JS Component to initialize. This is because the storefront code does not start fetching JS Component mixins until the declarative component code has been found in the DOM and evaluated.

## Usage

At the time of writing, JS Component mixins are only [used in 1 place](https://github.com/magento/magento2/blob/07ef8c12bec98846a93a1943448ea50e974fbeee/app/code/Magento/Catalog/view/frontend/templates/product/view/gallery.phtml#L43) in core. Because of the lack of documentation, it's unlikely (but not impossible) that 3rd party code uses this feature.

## Solution

Deprecate JS Component mixins, with a plan to remove them in a future version of m2. This will require a slight change to how the `magnifier/magnify` AMD module attaches itself to `Fotorama` on product pages.