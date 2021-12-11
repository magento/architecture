# Allow class attribute for referenceBlock type in layout xml

## Problem

It is **impossible to substitute a block class** and/or customize block method(s) implementation **on a specific page keeping layout configuration of the block** (position, sorting in the layout, child blocks, arguments).

Currently available block class substitution approaches do not resolve the problem:

| Currently available approach                                                      | Why the approach does not resolve the problem  |
| --------------------------------------------------------------------------------- |:----------------------------------------------:|
| Introduce a preference for block class in di.xml                                  | Block is substituted globally, not only on the desired page |
| Remove the existing block from the layout and declare a new one                   | Layout configuration related to this block is lost |
| Use plugins                                                                       | Block functionality is affected for all instances, not only on the desired page |
| Use plugins that change method functionality only for specific page/layout handle | This approach is imperative, implicit, prone to mistakes, bugs and implementation inconsistency, it is not compatible with layout updates | 

## Solution

Allow the `class` attribute of `blockReferenceType` in `lib/internal/Magento/Framework/View/Layout/etc/elements.xsd`

See: https://github.com/magento/magento2/pull/18933

### Pros

* Possible to substitute a block class for a specific page easily and in declarative way preserving layout configuration of the block

### Cons

* Allows developer to substitute a block class with a class that is not compatible with a template

## POC

POC is implemented in public pull reqeust: https://github.com/magento/magento2/pull/18933
