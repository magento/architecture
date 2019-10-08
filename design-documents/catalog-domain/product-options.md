# Product options

# Definition 
Product options is a set of predefined variants how a buyer can customize product before purchase.
The purpose of product options are displaying variants to a buyer at PDP.    

## Product option may include 
* title: explains option purpose.
* isRequired: (flag) says is option mandatory for selection.  
* isMulti: (flag) says does the option support selection of multiple values.
* values: set of option values.

## Product Option values:
* title: explains option selection value.
* price: explains price adjustment if applicable.
* link: (extension) allows to specify a link associated with a value.

```json
{
  "id": "[UUID]",
  "title": "Option title",
  "isRequired": true,
  "isMulti": false,
  "values": [
    {
      "id": "[UUID]",
      "title": "Value Title",
      "price": null,
      "link": null
    }  
  ]
}
```
