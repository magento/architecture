# Default Processing Rules

Default Magento CSV formal contains data, which are cannot be directly imported into Magento.

For processing different types of Import, we have to define default rules processing, for a case when User want to import CSV data in default Magento Format.

In additional, we want to provide possibility to developers to define own default rules sequence for minimize requests body and define own rules.


## Configuration file

For this case we will provide new configuration file for the module.

`default_rules_sequence.xml`


Example for Advanced Pricing import Rules sequence.

```
<?xml version="1.0"?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/sequence.xsd">

    <sequence name="default_advanced_prices">
        <rule name="WebsiteNameToId" sortOrder="10">
            <paramether name="apply_to" xsi:type="array">
                <item name="website_id" xsi:type="string">website_id</item>
            </paramether>
        </rule>
        <rule name="StringToLower" sortOrder="10">
            <paramether name="apply_to" xsi:type="array">
                <item name="price_type" xsi:type="string">price_type</item>
            </paramether>
        </rule>
    </sequence>

</config>
```

### XSD Schema

```
<?xml version="1.0" encoding="UTF-8"?>

<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema">

    <xs:element name="sequences">
        <xs:complexType>
            <xs:sequence>
                <xs:element ref="sequence" minOccurs="1" maxOccurs="unbounded" />
            </xs:sequence>
        </xs:complexType>

        <xs:unique name="uniqueSequenceName">
            <xs:annotation>
                <xs:documentation>
                    Sequence Name have to be unique
                </xs:documentation>
            </xs:annotation>
            <xs:selector xpath="sequence"/>
            <xs:field xpath="@name"/>
        </xs:unique>

    </xs:element>

    <xs:element name="sequence">
        <xs:complexType>
            <xs:sequence>
                <xs:choice minOccurs="1" maxOccurs="unbounded">
                    <xs:element ref="rule" />
                </xs:choice>
            </xs:sequence>
            <xs:attributeGroup ref="sequenceAttributeGroup"/>
        </xs:complexType>
    </xs:element>

    <xs:element name="rule">
        <xs:complexType>
            <xs:sequence>
                <xs:choice minOccurs="1" maxOccurs="1">
                    <xs:element ref="paramethers" />
                </xs:choice>
            </xs:sequence>
            <xs:attributeGroup ref="ruleAttributeGroup"/>
        </xs:complexType>
    </xs:element>

    <xs:element name="paramethers">
        <xs:complexType>
            <xs:sequence>
                <xs:choice minOccurs="1" maxOccurs="unbounded">
                    <xs:element ref="item" />
                </xs:choice>
            </xs:sequence>
            <xs:attributeGroup ref="parametherAttributeGroup"/>
        </xs:complexType>
    </xs:element>

    <xs:element name="item">
        <xs:annotation>
            <xs:documentation>
                Single parameter
            </xs:documentation>
        </xs:annotation>

        <xs:complexType mixed="true">
            <xs:sequence>
                <xs:any minOccurs="0" maxOccurs="unbounded" />
            </xs:sequence>
            <xs:attributeGroup ref="itemAttributeGroup"/>
        </xs:complexType>
    </xs:element>

    <xs:attributeGroup name="itemAttributeGroup">
        <xs:attribute name="name" type="xs:string" use="required" />
    </xs:attributeGroup>

    <xs:attributeGroup name="parametherAttributeGroup">
        <xs:attribute name="name" type="xs:string" use="required" />
    </xs:attributeGroup>

    <xs:attributeGroup name="sequenceAttributeGroup">
        <xs:attribute name="name" type="xs:string" use="required" />
    </xs:attributeGroup>

    <xs:attributeGroup name="ruleAttributeGroup">
        <xs:attribute name="name" type="xs:string" use="required" />
        <xs:attribute name="sortOrder" type="xs:float" use="optional" />
    </xs:attributeGroup>

</xs:schema>
```

With using this structure we can cover cases for dynamic converting rules: 

```
"convertingRules": [
        {
            "name": "WebsiteCodeToIdConvertor",
            "sort": 10
        },{
            "name": "SomeCustomName",
            "sort": 10,
            "parameters": [1, 3, "param"]
        },
       ....
    ]
```
    