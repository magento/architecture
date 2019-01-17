# Magento default xml encoding

As Magento 2, uses xml files for configuration and static data inputs, 
it should use a specified encoding to avoid errors incase XML documents can contain international characters,
 like Norwegian øæå or French êèé.

## Current state

- No encoding is specified in any xml file used as input or,
 generated via [Xml Parser](https://github.com/magento/magento2/blob/2.3-develop/lib/internal/Magento/Framework/Xml/Generator.php)

## Proposed state

- Use of `UTF-8` as the default encoding for all configuration xmls and generated xmls.

 

