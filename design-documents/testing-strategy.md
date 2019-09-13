## Updated Magento Commerce Testing Strategy

### Terms

<!-- Describe any new terms used in the document -->

### Overview

<!-- Describe what we are trying to solve and a few sentence summary of design approach. -->

Despite running 20,000 - 30,000 tests per pull request, we have little to no confidence in the stability of the product. This means that we still need a manual regression process that lasts weeks in the best of cases.

A sizable portion of this can be attributed to our current high-level testing mentality that prioritizes system-level black box testing over other types of tests. While these types of tests are necessary and good to have, they only provide implicit coverage to the code paths of the concrete implementations that happen to be loaded in the high-level black box tests. We still need explicit coverage for the concrete implementations that extension developers have hard dependencies on. This is true for the simple code paths as well as variations both positive and negative flows.

### Design

* Any class marked with an `@api` annotation MUST be covered with explicit test coverage. This coverage MAY be accomplished via  integration or unit tests (as possible). All behavior that can be invoked either directly or indirectly MUST have some form of test coverage that ensures that the concrete implementation (that is, the subject of the test) can not be inadvertently changed except in the test itself.
* You MUST write as much incremental test coverage as possible prioritized based on level:
  1. **Components MUST be tested on their own.** This MAY be done through unit or integration testing as appropriate. This level of test coverage ensures that the component can properly function on its own.  
  1. **Components MUST be shown to work with known collaborators.** This MUST be done through integration tests and ideally SHOULD involve as few components as possible per test case. This level of tests ensures that the component will work with its known associates. Examples of such associates would include aggregators, composites, or TODO.
  1. webapi behavior MUST be covered by blackbox tests to assert that the service contracts are properly satisfied. This ensures that the software package as a whole functions properly when everything is compiled together. This is in addition to other testing explained.
  1. MFTF tests SHOULD only be used to cover P0/P1 testing scenarios. Until these scenarios are all completed, a product owner MUST be consulted for approval of new tests. Additionally, MFTF MUST NOT be used for testing scenarios that are not P0/P1


#### Acceptance Criteria Fulfillment

<!-- If the document is intended for an existing story/task, provide alignment with its acceptance criteria. -->

* A new Definition of Done will be created and serve as a definitive test coverage rule for all of Magento Commerce.
* Training presentations will also be given to help ease this transition. 

#### Extension Points and Scenarios

Theoretically we may need to add additional functionality to our integration test framework to be able to cover certain types of extensibility corner cases. This is a problem that has often been discussed but nobody has yet to present an actual use case so the way in which we may need to make the framework more flexible isn't actually known at this time.

