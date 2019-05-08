# Magento Adobe Stock Integration

Magento Adobe Stock integration provides an ability to search and download Adobe Stock assets from the Magento Admin Panel.

## Modules

Magento Adobe Stock Integration modules are provided via `magento/adobe-stock-integration` metapackage that contains:
* `magento/module-adobe-stock-api` module responsible for the general api declaration
* `magento/module-adobe-stock` module responsible for the general api implementation
* `magento/module-adobe-stock-image-api` module responsible for the image api declaration
* `magento/module-adobe-stock-image` module responsible for the image api implementation
* `magento/module-adobe-stock-image-admin-ui` module responsible for the admin panel UI implementation
 
<p align="center">
 <img alt="Modules structure" width="600" src="https://user-images.githubusercontent.com/2028541/57133879-2be0f680-6d9c-11e9-82ec-0cd2d2cbc3a3.png">
</p>

## WebAPI

The Adobe Stock integration introduces the WebAPI endpoints for retrieving, searching, saving preview and purchasing/licensing images.

![Adobe Stock Integration WebAPI](https://user-images.githubusercontent.com/2028541/57308463-4e497b80-70de-11e9-9556-46967e346dc3.png)

## Storage

Adobe Stock integration functionality requires storage for the assets downloaded from Adobe Stock (filesystem), assets information/metadata (database), Adobe Stock user profile information and a table containing authentication request ids (to match callbacks from the Adobe IMS to an admin user)

![Database Schema](https://user-images.githubusercontent.com/2028541/57133561-13241100-6d9b-11e9-9807-b32abd1236e4.png)

## User Interface

The functionality for downloading preview and licensed Adobe Stock images is accessible in the admin panel from the Magento media gallery.

![Adobe Stock Integration UI](https://user-images.githubusercontent.com/2028541/57383032-fb3afb80-71a5-11e9-8266-e858e9928615.png)

The implementation considers the introduction of a new kind of listing UI component visually adjusted for displaying images and their attributes and a new column UI component for the same purpose.

The newly introduced UI components should be delivered to the `magento/module-ui`

## External Dependency

The Adobe Stock Integration is relying on `adobe/stock-api-libphp` package for communication with the Adobe Stock API.

## Adobe Stock Integration Wiki

More details and further information is available at [Adobe Stock Integration Project Wiki](https://github.com/magento/adobe-stock-integration/wiki/)
