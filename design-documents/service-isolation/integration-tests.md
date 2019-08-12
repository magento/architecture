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
1. validate integration with other existing components. Integration tests don't use mocks (usually)
1. validate integration with the environment. Integration tests run on installed application, with real DB, etc
1. replacement for unit tests **inside a module**, w/o mocking of module dependencies


## Solution

From the purpose of integration tests described above, 
- 2 and 3 can and should be covered by functional tests as they validate real scenarios and so bring more value
- 1 and 4 still makes sense for integration tests, but only in scope of one instance (single-area application or service) because this is where in-process extension points are supported  

Based on this, it makes sense to run integration tests for a single-area application instance (or for a service in the future).

It should be taken into account that in case of multiple single-area application, DB is shared between all of them.

Changes to integration tests:
1. Break down tests by module (according to new module structure).
  1. Testing package declares which Magento package it depends on
  1. Testing packages are deployed on the same instances their dependencies are deployed. Deployment can be managed in the same way as Magento feature packages
1. Each instance has a dedicated DB (cache, other resources) for tests, so tests can run in isolation
1. Integration tests must not make network calls to other instances

### Scope of Work

Proposed changes require revisiting all or almost all of existing integration tests which is a lot of work.

For existing tests:
- no changes
- clearly document that old integration tests can be run only on a monolithic deployment
- some tests may be converted to the new tests if required during Service Isolation work

For new tests:
- located in a Magento module (separate or the one being tested) 
- MUST NOT execute network calls to controllers that may belong to a different instance (either because of different area or different service)
