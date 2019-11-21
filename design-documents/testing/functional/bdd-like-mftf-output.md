# Overview

Magento Functional Testing Framework (MFTF) aims to generate valuable output for different cases:

* **Debug** - understanding what happens _under the hood_, what are the names of steps and what selectors were used.
  ```gherkin
  [amOnAdminLoginPage] I am on Admin URL (Admin URL: "/god/admin")
  [fillUsername] I fill username field with "god" (xpath: "#username")
  [fillPassword] I fill password field with "79XedTOBjPiHFtLt" (xpath: "#login")
  [clickOnSignIn] I click Login button (xpath: ".actions .action-primary")
  [waitForPageload] I wait for page load (timeout: 30)
  [seeAdminLoginUrl] I see Admin URL in current url (Admin URL: "/god/admin")
  PASSED
  ```

* **Non-verbose** (default) - showing just informational messages.
  ```gherkin
  I am on Admin URL
  I fill username field with "god"
  I fill password field with "79XedTOBjPiHFtLt"
  I click Login button
  I wait for page load
  I see Admin URL in current url
  PASSED
  ```

* **Behavioral** - focused on behavior, not specific actions.
  ```gherkin
  I am logged in as Administrator (login: "god", password: "xyz123")
  I am on Product Grid
  I click Add Product button
  I save screenshot
  ```

## PoC

https://github.com/magento/magento2-functional-testing-framework/issues/364

## Usage

Output should be consistent across the outputs:
- Allure Report
- CLI Report
- Documentation

## Behavioral output

The real value of such output is possibility to confront the Business Requirements with Test Scenario without any extra interpretation. The content should be obvious to non-technical person.

Example of such is Sylius Framework: https://github.com/Sylius/Sylius/blob/48729b86334ba78d8f2011b9743389024c334d56/features/checkout/paying_for_order/paying_with_paypal_during_checkout.feature#L7-L24

### User-defined Label Attribute

Adding extra attribute to XML nodes that allows to define user-friendly label for element / action / action group.

* Element
 ```xml
 <element name="password" type="input" selector="#login" label="Password Input"/>
 ```
* Performed Action
  ```xml
  <click selector=".niceButton" stepKey="clickButton" label="Click Nice Button"/>
  ```
* Action Group
  ```xml
  <actionGroup name="resetProductGridToDefaultView" label="Reset Product Grid">
  ```
* Action Group Execution
  ```xml
  <actionGroup ref="someActionGroup" label="Running Action Group">
  ```

#### Benefits

* MFTF user is able to provide domain-consistent labels.

### Generated Label

* Element
  ```xml
  <element name="password" type="input" selector="#login"/>
  ```
  generates:
  ```gherkin
  Password input
  ```
* Action Group
  ```xml
  <actionGroup name="StorefrontFillCustomerAccountCreationFormActionGroup">
      <arguments>
          <argument name="customer" type="entity" />
      </arguments>

      <fillField  stepKey="fillFirstName" userInput="{{customer.firstname}}" selector="{{StorefrontCustomerCreateFormSection.firstnameField}}" />
      <fillField  stepKey="fillLastName"  userInput="{{customer.lastname}}" selector="{{StorefrontCustomerCreateFormSection.lastnameField}}" />
      <fillField  stepKey="fillEmail" userInput="{{customer.email}}" selector="{{StorefrontCustomerCreateFormSection.emailField}}"/>
      <fillField  stepKey="fillPassword" userInput="{{customer.password}}" selector="{{StorefrontCustomerCreateFormSection.passwordField}}"/>
      <fillField  stepKey="fillConfirmPassword" userInput="{{customer.password}}" selector="{{StorefrontCustomerCreateFormSection.confirmPasswordField}}"/>
  </actionGroup>
  ```
  generates:
  ```gherkin
  Action group summary:
   - Storefront Fill Customer Account Creation Form

  Requires arguments:
   - 'customer' entity

  Steps:
  1. Fill First Name
  2. Fill Last Name
  3. Fill Email
  4. Fill Password
  5. Fill Confirm Password
  ```

#### Benefits

* Automatic label generation enforces MFTF user to provide descriptive `name`s / `stepKey`s in Test Scenarios

# Implementation

1. **MFTF Naming Rules**

  There's no standard regarding naming in MFTF Tests. One should be formed and published in Magento DevDocs, as well as in the MFTF repository. Tests migrated / created after forming such rules should follow the standards.

2. **Naming interpretation strategy**

  Having consistent Naming for Tests, we are able to get benefits of it. We need to prepare strategy that describes how we convert names to human-readable labels, for example:

  *  `StorefrontFillCustomerAccountCreationFormActionGroup` -> `Storefront Fill Customer Account Creation Form`
  * `fillPasswordField` -> `Fill Password Field`

  Having strategies established, Issues should be created to cover implementation in `magento/magento2-functional-testing-framework` repository.

3. **Codeception CLI and Allure reports**

  Having all previous done, the final step is to implement adapters both for CLI output and Allure Reports. As a result, we should receive the output with desired verbosity:
  * Debug
  * Non-verbose
  * Behavioral
