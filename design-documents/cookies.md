# Cookies in Magento

Original document: https://docs.magento.com/m2/ce/user_guide/stores/cookie-reference.html

The goal of this document is to align cookies existing in Magento to features relying on them, so a customer may choose to disable a feature and not populate a specific cookie.


| Cookie  | Module |
| ------- | -------- |
| `guest-view` | `Magento_Sales` |
| `login_redirect` | `Magento_Customer`, `Magento_Checkout`  |
| `mage-messages` | `Magento_Theme` |
| `mage-translation-storage`  | `Magento_Translation` |
| `mage-translation-file-version` | `Magento_Translation`. Description: Tracks version of translations in local storage. |
| `product_data_storage`  | `Magento_Catalog` |
| `recently_compared_product`  | `Magento_Catalog` |
| `recently_compared_product_previous`  | `Magento_Catalog` |
| `recently_viewed_product`  | `Magento_Catalog` |
| `recently_viewed_product_previous`  | `Magento_Catalog` |
| `stf`  | `Magento_SendFriend` |
| `X-Magento-Vary` (note: original document has typo)  | `Magento_PageCache` |
| `amz_auth_err`  | Amazon Pay CBE |
| `amz_auth_logout`  | Amazon Pay CBE |
| `form_key`  | Multiple modules. Any module that has forms should use it. |
| `mage-cache-sessid` | `Magento_Customer`, `Magento_Persistent` |
| `mage-cache-storage`  | `Magento_Customer`, `Magento_Persistent`, `Magento_NegotiableQuote` |
| `mage-cache-storage-section-invalidation`  | `Magento_Customer` |
| `persistent_shopping_cart`  | `Magento_Persistent` |
| `private_content_version`  | `Magento_PageCache`, `Magento_Customer` |
| `section_data_ids`  | `Magento_Customer` |
| `store`  | `Magento_Store` |
| `mage-banners-cache-storage` | `Magento_Banner` (Magento Commerce). Description: Local storage for Banner functionality. |
