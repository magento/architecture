# REST API

## 1. Source Data Upload Endpoint (optional step)

The entry point for start uploading Source data to instance storage (Currently supported only local instance)

**Request:**

`POST  /V1/import/source`

```
{
  "source": {
    "sourceType": "http",
    "sourceDefinition": "http://some.domain/file.csv",
    "sourceDataFormat": "CSV"
  }
}
```

**Response:**

```
{
  "filename": "var/import/filename.csv"
}
```


## 2. Start Import Endpoint

Single entry point for **partial** reading and import of data.

`POST  /V1/import/csv`

```
{
  "source": {
    "sourceType": "http"
    "sourceDefinition": "http://some.domain/file.csv"
    "sourceDataFormat": "CSV"
  }
  "format': {
    "escape": "|"
    "enclosure": "|"
    "delimiter": "|"
    "multipleValueSeparator": "|"
    "extensionAttributes": []
  }
  "convertingRules": [
    {
      "identifier": "string_replace"
      "applyTo": ["sku", "name"]
      "parameters": {
        "search": "_"
        "replace": "-"
      }
    }
  ]
  "import": {
    "importType": "advanced_pricing"
    "importBehaviour": "add"
    "extensionAttributes": []
  }
}
```

Where is:
- `source` is **required**. Describes how to retrieve data from Source
  - `sourceType` is **required**. Example: `HTTP|HTTPS|local_file|base_64`. Could contains adiditonal information as `encoded string` if needed.
  - `sourceDefinition` is **required**. Depends on `sourceType`. Example: `http://some.domain/file.csv|var/import/filename.csv`
  - `sourceDataFormat` is **optional**. Needed for preliminary validation before data retrieving.
- `format` is **optional**. Describes how to parse data.
- `convertingRules[]` is **optional**. Describes how to change data before Import process. Additional data could be placed in `parameters`. 
- `import` is **required**. Describes how to import data.
  - `importType` is **required**. Example: `stock|advanced_pricing|product`.
  - `importBehaviour` is **required**. Example: `add|delete`

**Result:**

```
{
  "uuid": "uuid_string"
}
```
