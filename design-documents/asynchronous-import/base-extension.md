# Phase 1

With phase 1 we are planning to develop next functionality: 
- Endpoint to receive file
- Endpoint to start processing
- Endpoint to receive status


```
```


## File Upload Endpoint
### Main endpoint

POST  `/V1/import`
 
This request can accept files from different sources:
- Local file path
- Direct link to the file
- Base64 encoded file content 
 
#### Local file path

```
{
  "importEntry": {
    "file_id": 0,
    "source": {
      "import_data": "/var/www/html/bulk-api/async-import/var/catalog_product.csv",
      "type": "local_path",
      "file_type": "csv"
    }
  }
}
```

#### Direct link to the file

```
{
  "importEntry": {
    "file_id": 0,
    "source": {
      "import_data": "http://some.domain/file.csv",
      "type": "external",
      "file_type": "csv"
    }
  }
}
```

#### Base64 encoded file content 

```
{
  "importEntry": {
    "file_id": 0,
    "source": {
      "import_data": "c2t1LHN0b3JlX3ZpZXdfY29kZSxhdHRyaWJ1dGVfc2V0X2NvZGUscHJvZHVjdF90eXBlLGNhdGVnb3JpZXMscHJvZHVjdF93ZWJzaXRlcyxuYW1lLGRlc2NyaXB0aW9uLHNob3J0X2Rlc2NyaXB0aW9uLHdlaWdodCxwcm9kdWN0X29ubGluZSx0YXhfY2xhc3NfbmFtZSx2aXNpYmlsaXR5LHBya......",
      "type": "base64_encoded_data",
      "file_type": "csv"
    }
  }
}
```

Import of big file also can divided in several parts
In this case input request will looks like:


```
{
  "importEntry": {
    "file_id": 0,
    "source": {
      "import_data": "c2t1LHN0b3JlX3ZpZXdfY29kZSxhdHRyaWJ1dGVfc2V0X2NvZGUscHJvZHVjdF90eXBlLGNhdGVnb3JpZXMscHJvZHVjdF93ZWJzaXRlcyxuYW1lLGRlc2NyaXB0aW9uLHNob3J0X2Rlc2NyaXB0aW9uLHdlaWdodCxwcm9kdWN0X29ubGluZSx0YXhfY2xhc3NfbmFtZSx2aXNpYmlsaXR5LHBya...",
      "data_hash" : "sha256 encoded data of the full 'import_data' value"
      "pieces_count": "5"
      "piece_number": "1",
      "type": "base64_encoded_data",
      "file_type": "csv"
    }
  }
}
```
where *import_data* is a 1/N part of the whole content, and *data_hash* contains sha256 hash of full import_data body.

`pieces_count` - iss amount of pieces that will be transferred for 1 file. We need it to be sure that import is completed and then we could detect if it was successfully finished or failed

`piece_number` - its a number that detects which part of file currently transferred. This is required to have to support Asynchronous File import when we dont need to send parts in correct sequence



Then all following parts of imported file will look like:

```
{
  "importEntry": {
    "file_id": 10,
    "source": {
      "import_data": "c2t1LHN0b3JlX3ZpZXdfY29kZSxhdHRyaWJ1dGVfc2V0X2NvZGUscHJvZHVjdF90eXBlLGNhdGVnb3JpZXMscHJvZHVjdF93ZWJzaXRlcyxuYW1lLGRlc2NyaXB0aW9uLHNob3J0X2Rlc2NyaXB0aW9uLHdlaWdodCxwcm9kdWN0X29ubGluZSx0YXhfY2xhc3NfbmFtZSx2aXNpYmlsaXR5LHBya...",
    }
  }
}
```
where file_id is an ID which you will receive as return from first call.

### Return values

As return user will receive:

| Key | Value |
| --- | --- |
| file_id | Imported File ID |
| file_status | Status of this file. Possible values: completed, not_completed, error |
| error | Error message if exists |

Example:

```
{
	"file_id": 10,
	"file_status": "completed",
	"error": ""
}
```

In case of partial uploads, status will be *not_completed* till whole file will be fully received by system.

## Start File Import Endpoint
### Main endpoint

Current Endpoint starts import process based on FileID. Where Module will read file, split it into message and send to Async API

POST  `/V1/import/start/{fileId}`

Start File Import

```
{
	"entity_type": "catalog_product",
	"behaviour": "add_update, delete, update, add, replace ..."
	"import_image_archive": "string",
      	"import_images_file_dir": "string",
      	"allowed_error_count": 0,
      	"validation_strategy": "string",
      	"empty_attribute_value_constant": "string",
      	"csv_separator": "string",
      	"csv_enclosure": "string",
      	"csv_delimiter": "string",
      	"multiple_value_separator": "string"
}
```

Q&A - Swagger implementation, are there a sence to move TYPE to URL? 
POST  `/V1/import/type/catalog_product/start/{fileId}`

#### Return

```
{
	"file_id": int
	"success":bool,
	"error": "string"
}
```

### Get import status

Receive information about imported file
GET  `/V1/import/:fileId`

#### Return

Will be returned list of objects that we tried to import

```
{
  "file_status": "string", 
  "error": "string",
  "file_id": 0,
  "entity_type": "products, customers ....",
  "items": [
  {
    "id": 0,
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
| file_id | Imported File ID |
| file_status | Status of this file. Possible values: completed, not_completed, error |
| error | Error message if exists |
| entity_type | Import type: eg. customers, products, etc ... |
| items | List of items that were imported. As an array |

##### And Item object will contain

This values are based on magento "magento_operation" table

| Key | Value |
| --- | --- |
| id | Imported File ID |
| status | Import status. "pending, failed, processing, completed" |
| serialized_data | Data that was send to Magento, via Async endpoint |
| result_serialized_data | Data that Magento returned for this object after import |
| error_code | Error code |
| result_message | Result message of operation execution |
