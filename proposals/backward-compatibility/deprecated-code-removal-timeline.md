# Timeline for Removal of Deprecated Code

## Overview

Concern raised by the community that it's not clear how long deprecated code will be preserved based on the rules documented at [Removal of deprecated code](http://devdocs.magento.com/guides/v2.2/contributor-guide/backward-compatible-development/#removal-of-deprecated-code).

## Requirements and Acceptance Criteria

1. Clear message about the timeline of deprecated code removal, which allows the community to plan their activities on refactoring the extensions
1. Strict enough rules, so that the extension developers have enough time for upgrading their code
1. Loose enough rules, so that Magento core team could avoid unreasonable maintenance costs caused by deprecated code

## Theory of Operation

### Option 1: Component-based rules (current)

Preserve current rules. Deprecated code is preserved:
* `@api` code: Until the next major version of the **component**
* non-`@api` code: The next 2 minor releases or until a major release of the **component**

Additionally, add an explanation of relation between product version and component version:

* patch version of the product may contain only patch and/or minor version upgrades of components
* minor version of the product may contain major, minor or patch version upgrades of components

### Option 2: Product-based rules

Deprecated code is preserved:
* `@api` code: Until the next minor version of the **product**
* non-`@api` code: Until the next patch version of the **product**

### Option 3: Term-based rules

* `@api`-code: preserved for the next X months
* non-`@api` code: no guarantees or short term

The term to be discussed separately if this approach will be chosen.
