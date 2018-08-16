# Declarative Theme Media Breakpoints in `etc/view.xml`

## Context

Currently we declare theme breakpoints through LESS variables, notably `@screen__m` declares the breakpoint at which the design should change to a mobile design.

Page Builder requires more dynamic generation of styles due to the dynamic nature of how the content is created. We have background image functionality which requires the generation of CSS to provide a high performant solution, due to this we need to be able to determine the current themes mobile breakpoint.

## Implementation Strategy

Currently themes implement a `etc/view.xml` configuration file which contains module specific configuration.

For instance the file for both `luma` and `blank` contains the following nodes:

```xml
<vars module="Magento_Catalog">
	...
    <var name="breakpoints">
        <var name="mobile">
            <var name="conditions">
                <var name="max-width">767px</var>
            </var>
            <var name="options">
                <var name="options">
                    <var name="nav">dots</var>
                </var>
            </var>
        </var>
    </var>
    ...
</vars>
```

We would look to unify the breakpoint configuration for all themes under a single node and not specific for a particular module. 

We have 3 different options for this proposal

### Define full breakpoint

```xml
<breakpoints>
    <breakpoint name="large">@media(-webkit-min-device-pixel-ratio:1.5),(min--moz-device-pixel-ratio:1.5),(-o-min-device-pixel-ratio:3/2),(min-resolution:1.5dppx)</breakpoint>
    <breakpoint name="desktop">@media all and (min-width: 960px) and (max-width: 1199px)</breakpoint>
    <breakpoint name="tablet">@media all and (min-width: 767px)</breakpoint>
    <breakpoint name="mobile">@media all and (min-width: 480px)</breakpoint>
</breakpoints>
```

**Pros:**

- Greater control over the point at which we display mobile content

**Cons:**

- Harder to import into the LESS files at a later date

### Define variables to be later determined how to be consumed

```xml
<var name="breakpoints_variables">
    <var name="large">767px</var>
    <var name="desktop">767px</var>
    <var name="mobile">480px</var>
    <var name="ratio">1.5</var>
</var>
```

**Pros:**

- Much simpler configuration
- Could be imported into LESS via `@screen__NAME`, similar to how we currently implement this.

**Cons:**

- Values may seem confusing as the underlying logic which determines whether these are max-width / min-width or other properties is hidden within LESS, PHP, JS.

### Build conditions in the XML

```xml
<var name="breakpoints">
    <var name="mobile">
        <var name="conditions">
        	<var name="max-width">767px</var>
        </var>
    </var>
    <var name="desktop">
        <var name="conditions">
            <var name="min-width">960px</var>
        	<var name="max-width">1280px</var>
        </var>
    </var>
    <var name="large">
    	<var name="conditions">
        	<var name="min-resolution">1.5dppx</var>
        </var>
    </var>
</var>
```

**Pros:**

- Existing `Magento_Catalog` entries already use this approach
- We have a better understanding of what the breakpoint wants to acheive without having to anaylse the string as in option 1

**Cons:**

- Might be difficult to build certain complex media queries

## Overall Concerns

This does mean, in the current situation, we have 2 sources of truth for breakpoints within a theme. Those defined within the themes LESS and those defined in the `etc/view.xml`. We do already have this issue with the `Magento_Catalog` module declaring these breakpoints for the product gallery.