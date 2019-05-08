# Magento MFTF Tests Versioning and Backward compatibility Policy
 
## Goals and Requirements
1. Release MFTF Tests as separate magento package on repo.magento.com
2. Define versioning strategy for MFTF test packages
3. Outline all what is considered Backward Incompatible change in MFTF Tests
4. List of what should be implemented

## Backwards Compatibility Definition for MFTF Tests
Backwards compatibility is when tests undergoes changes, but allows to achieve same testing results as before and be compatible with potential test customizations.

Let's classify changes from different perspectives:

- **Test Flow change (Test/ActionGroup)** - Compatible modification of a test flow would not diminish the original set of actions in test. Some changes may change action's sequence (behavior), but as long as they allow any extension to achieve same test results without changing test extension (merge file as we call).
- **Test Entity change (Data/Section/Page/Metadata)** - Compatible modification of entities is usually an adding new ones or updating `value` of existing one in a way when any of the test will **NOT** require updates.
- **Test Annotation change** - Can be changed without limitation and will be Backward Compatible change, but removing or changing `<group />` annotation will be considered as Backward Incompatible change.
- Changes which are deleting and/or renaming (Test/Action Group/Data/Metadata/Page/Section/Action)'s id attribute will be considered as Backward Incompatible change. Changing reference to Data entity will also be considered as as Backward Incompatible change.

