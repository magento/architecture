# Phase 1 - MVP

With phase 1 we are planning to develop next functionality: 
- Retrieving source data endpoints
    * Source Retrieving: (local, remote file, base64 encoded)
- Import process
    * Import of Product Advanced Prices
    * Import of Stock / Sources 
    * Simple products import

## Source management
### Retrieving source data endpoints

So import will start from uploading import Source file. Currently we will support "csv" files format only

POST  `/V1/import/sources/csv`
 
This request can accept files from different sources:
- Local file path
- Direct link to the file
- Base64 encoded file content 
 
#### Local file path

Path is relative from Magento Root folder

```
{
  "uuid": "UUID",
  "source_data": {
    "source_type": "local",
    "source_data": "var/catalog_product.csv" 
  } 
  "format": {
    "delimiter": "string",
    "enclosure": "string",
    "escape": "string",
    "multiple_value_separator": "string"
  }
}
```

#### Direct link to the file

```
{
  "uuid": "UUID",
  "source_data": {
    "source_type": "remote",
    "source_data": "http://some.domain/file.csv" 
  } 
  "format": {
    "delimiter": "string",
    "enclosure": "string",
    "escape": "string",
    "multiple_value_separator": "string"
  }
}
```

#### Upload file (Base64 encoded)

```
{
  "uuid": "UUID",
  "source_data": {
    "source_type": "upload_file",
    "source_data": "c2t1LHN0b3JlX3ZpZXdfY29kZSxhdHRya..." 
  } 
  "format": {
    "delimiter": "string",
    "enclosure": "string",
    "escape": "string",
    "multiple_value_separator": "string"
  }
}
```

### Return values

As return user will receive:

| Key | Value |
| --- | --- |
| uuid | Imported File ID |
| status | Status of this file. Possible values: completed, uploaded, failed |
| error | Error message if exists |

Example:

```
{
    "uuid": null
}
```

### Update Uploaded Source Format

Update of 

PUT  `/V1/import/sources/csv/:uuid`

```
{
  "uuid": "UUID",
  "format": {
    "delimter": "string",
    "enclosure": "string",
    "escape": "string",
    "multiple_value_separator": "string"
  }
}
```

### Get List of sources

GET `/V1/import/sources/?searchCriteria`

Will return list of Source that was uploaded before.

```
{
  "sources": [
  {
      "uuid": "uuid",
      "format": {
          "delimiter": "string",
          "enclosure": "string",
          "escape": "string",
          "multiple_value_separator": "string"
      }
  },
  {
        "uuid": "uuid",
        "format": {
            "delimiter": "string",
            "enclosure": "string",
            "escape": "string",
	    "multiple_value_separator": "string"
        }
    }
    .....
  ],
  "search_criteria": {
      "filter_groups": [
        {
          "filters": [
            {
              "field": "string",
              "value": "string",
              "condition_type": "string"
            }
          ]
        }
      ],
      "sort_orders": [
        {
          "field": "string",
          "direction": "string"
        }
      ],
      "page_size": 0,
      "current_page": 0
    },
    "total_count": 0
}
```

### Base CSV Parser

As in first iteration we will support CSV format only, will be implemented CSV reader.

#### Problem
In case of heavy import data, it may require a lot of memory resources to hold all information from the file in the memory.

#### Solution
Will be implemented CSV reader that will read source partially, process information and then proceed with next part.

### Asynchronous web endpoints
As we are moving in service isolation approach, we will support 2 ways of communicating between Asynchronous Import and Magento.

* Service installed with monolith: will be user MFQF and Direct usage of Service Contracts
* Service installed independently: HTTP communication by using Bulk API


## Start Import Endpoint

### Main endpoint

Current Endpoint starts import process based on FileID. Where Module will read file, split it into message and send to Async API

POST  `/V1/imports`

Start Import

```
{
    "import": {
        "uuid": "",
        "source_uuid": "",
        "import_type": "catalog_product",
        "validation_strategy": "string",
        "allowed_error_count": 0,
        "import_image_archive": "string",
        "import_images_file_dir": "string",
	"convertingRules": [
          {
              "name": "string",
              "parameters": [
                  {"name": "PARAM NAME", "value": mixed},
		  {"name": "PARAM NAME", "value": mixed}
              ],
              "sort": "string",
          }
    	]
    }
    
}
```

