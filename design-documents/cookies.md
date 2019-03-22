# Cookies in Magento

Original document: https://docs.magento.com/m2/ce/user_guide/stores/cookie-reference.html

The goal of this document is to align cookies existing in Magento to features relying on them, so a customer may choose to disable a feature and not populate a specific cookie.


| Cookie  | Module | Related configuration |
| ------- | -------- | ------------- |
| `guest-view` | `Magento_Sales` | Guest orders view. Used in "Orders and Returns" widget. |
| `login_redirect` | `Magento_Customer`, `Magento_Checkout`  | Used in minicart for logged in customers if "Shopping Cart Sidebar" > "Display Shopping Cart Sidebar" is set to "Yes" |
| `mage-messages` | `Magento_Theme` | No option to disable |
| `mage-translation-storage`  | `Magento_Translation` | Used when "Translation Strategy" configured as "Dictionary (Translation on Storefront side)". |
| `mage-translation-file-version` | `Magento_Translation`. | Tracks version of translations in local storage. Used when "Translation Strategy" configured as "Dictionary (Translation on Storefront side)". |
| `product_data_storage`  | `Magento_Catalog` | |
| `recently_compared_product`  | `Magento_Catalog` | |
| `recently_compared_product_previous`  | `Magento_Catalog` | |
| `recently_viewed_product`  | `Magento_Catalog` | |
| `recently_viewed_product_previous`  | `Magento_Catalog` | |
| `stf`  | `Magento_SendFriend` | |
| `X-Magento-Vary` (note: original document has typo)  | `Magento_PageCache` | |
| `amz_auth_err`  | Amazon Pay CBE | |
| `amz_auth_logout`  | Amazon Pay CBE | |
| `form_key`  | Multiple modules. Any module that has forms should use it. | |
| `mage-cache-sessid` | `Magento_Customer`, `Magento_Persistent` | |
| `mage-cache-storage`  | `Magento_Customer`, `Magento_Persistent`, `Magento_NegotiableQuote` | |
| `mage-cache-storage-section-invalidation`  | `Magento_Customer` | |
| `section_data_ids`  | `Magento_Customer` | |
| `persistent_shopping_cart`  | `Magento_Persistent` | |
| `private_content_version`  | `Magento_PageCache`, `Magento_Customer` | |
| `store`  | `Magento_Store` | |
| `mage-banners-cache-storage` | `Magento_Banner` (Magento Commerce). Description: Local storage for Banner functionality. | |
