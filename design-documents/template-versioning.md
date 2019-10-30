## Magento Frontend Template Versioning

### Terms

<!-- Describe any new terms used in the document -->

### Overview

Let's say ModuleA has template foo.phtml and CoolTheme overrides foo.phtml. Magento makes changes to foo.phtml and other files in ModuleA. Now CoolTheme has several problems:

- Hard to know if there were changes to templates it has replaced 
- Hard to know if how essential the changes are (security fix, new feature, typo, etc)
- Hard to know which files to even update 
- Hard to know which of the updates need to be applied to their overrides from the changes made to the base module template 

To make this problem even worse there have no rules surrounding backwards compatibility of templates in general. The DOM structure can change without notice and our automated tests will pass and all extensions will break with all of the above problems when trying to fix them without any rule violations.

These are very common scenarios and they have large security implications as well. Let's say there are some security fixes that spans across many templates. This could be the simple addition of a new hidden field, or something else like a new escapement policy. Whether it affects one or all of our template files, only a select few will actually ever be updated on merchant websites due to overrides and the difficulty of upgrading the templates.

#### Example

Product gallery template for 2.3.1 was large and contained all sorts of HTML and JS and JS config. https://github.com/magento/magento2/blob/2.3.1/app/code/Magento/Catalog/view/frontend/templates/product/view/gallery.phtml
Product gallery template for 2.3.2 was dramatically changed and hardly contains anything. https://github.com/magento/magento2/blob/2.3.2/app/code/Magento/Catalog/view/frontend/templates/product/view/gallery.phtml

If I have a template override based on 2.3.1 for gallery.phtml and upgrade Magento to 2.3.2 how do I know my override is compatible? How do I update the template? How would I even know this change was made to the template I'm overriding? Maybe everything looks fine in my template override but it was a security fix, how do I know if my override needs to be patched or if the changes were cosmetic? Essentially, how can I answer the basic question of "are my overridden templates up-to-date regardless of how customized they are?"

### Design

At a very high level this proposal aims to introduce several new things:

- A template versioning system with backwards compatibility rules for version changes
- Merchant visibility into theme compatibility

In an attempt to keep the scope of this design feasible, only the file fallback mechanism is being addressed at this time. Any other template overrides through manually calling block methods or other forms of overrides are out of scope.

#### Template versioning

A system of rules for html and phtml files using a new `@version x.x.x` annotation that allows templates to be updated in a way that allows developers to not only answer the problems listed above but also to handle the updating of their overrides in a much more controlled and expected way. The rules for versioning would also aim to prevent developers from creating backwards incompatible template changes between magento versions.

#### Merchant visibility

When a merchant installs a theme or upgrades their magento instances it would be helpful for them and for the theme developers to know that what they are using may not be compatible with the version of Magento installed. This will alert the merchant that their theme may not have the latest security fixes and that the theme needs to be updated. Not just for security fixes but other issues as well. They may be notified through some admin notification upon them install or magento upgrade and a new theme dashboard could be created to show which theme files are incompatible.

Maybe this can also include a simple tool that allows theme developers to easily view what changes have been made since the last version upgrade of the templates so they can patch their templates. Seeing the difference between their template and the Magento template isn't helpful in most cases since other unrelated customizations will have been made. This is a stretch goal and is already available in 3rd party tools. 
  
<!-- In this section provide relevant details at a high level, including the introduction of any new technologies being utilized for the design. 

Hints:
1. What breaking changes are expected? 
1. What information will be logged?
1. New data or config that should be propagated from dev to production?
1. Will this work on read-only filesystem? If no, provide details about what functionality requires writable filesystem.
1. Does this increase downtime?
1. What data or code migration is required? Describe possible ways of automatic migration, as well as highlight what can be done only manually.
1. Is there existing open source solution that can be used here? Can it be implemented using existing Magento feature?
1. Is any performance degradation expected, including application under high load?
1. Will it influence horizontal scalability of Magento? Does it introduce new tables? New foreign Keys? Can it be put to separate database? Is it failsafe?
1. Any new vulnerability type possible?
   1. New entry point introduced?
   1. Store or user data exposed?
   1. Is encryption needed?
   1. New ACL rule is needed?
1. New type of tests needed? New static tests?
1. Any staged content? How will it work with staged content?
1. Any new cacheable content? What pages will have to be invalidated if the content changes? Any new pages? More versions of existing content should be cached? Modifications to caching engine?
1. Is it isolated? Is it a routine work that does not require domain knowledge?
-->

#### Acceptance Criteria Fulfillment

TBD

#### Component Dependencies

<!-- List of components or epics that should be implemented to finish this epic --> 

#### Extension Points and Scenarios

TBD

<!-- In this section describe customization points that can be used by third party developers to customize behavior described in design -->

### Prototype or Proof of Concept

TBD

<!-- Is a proof of concept available for the design? If so provide a git gist or branch demonstrating the design. --> 

### Data size and Performance Requirements

TBD

<!-- If new behaviour is planned to be implemented, data and performance requirements must be described here. No significant resource consumption growth is allowed. --> 