| Key | Value |
| --- | --- |
| uuid | UUID of the import |
| source_uuid | UUID of the source |
| import_type | Type of the import: products, categories, prices, etc ... |
| behaviour | Import behaviour (add_update, delete, update, add, replace) |
| import_image_archive | Relative path to product images archive file |
| import_images_file_dir | Relative path to product images files |
| validation_strategy | Moved from main standard Import, not sure if we will use if |
| allowed_error_count | How many errors allowed to be during the import |

*convertingRules*

| Key | Value |
| --- | --- |
| name | Name of the rule - will be defined in configuration xml |
| parameters | array of paramethers. Flexible array here, will depend from method used |
| sort | Sorf order of rules execution |


#### Return

```
{
	"uuid": string	
}
```

### Get import status

Receive information about imported file
GET  `/V1/imports/:uuid`

#### Return

Will be returned list of objects that we tried to import

```
{
  "status": "string", 
  "errors": [],
  "created_at": "string",
  "finished_at": "string",
  "import": {
        "uuid": "",
        "source_uuid": "",
        "import_type": "catalog_product",
        "validation_strategy": "string",
        "allowed_error_count": 0,
        "import_image_archive": "string",
        "import_images_file_dir": "string"
    }
}
```

## Import types

MVP will implement import of following objects:

* Advanced Pricing
    * Add/Update
    * Delete
* Simple Products import
    * Add/Update
    * Delete
* Single stock inventory
    * Add/Update
* Multi stock inventory
    * Add/Update
    * Delete


# Phase 2

### Partial Source Upload

Import of big file also can be divided in several parts.
For this case we have separate endpoint

POST  `/V1/import/sources/csv/partial/`

Input request will looks like:

```
{
  "uuid": "UUID",
  "source_data": {
    "source_type": "upload_file",
    "source_data": "c2t1LHN0b3JlX3ZpZXdfY29kZSxhdHRya...",
    "data_hash" : "sha256 encoded data of the full 'import_data' value",
    "pieces_count": "5"
    "piece_number": "1"
  } 
  "format": {
    "delimiter": "string",
    "enclosure": "string",
    "escape": "string",
    "multiple_value_separator": "string"
  }
}
```

where *source_data* is a 1/N part of the whole content, and *data_hash* contains sha256 hash of full source_data body.

`pieces_count` - its an amount of pieces that will be transferred for 1 file. We need it to be sure that import is completed and then we could detect if it was successfully finished or failed

`piece_number` - its a number that detects which part of file currently transferred. This is required to have to support Asynchronous File import when we dont need to send parts in correct sequence

Those parts could be send asynchronously. They will be merged together after all data are transferred.

### Delete Imported Source Format

DELETE  `/V1/import/sources/:uuid`

### Get single import operation status

Receive information about imported file
GET  `/V1/import/operation/:uuid`

#### Return

Will be returned specific object that we tried to import

```
{
  "status": "string", 
  "error": "string",
  "uuid": 0,
  "entity_type": "catalog_product, customers ....",
  "user_id": "User ID who created this request",
  "user_type": "User Type who created this request",
  "items": [
  {
    "uuid": 0,
    "status": "",
    "serialized_data": "",
    "result_serialized_data": "",
    "error_code":"",
    "result_message":""
  }]
}
```

| Key | Value |
| --- | --- |
| uuid | Imported File ID |
| status | Status of this file. Possible values: completed, not_completed, error |
| error | Error message if exists |
| entity_type | Import type: eg. customers, products, etc ... |
| items | Item that were requested. For do not change interfaces it will be still as an array, but always one item |

##### And Item object will contain

This values are based on magento "magento_operation" table

| Key | Value |
| --- | --- |
| uuid | Entity UUID - defined by customer OR auto-generated by system |
| status | Import status. "pending, failed, processing, completed" |
| serialized_data | Data that was send to Magento, via Async endpoint |
| result_serialized_data | Data that Magento returned for this object after import |
| error_code | Error code |
| result_message | Result message of operation execution |


## Repeatable call endpoint

Main idea, is in case some operation import failure, that user could align input data and repeat import for this particular item

PUT  `/V1/import/operation-id/{uuid}`
 
```
{
  "serialized_data":""
}
```

### Return

```
[]
```


More will come ...
