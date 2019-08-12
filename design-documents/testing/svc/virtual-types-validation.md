# Validation of backward incompatible changes for Virtual Types

## Problem

 According to [Versioning Policy](https://devdocs.magento.com/guides/v2.3/extension-dev-guide/versioning/codebase-changes.html) 
 we check changes for all Virtual Types in the same way as changes for API classes.

## Solution

* Upgrade our [Versioning Policy](https://devdocs.magento.com/guides/v2.3/extension-dev-guide/versioning/codebase-changes.html) 
and specify that all this is true only for Virtual types marked as API.

* Agree that non-API Virtual types are private code. According to our best practices we must not use virtual types/clases directly in the code.

* Review our codebase and remove unnecessary ```<!-- @api -->``` annotations related to the Virtual types.

* Introduce a new optional attribute ```api``` for Virtual Types

    Before
    ```xml
        <!-- @api -->
        <virtualType name="shellBackground" type="Magento\Framework\Shell">
            <arguments>
                <argument name="commandRenderer" xsi:type="object">Magento\Framework\Shell\CommandRendererBackground</argument>
            </arguments>
        </virtualType>
    ```
    
    after
    ```xml
        <virtualType name="shellBackground" type="Magento\Framework\Shell" api="true">
            <arguments>
                <argument name="commandRenderer" xsi:type="object">Magento\Framework\Shell\CommandRendererBackground</argument>
            </arguments>
        </virtualType>
    ```

* Explicitly present these changes in our documentation.

### Pros

* More flexibility in changes

* Reliability in recognition of API Virtual Classes

### Cons

* This may be some issues with Virtual Types previously marked as API

* No other, if developers follow our best practices