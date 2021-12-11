# Overview

The `Web Api` currently only allows for GET, PUT, POST, and DELETE.  I would like to add OPTIONS to this list.  The module I am currently working on depends on another service connecting in though the web api using the OPTIONS HTTP method.

## Details

Here is where the allowed methods are listed.
https://github.com/magento/magento2/blob/2.2-develop/app/code/Magento/Webapi/etc/webapi.xsd#L27

```
        <xs:attribute name="method" use="required">
            <xs:simpleType>
                <xs:restriction base="xs:string">
                    <xs:enumeration value="GET"/>
                    <xs:enumeration value="PUT"/>
                    <xs:enumeration value="POST"/>
                    <xs:enumeration value="DELETE"/>
                </xs:restriction>
            </xs:simpleType>
        </xs:attribute>
```

I propose it be changed to this:

```
        <xs:attribute name="method" use="required">
            <xs:simpleType>
                <xs:restriction base="xs:string">
                    <xs:enumeration value="GET"/>
                    <xs:enumeration value="PUT"/>
                    <xs:enumeration value="POST"/>
                    <xs:enumeration value="DELETE"/>
                    <xs:enumeration value="OPTIONS"/>
                </xs:restriction>
            </xs:simpleType>
        </xs:attribute>
```
