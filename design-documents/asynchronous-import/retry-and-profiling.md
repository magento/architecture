# Phase 3

We planning to build next functionality during this phase:
* Repeatable calls
* Provide endpoints to make possible repeat failed import operation. 

## Repeatable call endpoint

Main idea, is in case some operation import failure, that user could align input data and repeat import for this particular item

PUT  `/V1/import/{fileId}/operation-id/{operationId}`
 
```
{
  "serialized_data":""
}
```

### Return

```
true
```


## Profiling

We want to provide possibility to users to define profiles for their import, so any (or almost any) custom import file can be processed.
Current default Magento CSV import format will stay as default. 

### Create new profile

This call creates new profile for future imports

POST  `/V1/import/profile`

```
{
"profile": [
     	{
        "source_attribute": "string",
        "destination_attribute": "string",
        "processing_rules": "string",
        "taxonomy": "string",
        "values_mapping": [
          {
            "old_value": "string",
            "new_value": "string",
          }
        ]
      }
    ]
}
```


| Key | Value |
| --- | --- |
| source_attribute | Column name (tag name, key) in imported file |
| destination_attribute | Magento attribute code |
| processing_rules | This can be a defined set of possible transformations of imported values: str_replace. trim, .... |
| old_value | Values mapping. Current valus in imported file. As example: "Yes" in import file have to be converted to "1" in Magento import |
| new_value | Values mapping. New value after import in Magento. As example: "Yes" in import file have to be converted to "1" in Magento import  |


#### Return

Returns profile ID.

```
{
	"profile_id": int
}
```

### Update Profile

PUT  `/V1/import/profile/{profileId}`

See “Create new profile“

### Delete Profile 

Delete profile from system

DELETE  `/V1/import/profile/{profileId}`

### Get Profiles

Get profiles list
GET  `/V1/import/profile`

Get by ID
GET  `/V1/import/profile/:id`


After implementation of Profiles logic, start import call will looks like this: 

```
{
	"file_id": int,
	"profile_id": int,
	"type": "products, customers ...."
}
```

