# Eliminate config elements instantiation on the fly

## Current state
GraphQL framework component has introduced a family of config data objects which simplified work with the schema objects.
At the moment config object is responsible for constructing such objects by using Magento ObjectManager.
In general, this is a cheap operation.
But still, it requires some time and depends on the number of elements that have to be created.

## Proposal

So we have a few areas for improvements here.

* Decoupling of operations
We can extract entities creation logic from Config object.
This will simplify class, and remove some dependencies.
* Performance improvement
The main benefit will bring a fact that Config object will not spend time on object creation.
Due to a fact, that config entities are the simplest data objects GraphQL config can create them only once and store them to cache as serialized objects. 

To achieve this we can introduce specific serializer for config data. `Magento\Framework\Config\Data` can be configured to use a custom serializer though the DI.
```xml
    <virtualType name="Magento\Framework\GraphQl\Config\Data" type="Magento\Framework\Config\Data">
        <arguments>
            <argument name="reader" xsi:type="object">Magento\Framework\GraphQlSchemaStitching\Reader</argument>
            <argument name="cacheId" xsi:type="string">Magento_Framework_GraphQlSchemaStitching_Config_Data</argument>
            <argument name="serializer" xsi:type="string">Magento\Framework\GraphQl\Config\Data\Serializer</argument>
        </arguments>
    </virtualType>
```
In a fact, Magento knows all config data object.
Magento does not use external resources as a configuration source.
So we can create a lightweight serializer which may utilize 
[serialize](https://secure.php.net/manual/en/function.serialize.php)
/*
[unserialize](https://secure.php.net/manual/en/function.unserialize.php) functions with all known GraphQL configuration data objects mentioned in *`allowed_classes` option.
Mentioned above guarantees us that we are addressing security concerns connected to `unserialize` function usage for our case.