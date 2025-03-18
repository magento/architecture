
**Magento 2 Architecture Goals**


1. *Streamline customizations*:  Reduce implementing time for frontend and backend features. 
    - Loose coupling between modules. 
    - Configuration over customization (Dependency Injection,Object Manager).
    - Design Patterns
- *Simplify external integrations:* Reduce complexity for integrations with 3rd party services 
(e.g. Payment Service Providers, Shipping Providers, Mobile Apps, ERP  etc.).
  - Service layer 

## Design Document Template

Copy this document to start a new Design Document.

### Terms

<!-- Describe any new terms used in the document -->

### Overview

<!-- Describe what we are trying to solve and a few sentence summary of design approach. -->

### Design

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

<!-- If the document is intended for an existing story/task, provide alignment with its acceptance criteria. -->

1. Acceptance Criteria 1
  <!-- Description how acceptance criteria will be achieved --> 

#### Component Dependencies

<!-- List of components or epics that should be implemented to finish this epic --> 

#### Extension Points and Scenarios

<!-- In this section describe customization points that can be used by third party developers to customize behavior described in design -->

### Prototype or Proof of Concept

<!-- Is a proof of concept available for the design? If so provide a git gist or branch demonstrating the design. --> 

### Data size and Performance Requirements

<!-- If new behaviour is planned to be implemented, data and performance requirements must be described here. No significant resource consumption growth is allowed. --> 


