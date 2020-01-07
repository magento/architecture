## MFTF Test Entities Deprecation Process

### Terminologies

`MFTF` - Magento Functional Testing Framework

`MFTF Test Entities` - Variously typed building blocks which can be used to construct `MFTF Tests`. Types of `MFTF Test Entities` are `Test`, `Action Group`, `Page`, `Section`, `Data`, `Meta Data`, and `Suite`

`MFTF Tests` - Tests written to verify Magento functionality using `MFTF`

`BC` - Backward Compatible

`BIC` - Backward Incompatible

`Deprecation Script` - Script which removes all `MFTF Test Entities` with _deprecated_ annotation from Magento 2 Core code before a Magento Major Release

`MFTF Deprecation Static Check` - Static check script which runs during PR and UR build to make sure no reference of deprecated `MFTF Test Entities` are used in Magento 2 core code

### Overview

Over time with the increasing number of `MFTF Tests`, it becomes necessary for developers to add, remove, or rename `MFTF Test Entities` in existing `MFTF Tests`. Taking [MFTF Test Compatibility Policy][] into consideration, however, not all changes are backward compatible. To deal with BIC changes, MFTF deprecation syntax is introduced and deprecation process is defined in this document. Two version of deprecation syntax are presented to gather feedback from broader audience.

#### Applicable Scope for Deprecation

Besides deprecations at the entity level,  section elements are independent by themselves and can be deprecated without deprecating the entire entity. The following table shows full scope of where deprecation can occur.

|**Entity Types**|**Entity Elements**|**Deprecation Applicable?**|
|---|---|---|
|ActionGroup|`<actionGroup>`|**Yes**
| |`<actionGroup>` `<action>`|No
| |`<actionGroup>` `<argument>`|No
|Data|`<entity>`|**Yes**|
| |`<entity>` `<array>`|No
| |`<entity>` `<array>` `<item>`|No
| |`<entity>` `<data>`|No
| |`<entity>` `<required-entity>`|No
| |`<entity>` `<var>`|No
|Metadata|`<operation>`|**Yes**
|Page|`<page>`|**Yes**
| |`<page>` `<section>`|No
|Section|`<section>`|**Yes**
| |`<section>` `<element>`|**Yes**
|Test|`<test>`|**Yes**
| |`<test>` `<action>`|No
| |`<test>` `<actionGroup>`|No
| |`<test>` `<before/after>` `<action>`|No
| |`<test>` `<before/after>` `<actionGroup>`|No
| |`<test>` `<annotations>` `<annotation>`|No
|Suite|`<suite>`|**Yes**
| |`<suite>` `<include/exclude>` `<group/test/module>`|No
| |`<suite>` `<before/after>` `<action>`|No
| |`<suite>` `<before/after>` `<actionGroup>`|No

### Deprecation Syntax

Two deprecation syntax are presented below. Deprecation syntax 1 uses _deprecated_ attribute for all entities. Deprecation syntax 2 uses _deprecated_ tag for Test and Action Group, and _deprecated_ attribute for other entities the same way as deprecation syntax 1.

#### Test Deprecation Syntax

##### Syntax 1
```xml
<test name="Test" deprecated="Comments about the Entity to be replaced by New Entity or deleted">
    ...
</test>
```
Or
##### Syntax 2
```xml
<test name="Test">
    <annotations>
        <deprecated>Comments about the Entity to be replaced by New Entity or deleted</deprecated>
        <description value="Test description"/>
        ...
    </annotations>
    ...
</test>
```

#### Action Group Deprecation Syntax

##### Syntax 1
```xml
<actionGroup name="ActionGroup" deprecated="Comments about the Entity to be replaced by New Entity or deleted">
    ...
</actionGroup>
```
Or
##### Syntax 2
```xml
<actionGroup name="ActionGroup">
    <annotations>
        <deprecated>Comments about the Entity to be replaced by New Entity or deleted</deprecated>
        <description>ActionGroup description</description>
    </annotations>
    ...
</actionGroup>
```

