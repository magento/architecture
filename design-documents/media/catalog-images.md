# Catalog Images

## Asset Delivery - High-Level Vision

### Terminology

* **Asset** - anything that exists in a binary format and comes with the right to use.
   * **Image**, **video** are types of assets
* [DAM (Digital Asset Management)](https://en.wikipedia.org/wiki/Digital_asset_management) - a system responsible for asset management (store, create, update, delete, organize)
* [CDN (Content delivery network)](https://en.wikipedia.org/wiki/Content_delivery_network) - a system responsible for content delivery. In scope of this document, for delivery of images and video.
   * CDN may be part of DAM (if DAM provides public URLs for assets), or DAM can be integrated with CDN
* **Asset delivery** - providing publicly available URL for the asset. Usually involves CDN. This is different from asset management, which focuses on the admin side of interactions with the assets, while delivery is the cliend side of it.
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

![Asset flow](https://app.lucidchart.com/publicSegments/view/21c14319-73c6-4bb4-9d5f-e71a11a58321/image.png)

#### Asset Management

1. Asset is uploaded to DAM
2. DAM may perform transformations

#### Assign an image to a product

1. Admin opens product edit page
2. Admin uses asset picker UI (provided as part of DAM integration) to select necessary image
3. Admin clicks "Save"
  * Image is linked to the product as provided by DAM
  * Image path relative to DAM base URL is stored as image path
  * Asset is assigned to the product in DAM
4. Asset relation is synced to Storefront service

#### Display an image on product details or products list page

1. User opens PDP (product details page)
2. PWA application loads and requests product details from GraphQL application
3. GraphQL application requests product details (including asset URLs) from the SF service.
4. SF service returns full image URL of the original image
5. PWA application fetches asset from the CDN by the provided URL
   * PWA may include transformation parameters
6. CDN returns the asset
   * The asset is requested from origin if necessary. Origin may perform necessary transformations
   * CDN may perform necessary transformations
   
Asset transformation is responsibility of either DAM or CDN, depending on the system setup.
Both services may provide some level of transformations.
CDN usually provides more basic transformations (resize, rotation, crop, etc), while DAM may provide more smart transformations (e.g., smart crop).
Client application (PWA) does not care which part is performing transformations, it must only follow supported URL format when including transformation parameters.

In first stage (and until further necessary) it is assumed that client application (PWA) is responsible for knowing format of transformed image URL, and so client developer should have knowledge of which DAM/CDN it works with.

As a more smart step, backend application can provide information necessary for URL formatting.

## Synchronization from Backoffice to Store-Front

### Concepts:

1. Store-Front stores only original image URLs
2. Store-Front is not responsible for physical images. This is responsibility of DAM (which can be a specialized DAM or Magento Back office). 
2. Image URLs support image transformation by parameters (e.g., `https://some.domain.com/media/catalog/product/1/2/3.jpg?w=100&h=100`)
   1. Image transformation is performed by CDN. The URLs passed from the Backoffice to Store-Front are CDN-based.
   2. Image transformation can be done by web server on the Magento side (e.g., Store-Front), but this looks less efficient than using a CDN. Benefits of this approach are not clear. Such scenario is assumed useful for development scenarios. [Images Upload Configuration](./img/images-upload-config.png) allows to cap image size, which makes the issue less critical less critical. 
3. Image types (`small`, `thumbnail`, etc) are included in the information stored on Store-Front side.

### Questions:

1. Do we sync full image URL to SF or provide Base CDN URL as configuration for the store?
   1. Full URLs as part of product data: full reindex will be required in case CDN URL changes.
   2. Base CDN URL as a store configuration + relative asset path as part of product data: added complexity of the SF App due to additional knowledge about CDN/Media Base URLs (especially in case multiple should be supported).
   3. What do wee do with secure/unsecure URLs in case of full URL?

## Dependencies

1. To allow only original image URLs stored at the Store-Front side, new format of transformed images should be supported with transformation parameters are passed as parameters to the original URL. Current transformed URL: `https://magento.store.com/media/catalog/product/cache/1/2/3/<hash>.jpg` (where `<hash>` is generated based on transformation parameters and those become invisible). Desired transformed URL: `https://store.cdn.com/media/catalog/product/1/2/3/product-image.jpg?w=100&h=100` (transformation parameters are clearly visible).
   1. On first iteration we can just serve original image URLs by Store-Front. This would fully cover use cases where Base Media URL is a URL of CDN that supports image transformation, and client is responsible for transformed image URLs generation.

## Breaking Changes

1. Current GraphQL returns transformed images instead of originals (❗ validate this. Might be just broken URL, as it's not clear which exact transformation GraphQL provides from the entire list). Problems with this approach: 
   1. Transformation depends on Magento theme, which is beyond GraphQL scope (GraphQL knows nothing about old Magento themes, like Luma).
   2. GraphQL clients can't perform or request necessary transformation, only predefined (by irrelevant Magento themes) transformations are provided.