## Versioning Policy
An approach of defining what each release should include was taken from [Semantic Versioning](https://semver.org/).

3-component version numbers
---------------------------

    X.Y.Z
    | | |
    | | +-- Backward Compatible changes (bug fixes)
    | +---- Backward Compatible changes (new features)
    +------ Backward Incompatible changes

### Z Release
  Patch version **Z** MUST be incremented if only backward compatible changes to tests are introduced.
  A fix which aims to resolve test flakiness. It can be done by updating unreliable selector, adding wait for element, updating data entity value.
  
### Y Release
  Minor version **Y** MUST be incremented if new, backwards compatible Test or Test Entity is introduced.
  It MUST be incremented if any Test or Test entity is marked as deprecated.
  It MAY include patch level changes. Patch version MUST be reset to 0 when minor version is incremented.

### X Release
  Major version **X** MUST be incremented if any backwards incompatible changes are introduced to the Test or Test Entity.
  It MAY include minor and patch level changes. Patch and minor version MUST be reset to 0 when major version is incremented.

## Implementation tasks
1. Add Semantic Version analyzer to be able automatically define release type of MFTF tests package.
2. Update publication infrastructure to exclude tests from `magento2-module` package type.
3. Introduce publication functionality for publishing `magento2-test-module` package type.
4. Create metapackage with test packages only for each Magento edition.

## Version Increase Matrix
  
  |Entity Type|Change|Version Increase|
  |---|---|---|
  |ActionGroup|`<actionGroup>` added|MINOR
  | |`<actionGroup>` removed|MAJOR
  | |`<actionGroup>` `<action>` added|MINOR
  | |`<actionGroup>` `<action>` removed|MAJOR
  | |`<actionGroup>` `<action>` type changed|PATCH
  | |`<actionGroup>` `<action>` attribute changed|PATCH
  | |`<actionGroup>` `<argument>` added|MAJOR
  | |`<actionGroup>` `<argument>` removed|MAJOR
  | |`<actionGroup>` `<argument>` changed|MAJOR
  |Data|`<entity>` added|MINOR
  | |`<entity>` removed|MAJOR
  | |`<entity>` `<array>` added|MINOR
  | |`<entity>` `<array>` removed|MAJOR
  | |`<entity>` `<array>` `<item>` removed|PATCH
  | |`<entity>` `<field>` added|MINOR
  | |`<entity>` `<field>` removed|MAJOR
  | |`<entity>` `<required-entity>` added|MAJOR
  | |`<entity>` `<required-entity>` removed|MAJOR
  | |`<entity>` `<var>` added|MAJOR
  | |`<entity>` `<var>` removed|MAJOR
  |Metadata|`<operation>` added|MINOR
  | |`<operation>` removed|MAJOR
  | |`<operation>` changed|PATCH
  |Page|`<page>` added|MINOR
  | |`<page>` removed|MAJOR
  | |`<page>` `<section>` added|MINOR
  | |`<page>` `<section>` removed|MAJOR
  |Section|`<section>` added|MINOR
  | |`<section>` removed|MAJOR
  | |`<section>` `<element>` added|MINOR
  | |`<section>` `<element>` removed|MAJOR
  | |`<section>` `<element>` changed|PATCH
  |Test|`<test>` added|MINOR
  | |`<test>` removed|MAJOR
  | |`<test>` (before/after) `<action>` added|MINOR
  | |`<test>` (before/after) `<action>` removed|MAJOR
  | |`<test>` (before/after) `<action>` changed|PATCH
  | |`<test>` (before/after) `<action>` type changed|PATCH
  | |`<test>` `<annotations>` `<annotation>` added|PATCH
  | |`<test>` `<annotations>` `<annotation>` changed|PATCH
  | |`<test>` `<annotations>` `<annotation>` GROUP removed|MAJOR

---------------------------

 ⃰ - `<action>` === (actionGroup, acceptPopup, actionGroup, amOnPage, amOnUrl, amOnSubdomain, appendField, assertArrayIsSorted, assertArraySubset, assertElementContainsAttribute, attachFile, cancelPopup, checkOption, clearField, click, clickWithLeftButton, clickWithRightButton, closeAdminNotification, closeTab, comment, conditionalClick, createData, deleteData, updateData, getData, dontSee, dontSeeJsError, dontSeeCheckboxIsChecked, dontSeeCookie, dontSeeCurrentUrlEquals, dontSeeCurrentUrlMatches, dontSeeElement, dontSeeElementInDOM, dontSeeInCurrentUrl, dontSeeInField, dontSeeInFormFields, dontSeeInPageSource, dontSeeInSource, dontSeeInTitle, dontSeeLink, dontSeeOptionIsSelected, doubleClick, dragAndDrop, entity, executeJS, executeInSelenium, fillField, formatMoney, generateDate, grabAttributeFrom, grabCookie, grabFromCurrentUrl, grabMultiple, grabPageSource, grabTextFrom, grabValueFrom, loadSessionSnapshot, loginAsAdmin, magentoCLI, makeScreenshot, maximizeWindow, moveBack, moveForward, moveMouseOver, mSetLocale, mResetLocale, openNewTab, pauseExecution, parseFloat, performOn, pressKey, reloadPage, resetCookie, submitForm, resizeWindow, saveSessionSnapshot, scrollTo, scrollToTopOfPage, searchAndMultiSelectOption, see, seeCheckboxIsChecked, seeCookie, seeCurrentUrlEquals, seeCurrentUrlMatches, seeElement, seeElementInDOM, seeInCurrentUrl, seeInField, seeInFormFields, seeInPageSource, seeInPopup, seeInSource, seeInTitle, seeLink, seeNumberOfElements, seeOptionIsSelected, selectOption, setCookie, submitForm, switchToIFrame, switchToNextTab, switchToPreviousTab, switchToWindow, typeInPopup, uncheckOption, unselectOption, wait, waitForAjaxLoad, waitForElement, waitForElementChange, waitForElementNotVisible, waitForElementVisible, waitForJS, waitForLoadingMaskToDisappear, waitForPageLoad, waitForText, assertArrayHasKey, assertArrayNotHasKey, assertArraySubset, assertContains, assertCount, assertEmpty, assertEquals, assertFalse, assertFileExists, assertFileNotExists, assertGreaterOrEquals, assertGreaterThan, assertGreaterThanOrEqual, assertInstanceOf, assertInternalType, assertIsEmpty, assertLessOrEquals, assertLessThan, assertLessThanOrEqual, assertNotContains, assertNotEmpty, assertNotEquals, assertNotInstanceOf, assertNotNull, assertNotRegExp, assertNotSame, assertNull, assertRegExp, assertSame, assertStringStartsNotWith, assertStringStartsWith, assertTrue, expectException, fail, dontSeeFullUrlEquals, dontSee, dontSeeFullUrlMatches, dontSeeInFullUrl, seeFullUrlEquals, seeFullUrlMatches, seeInFullUrl, grabFromFullUrl)