#### Data Deprecation Syntax

```xml
<entity name="Data" deprecated="Comments about the Entity to be replaced by New Entity or deleted" extends="OtherData" type="SomeType">
    ...
</entity>
```

#### Meta Data Deprecation Syntax

```xml
<operation name="OperationMetaData" deprecated="Comments about the Entity to be replaced by New Entity or deleted" dataType="SomeType" type="create" auth="adminOauth" url="/V1/path" method="POST">
    ...
</operation>
```

#### Page Deprecation Syntax

```xml
<page name="Page" deprecated="Comments about the Entity to be replaced by New Entity or deleted" url="url" area="admin" module="Magento_Module">
    ...
</page>
```

#### Section Deprecation Syntax

```xml
<section name="Section" deprecated="Comments about the Entity to be replaced by New Entity or deleted">
    ...
</section>
```

#### Section Element Deprecation Syntax

```xml
<section name="Section">
    <element name="on" type="multiselect" selector="#on"/>
    <element name="off" deprecated="Comments about the Entity to be replaced by New Entity or deleted" type="multiselect" selector="#off"/>
</section>
```

#### Suite Deprecation Syntax

```xml
<suite name="Suite" deprecated="Comments about the Entity to be replaced by New Entity or deleted">
    ...
</suite>
```

### Syntax Comparison

Each deprecation syntax has pros and cons. Which syntax is more preferable?

|**Deprecation Syntax 1**|**Deprecation Syntax 2**|
|---|---|
|**Pros**|**Pros**|
|1. Consistent for all entity types|1. Preferable declarative syntax|
|2. Deprecation annotation is close to entity, immediately visible|2. Shorter lines|
|**Cons**|**Cons**|
|1. Longer lines impair code readability|1. Inconsistent for all entity types (Unnecessary complexity if adding annotations to all entity types to accomplish consistency)|
| |2. Deprecation annotation can be far away from entity, not immediately visible|

### Deprecation Process

MFTF tests deprecation process is the result of applying [MFTF Test Compatibility Policy][] with consideration of Magento Product Versioning. Magento Product Versioning falls behind semantic versioning convention, for example, Magento 2.x+1 is a major release comparing to Magento 2.x. Because of this convention, MINOR and PATCH MFTF Test changes are practically Backward Compatible (BC) changes and are allowed in all Magento releases, e.g. 2.3.x. MAJOR MFTF Test changes, however, are Backward Incompatible (BIC) changes and are only allowed in Magento Major releases. When BIC changes need to be committed to a branch for future release, deprecation annotation must be used. Deprecated entities will be removed as a one time process after code freeze before Magento Major release.

|**Magento Release/Branch**|**Deprecation Process**|
|---|---|
|Upcoming Major Release (e.g. 2.4.x)|1. Run Deprecation Script to remove ALL deprecated entities from entire MFTF tests|
|Upcoming Major Release Dev Branch (e.g. 2.4-develop)|1. All MAJOR, MINOR and PATCH MFTF test changes are allowed, no deprecation annotation required|
|Upcoming Minor Release (e.g. 2.3.4)|1. Should not run deprecation script|
|Existing Release Dev Branch (e.g. 2.3-develop)|1. MINOR and PATCH MFTF test changes are allowed, no deprecation annotation required|
| |2. MAJOR MFTF test changes must be committed using deprecation annotation|
| |3. Besides the targeted changes, backward compatibility should also be evaluated for other affected changes. In case of BIC, deprecation annotation must also be used|
| |4. In PR, UR build, MFTF Deprecation Static Check to make sure no reference of deprecated entities within Magento 2 core (CE, EE, B2B, PB)|

### Required Implementation
* MFTF to support _deprecated_ syntax, and show warnings when deprecated entities are referenced
* `MFTF Deprecation Static Check` defined in Terminologies section
* `Deprecation Script` defined in Terminologies section

<!-- Link definitions -->

[MFTF Test Compatibility Policy]: versioning-and-backward-compatibility-policy.md