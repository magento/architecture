# Catalog Images

## Asset Management - High-Level Vision

### Terminology

* **Asset** - anything that exists in a binary format and comes with the right to use.
   * **Image**, **video** are types of assets
* [DAM (Digital Asset Management)](https://en.wikipedia.org/wiki/Digital_asset_management) - a system responsible for asset management (store, create, update, delete, organize)
* [CDN (Content delivery network)](https://en.wikipedia.org/wiki/Content_delivery_network) - a system responsible for content delivery. In scope of this document, for delivery of images and video.
   * CDN may be part of DAM (if DAM provides public URLs for assets), or DAM can be integrated with CDN
* **Image transformation** - resizing, rotation, watermarking and other automated transformations on an original image.
   * Image transformation is responsibility of either DAM or CDN. This includes resizing, rotation, watermarking and so on.
   * Client should be able to fetch transformed images by its original URL with additional parameters
   * Transformation parameters may differ based on the CDN/DAM providing the transformation service
* **Magento back office (Magento Admin)** - in scope of this document, a system for products and categories management.
   * Links assets to products and categories
   * Provides basic functionality for images and video transformation. Long-term, should be offloaded to specialized systems (DAM, CDN)
* **Catalog Store-Front Application** - application providing product and category information suitable for store-front client scenarios.
   * Serves URLs of original assets.
   * Should not be aware of asset management functionality (no knowledge about underlying integration with external DAM systems).
   * Might have Base CDN URL. This is similar to current Base Media URL and serves the same purpose, but might exist in case both sources of assets should be supported (Magento and CDN).

## Scenarios

### Backoffice Scenarios

#### Assign an image to a product (using DAM integration)

This is a future desired scenario, provided here for better understanding of the future picture.

1. Admin opens product edit page
2. Admin uses asset picker UI (provided as part of DAM integration) to select necessary image
3. Admin clicks "Save"
  * Image is linked to the product as provided by DAM
  * Image path relative to DAM base URL is stored as image path
  * Asset is assigned to the product in DAM

#### Assign an image to a category

Similar to the product edit scenario

### Store-Front Scenarios

#### Display an image on product details or products list page

1. Client (GraphQL server) requests product details (including images) from the SF application.
2. SF application returns full image URL of the original image

## Synchronization from Backoffice to Store-Front

### Concepts:

1. Store-Fronts stores only original image URLs
2. Image URLs support image transformation by parameters (e.g., `https://some.domain.com/media/catalog/product/1/2/3.jpg?w=100&h=100`)
   1. Image transformation is performed by CDN. The URLs passed from the Backoffice to Store-Front are CDN-based.
   2. Image transformation can be done by web server on the Magento side (e.g., Store-Front), but this looks less efficient than using a CDN. Benefits of this approach are not clear. Such scenario is assumed useful for development scenarios only.
3. Image types (`small`, `thumbnail`, etc) are included in the information stored on Store-Front side.


### Questions:

1. Do we sync full image URL to SF or provide Base CDN URL as configuration for the store?
   1. Full URLs as part of product data: full reindex will be required in case CDN URL changes.
   2. Base CDN URL as a store configuration + relative asset path as part of product data: added complexity of the SF App due to additional knowledge about CDN/Media Base URLs (especially in case multiple should be supported).

## Dependencies

1. To allow only original image URLs stored at the Store-Front side, new format of transformed images should be supported with transformation parameters are passed as parameters to the original URL. Current transformed URL: `https://magento.store.com/media/catalog/product/cache/1/2/3/<hash>.jpg` (where `<hash>` is generated based on transformation parameters and those become invisible). Desired transformed URL: `https://store.cdn.com/media/catalog/product/1/2/3/product-image.jpg?w=100&h=100` (transformation parameters are clearly visible).
   1. On first iteration we can just serve original image URLs by Store-Front. This would fully cover use cases where Base Media URL is a URL of CDN that supports image transformation, and client is responsible for transformed image URLs generation.

## Breaking Changes

1. Current GraphQL returns transformed images instead of originals (❗ validate this. Might be just broken URL, as it's not clear which exact transformation GraphQL provides from the entire list). Problems with this approach: 
   1. Transformation depends on Magento theme, which is beyond GraphQL scope (GraphQL knows nothing about old Magento themes, like Luma).
   2. GraphQL clients can't perform or request necessary transformation, only predefined (by irrelevant Magento themes) transformations are provided.
