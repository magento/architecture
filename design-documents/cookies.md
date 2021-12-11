# Cookies in Magento

Original document: https://docs.magento.com/m2/ce/user_guide/stores/cookie-reference.html

The goal of this document is to align cookies existing in Magento to features relying on them, so a customer may choose to disable a feature and not populate a specific cookie.


| Cookie  | Is Secure | HTTP Only | Expiration Policy | Module | Related configuration |
| ------- | --------- | --------- | ----------------- | -------- | ------------- |
| `guest-view` | No | Yes | Session | `Magento_Sales` | Guest orders view. Used in "Orders and Returns" widget. |
| `login_redirect` | No | No | Session | `Magento_Customer`, `Magento_Checkout`  | Used in minicart for logged in customers if "Shopping Cart Sidebar" > "Display Shopping Cart Sidebar" is set to "Yes" |
| `mage-messages` | No | No | Duration: 1 year. Cleared on frontend when the message is displayed to the user. | `Magento_Theme` | No option to disable |
| `mage-translation-storage` (local storage) | No | No | Per local storage rules | `Magento_Translation` | Used when "Translation Strategy" configured as "Dictionary (Translation on Storefront side)". |
| `mage-translation-file-version` (local storage) | No | No | Per local storage rules | `Magento_Translation`. | Tracks version of translations in local storage. Used when "Translation Strategy" configured as "Dictionary (Translation on Storefront side)". |
| `product_data_storage` (local storage) | No | No | Per local storage rules | `Magento_Catalog` | |
| `recently_compared_product` (local storage) | No | No | Per local storage rules | `Magento_Catalog` | |
| `recently_compared_product_previous` (local storage) | No | No | Per local storage rules | `Magento_Catalog` | |
| `recently_viewed_product` (local storage) | No | No | Per local storage rules | `Magento_Catalog` | |
| `recently_viewed_product_previous` (local storage) | No | No | Per local storage rules | `Magento_Catalog` | |
| `stf` | Yes | Yes | Session | `Magento_SendFriend` | |
| `X-Magento-Vary` (note: original document has typo) | Yes | Yes | Based on PHP setting `session.cookie_lifetime` | `Magento_PageCache` | |
| `amz_auth_err` | No | No | 1 year | Amazon Pay CBE | Used if "Enable Login with Amazon" is enabled. |
| `amz_auth_logout` | No | No | 86400s (24h) | Amazon Pay CBE | Used if "Enable Login with Amazon" is enabled. |
| `form_key` | No | No | PHP: based on PHP setting `session.cookie_lifetime`. JS: session | Page Cache | |
| `mage-cache-sessid` | No | No | Session | `Magento_Customer`, `Magento_Persistent` | |
| `mage-cache-storage` (local storage) | No | No | Per local storage rules | `Magento_Customer`, `Magento_Persistent`, `Magento_NegotiableQuote` | |
| `mage-cache-storage-section-invalidation` (local storage) | No | No | Per local storage rules | `Magento_Customer` | |
| `section_data_ids` | No | No | Session | `Magento_Customer` | |
| `persistent_shopping_cart` | Yes | Yes | Based on Admin configuration in `Persistent Shopping Cart > General Options > Persistence Lifetime (seconds)` | `Magento_Persistent` | |
| `private_content_version` | Yes (based on the request)/No | No | PHP: 1 year/315360000s (10 years); JS: 1 day; JS local storage: per local storage rules (forever) | `Magento_PageCache`, `Magento_Customer` | It is set in multiple places: in PHP, in JS as a cookie, and in JS to local storate |
| `store` | No | Yes | 1 year | `Magento_Store` | |
| `mage-banners-cache-storage` (local storage) | No | No | Per local storage rules | `Magento_Banner` (Magento Commerce). Description: Local storage for Banner functionality. | |

HTTP Only = "Yes (based on the request)" means that the cookie is secure if set during HTTPS request, and unsecure if set during HTTP request.
