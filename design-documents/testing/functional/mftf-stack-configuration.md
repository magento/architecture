# Overview

Technically, Magento Functional Testing Framework has two types of stack dependencies:
1. Responsible for maintaining tests (PHP, Composer, Allure)
2. Setup that we run test on (Selenium, ChromeDriver, GeckoDriver, Phantom) and it's dependencies (Java)

## Current State

Current MFTF codebase requires to keep whole stack at single environment. Either dependencies necessary to maintain tests and the webdriver stack on which we run tests needs to be located at single machine. MFTF expects dependencies to be available by running local shell command (eg. `java -version 2>&1`).

Only ChromeDriver is supported at the moment.

## Problem

Current approach forces Developers to use single, monolith testing environment. It also couples MFTF with very specific stack, limiting it's further flexibility to use different web drivers.

As an example - Allure can be easily dockerized and used as an external service, MFTF does not support this setup.

## Desired state

1. Having single `.env` file containing environment-specific configuration for tested instance (Magento base URL, admin credentials)
2. Separate test environment configuration containing:
  - Selenium setup type (hub / standalone), connection details, path
  - WebDrivers configuration (currently ChromeDriver only, for future it MFTF should support Firefox / Safari / BrowserStack)
  - Allure server configuration (path or connection details)

## Backward compatibility

Configuration should have fallbacks to default paths for all the required libraries, to avoid breaking current setups that are build in a monolith way.
