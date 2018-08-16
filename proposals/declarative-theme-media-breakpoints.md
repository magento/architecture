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

### Long Term Solution

In the long term we'd like to be able to support population of LESS variables from the themes XML configuration. This allows for the greater Magento system to have an understanding of when various different content should be displayed, this could be represented in the themes XML as follows:

```
<theme_variables>
    <var name="screen__m">767px</var>
    <var name="default_font_size">14px</var>
</theme_variables>
```

These variables names would be converted over to `@screen__m` and `@default_font_size` respectively.

### Short Term Page Builder Specific Solution

The long term solution has a large requirement of resource and careful design to ensure it's implementation fits it's purpose correctly.

Page Builder requires an understanding of break points now ,_similar to how Magento_Catalog does currently_, due to this we can introduce new configuration specific for our module, within the `blank` and `luma` themes` as follows:

```
<vars module="Magento_PageBuilder">
    <var name="breakpoints">
        <var name="mobile">767px</var>
    </var>
</vars>
```

This allows us to understand the mobile breakpoint and correctly render our dynamic CSS. Utilising `view.xml` file we can also add defaults to our module within `Magento_PageBuilder/etc/view.xml` to ensure that mobile background images are always displayed.

## Overall Concerns

This does mean, in the current situation, we have 2 sources of truth for breakpoints within a theme. Those defined within the themes LESS and those defined in the `etc/view.xml`. We do already have this issue with the `Magento_Catalog` module declaring these breakpoints for the product gallery.