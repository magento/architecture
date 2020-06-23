# Get available stores for website

### Use case:
Implementing a store switcher; it is necessary to know which stores are available and some basic info about them (i.e. store code)

### Implementation detail:
Based on the store code passed via header (or default), returns the storeConfig for all stores available under the same website.

### Proposed schema
```graphql
type Query {
    availableStores: [StoreConfig] @doc(description: "Get a list of available store views and their config information.")
}

# Existing schema
type StoreConfig @doc(description: "The type contains information about a store config") {
    id : Int @doc(description: "The ID number assigned to the store")
    code : String @doc(description: "A code assigned to the store to identify it")
    website_id : Int @doc(description: "The ID number assigned to the website store belongs")
    locale : String @doc(description: "Store locale")
    base_currency_code : String @doc(description: "Base currency code")
    default_display_currency_code : String @doc(description: "Default display currency code")
    timezone : String @doc(description: "Timezone of the store")
    weight_unit : String @doc(description: "The unit of weight")
    base_url : String @doc(description: "Base URL for the store")
    base_link_url : String @doc(description: "Base link URL for the store")
    base_static_url : String @doc(description: "Base static URL for the store")
    base_media_url : String @doc(description: "Base media URL for the store")
    secure_base_url : String @doc(description: "Secure base URL for the store")
    secure_base_link_url : String @doc(description: "Secure base link URL for the store")
    secure_base_static_url : String @doc(description: "Secure base static URL for the store")
    secure_base_media_url : String @doc(description: "Secure base media URL for the store")
    store_name : String @doc(description: "Name of the store")
    # ... more fields added from other modules and 3rd parties
}
```

Sample query:
```graphql
query {
    availableStores {
        id
        code
        locale
        timezone
        base_url
    }
}
```

Sample response:
```json
{
  "data": {
    "availableStores": [
        {
          "id": 1,
          "code": "default",
          "locale": "en_US",
          "timezone": "America/Chicago",
          "base_url": "http://magento.test/"
        },
        {
          "id": 2,
          "code": "German",
          "locale": "de_DE",
          "timezone": "Europe/Berlin",
          "base_url": "http://magento.test/"
        }
    ]
  }
}
```
