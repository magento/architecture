# Integration Tests

## Problem Statement

Integration tests run in the same PHP process as Magento application.
A test from one module may depend on a data fixture or a model from another module.
In case of **single-area setup** or setup with **different application areas served by different instances**, integration tests and their dependencies may be deployed on **different instances** and, as a result, such tests won't be able to resolve their dependencies.

The question: should integration testing framework support distributed deployment or keep it compatible to monolithic deployment only?

Take into account future support of microservices architecture.

## Purpose of Integration Tests in Magento

Before deciding whether we should drop support of integration tests for distributed systems, let's revisit purpose of integration tests in Magento:
1. validate extension points of a component (potential integration with other components)
2. validate integration with other existing components. Integration tests don't use mocks (usually)
3. validate integration with the environment. Integration tests run on installed application, with real DB, etc


## Solution

From the purpose of integration tests described above, 
- 2 and 3 can and should be covered by functional tests as they validate real scenarios and so bring more value
- 1 still makes sense for integration tests, but only in scope of one instance (single-area application or service) because this is where in-process extension points are supported  

Based on this, it makes sense to run integration tests for a single-area application instance (or for a service in the future).

It should be taken into account that in case of multiple single-area application, DB is shared between all of them.

Changes to integration tests:
1. Break down tests by module (according to new module structure).
  1. Testing package declares which Magento package it depends on
  1. Testing packages are deployed on the same instances their dependencies are deployed. Deployment can be managed in the same way as Magento feature packages
1. Each instance has a dedicated DB (cache, other resources) for tests, so tests can run in isolation
1. Integration tests must not make network calls to other instances (though any network calls should not be necessary in integration tests)

Changes to integration testing framework:
1. Application area is configured per instance and does not change per test. `@magentoAppArea` annotation is eliminated as unnecessary.
1. `\Magento\TestFramework\TestCase\AbstractController` is eliminated. Consider converting tests relying on it (400+) to functional tests.

### Scope of Work

Proposed changes require revisiting all or almost all of existing integration tests.
It is a lot of work, but would help clean up and clarify purpose of integration tests.
By deprecating undesired framework functionality and using static tests validation, new tests can be implemented in a way that can be used with distributed deployment, while old tests will be considered compatible with monolithic deployment only.  
