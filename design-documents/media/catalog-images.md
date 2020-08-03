# Catalog Images

The document provides the vision for the desired state of assets management in Magento.
It covers currently existing managed Magento environments, or the ones envisioned in the future:

* Magento Cloud with Fastly CDN
* Magento Cloud + AEM Assets as DAM provider
   * Including Dynamic Media with Akamai CDN
   
Disclaimer: Other types of setup (e.g., on-prem + different CDN/DAM) can be supported in the similar fashion.
Current document just covers the above systems in more details to not grow in size exponentially.
There is no intention to limit usage of other types of setup.

- [Terminology](#terminology)
- [Scenarios](#scenarios)
  * [Asset Management](#asset-management)
  * [Assign an image to a product](#assign-an-image-to-a-product)
  * [Display an image on product details or products list page](#display-an-image-on-product-details-or-products-list-page)
- [Asset Transformations](#asset-transformations)
  * [Magento: Image Transformations](#magento-image-transformations)
  * [Fastly: Image Transformations](#fastly-image-transformations)
  * [AEM Assets: Image Transformations](#aem-assets-image-transformations)
- [Watermarking](#watermarking)
  * [Magento: Watermarking](#magento-watermarking)
  * [Fastly: Watermarking](#fastly-watermarking)
  * [AEM Assets: Watermarking](#aem-assets-watermarking)
    + [Watermarking with Akamai](#watermarking-with-akamai)
- [Placeholders](#placeholders)
  * [Magento: Placeholders](#magento-placeholders)
  * [Fastly: Placeholders](#fastly-placeholders)
  * [AEM Assets: Placeholders](#aem-assets-placeholders)
- [Risks](#risks)
  * [Sync full image URL from Magento Admin to Store Front](#sync-full-image-url-from-magento-admin-to-store-front)
  * [Full offload of image transformations to DAM/CDN](#full-offload-of-image-transformations-to-dam-cdn)
  * [Storefront application provides only original URL](#storefront-application-provides-only-original-url)
- [Questions](#questions)
- [Breaking Changes](#breaking-changes)

## Terminology

* **Asset** - anything that exists in a binary format and comes with the right to use.
   * **Image**, **video** are types of assets
* [DAM (Digital Asset Management)](https://en.wikipedia.org/wiki/Digital_asset_management) - a system responsible for asset management (store, create, update, delete, organize)
* [CDN (Content delivery network)](https://en.wikipedia.org/wiki/Content_delivery_network) - a system responsible for content delivery. In scope of this document, for delivery of images and video.
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

Detailed steps of the asset flow are described below.

### Asset Management

1. Step 1: Asset is uploaded to DAM
2. Step 2: DAM may perform transformations

### Assign an image to a product

1. Step 3: Admin opens product edit page
2. Step 4: Admin selects necessary image using asset picker UI (provided as part of DAM integration) and saves the product
   * Image is linked to the product as provided by DAM
   * Image path relative to DAM base URL is stored as image path
   * Asset is assigned to the product in DAM
3. Step 5: Asset relation is synced to Storefront service in a form of full URL to the asset, as part of product data
   * URL to the **original** image is synced
   * Each asset may also include specialized type: `thumbnail`, `small`, etc. The type is not related to the image size or quality and only helps the client understand where the image is supposed to be displayed
   * The asset itself is not synced to the Storefront and is stored in its original location (in the system responsible for its management: either external DAM or Magento Back Admin)

### Display an image on product details or products list page

1. Step 6: User opens PDP (product details page)
2. Step 7: PWA application loads and requests product details from GraphQL application
3. Step 8: GraphQL application requests product details (including asset URLs) from the SF service.
   * SF service returns full image URL of the **original** image
5. Step 9: PWA application fetches asset from the CDN by the provided URL
   * PWA may include transformation parameters
6. Step 10 (optional): CDN fetches the asset from the origin, if not cached
7. Step 11 (optional): Origin (DAM) may perform necessary transformations
8. Step 12 (optional): CDN may perform necessary transformations
   
## Asset Transformations

Asset transformation is responsibility of either DAM or CDN, depending on the system setup.
Both services may provide some level of transformations.
CDN usually provides more basic transformations (resize, rotation, crop, etc), while DAM may provide more smart transformations (e.g., smart crop).
Client application (PWA) does not care which part is performing transformations, it must only follow supported URL format when including transformation parameters.

In the first phase (and until further necessary) it is assumed that client application (PWA) is responsible for knowing format of transformed image URL, and so client developer should have knowledge of which DAM/CDN it works with.
As a more smart step, backend application (via GraphQL) can provide information necessary for URL formatting.

Image transformation can be done by web server on the Magento side (e.g., Store-Front), but this looks less efficient than using a CDN.
In the first phase, this is not going to be supported.
This may cause performance issues on pages with many assets loaded (such as product listing), but it is assumed that production systems should use CDN with image transformation support.
Scenario with no CDN is assumed to be a development workflow and loading unresized images is considered less critical in this situation, especially assuming [Images Upload Configuration](./img/images-upload-config.png) allows to cap image size.

Note: watermarking is a special case of asset transformation. See next section for its coverage.

### Magento: Image Transformations

Magento supports the following transformations for images:

1. resize
2. rotate
3. set/change quality
4. set background

See `\Magento\Catalog\Model\Product\Image` for details.

### Fastly: Image Transformations

Fastly provides image transformation features with [Fastly IO](https://www.fastly.com/io):

1. Convert format
2. Rotation
3. Crop
4. Set background color
5. Set quality
6. Montage (Combine up to four images into a single displayed image.)

and others. See https://docs.fastly.com/en/guides/image-optimization-api for detailed supported parameters.

Provided features fully cover Magento capabilities.

### AEM Assets: Image Transformations

AEM Assets work in integration with Dynamic Media (DM) to deliver assets, and DM provides asset transformation capabilities.
DM uses Akamai as CDN. Does it provide additional image transformation capabilities? Are those even needed taking into account that DM provides broad range of features?

1. DefaultImage - it allows the client to specify default image. Might be useful for placeholder implementation.
2. Resize: crop, scale, size, etc
3. Rotate and flip
4. Quality
5. Background color
6. Change image format

and many others.
See [Dynamic Media Image Serving and Rendering API - Command reference](https://docs.adobe.com/content/help/en/dynamic-media-developer-resources/image-serving-api/image-serving-api/http-protocol-reference/command-reference/c-command-reference.html) for more details.

Also, [Image Presets](https://docs.adobe.com/content/help/en/experience-manager-65/assets/dynamic/managing-image-presets.html) with image adjustments can be created in advance and then requested by the client.

In addition, Akamai CDN provides ability to setup [Image Manager Policies](https://learn.akamai.com/pdf/ImageManager-CS.pdf) with the following parameters:

1. Background color
2. Crop
3. Resize
4. Rotate
5. Scale

and others.

## Watermarking

Watermarking is a special case of image transformation.
It is special in a way that the **server** decides when and which watermark to apply.
All other transformations are initiated by the client.

### Magento: Watermarking

Applied during image transformations.
See `\Magento\Catalog\Model\Product\Image` for details.

### Fastly: Watermarking

See "Overlay" in https://docs.fastly.com/en/guides/image-optimization-api

Overlay must be specified via `x-fastly-imageopto-overlay` header rather than via a URL parameter, which allows the server control it. 
To make sure UX is acceptable, the workflow should be described in more details, taking into account Magento scopes.

### AEM Assets: Watermarking

Watermarking 
- [AEM Assets Watermarking](https://docs.adobe.com/content/help/en/experience-manager-65/assets/administer/watermarking.html)

Scoping?

#### Watermarking with Akamai

Akamai CDN also provides Watermarking feature using their [Image Manager policies](https://learn.akamai.com/pdf/ImageManager-CS.pdf).
Watermark can be applied to images based on client or server-based rules.
Potentially this may help implement Magento-style scoping for watermark images.

Also, see [WATERMARKING: A CONTENT OWNER'S MARK TO PREVENT PIRACY](https://blogs.akamai.com/2019/10/watermarking-a-content-owners-mark-to-prevent-piracy.html).
Looks like, it requires integration with a watermark provider.

## Placeholders

What should happen when requested asset is not available?
Use cases:

1. The asset has been removed/renamed, and the change is not synced to Storefront service yet
2. The asset has been removed/renamed by mistake or due to server issues

While we can hope that such things should not happen, it still should not be the visitor fault and there should be a good handling of such situations.
Currently, Magento handles it by loading a placeholder image.

With new architecture, CDN or DAM should be able to provide such a placeholder.

TBD: check how this can be supported with Fastly, AEM Assets/DM, Akamai. 

### Magento: Placeholders

Magento validates whether the file is present on the disk at the moment of URL generation, and provides URL to a placeholder image if the file is absent.
This limits the system to using local storage for the assets, and makes it difficult to fetch images from external DAM systems.

### Fastly: Placeholders

### AEM Assets: Placeholders

DefaultImage - allows client to specify default image.

See https://docs.adobe.com/content/help/en/dynamic-media-developer-resources/image-serving-api/image-serving-api/http-protocol-reference/command-reference/c-command-reference.html

## Risks

This section summarizes potential issues with certain scenarios.

### Sync full image URL from Magento Admin to Store Front

**Risk**: Changing Base Media URL will require full resync of Catalog to update image URLs.
**Mitigation**: Follow this procedure for changing Base Media URL:

1. Setup infrastructure so that new URL redirects to the old URL if asset doesn't exist.
2. Wait for the sync to finish.
3. Disable/drop old URL.

**Alternative**: Sync path to the assets and Base Media URL, add logic of composing full URL to the Storefront application.

The decision of selecting full sync approach is based on:

1. It is expected that Base Media URL is a rare event.
2. Mitigation steps are straightforward and may be necessary anyways.
3. Additional logic adds complexity. In this case, potential complexity is in:
   1. Fallback for websites -> stores -> store views
   2. In the future, Magento needs to be integrated with DAM. In case of integration in mixed mode (where both Magento-managed assets and DAM-managed assets are supported), the logic of composing the URL becomes even more complex as it requires Storefront to have knowledge of assets relation to products.

Base on the above, it looks more reasonable to have simple logic on Storefront side than trying to avoid full resync.

### Full offload of image transformations to DAM/CDN

**Risks**: Systems with no CDN/DAM will get performance hit on pages with many assets (such as product listing).
**Mitigation**: Possible options:

1. Upload pre-resized thumbnail images.
2. Utilize "Images Upload Configuration" to ensure huge images are not uploaded. 

In case the feature is highly requested, it can be implemented on the level of web server.
There was PoC made for resizing imaged by Nginx.

### Storefront application provides only original URL

**Risks**: 

1. Client application may use incorrect transformed URL. 
2. Clients may need to be rewritten when switching to a different CDN/DAM. 
3. Clients may be broken when switching to a different CDN/DAM.

**Mitigation**: Align client development with infrastructure of the back office. 

**Alternative**: Provide information necessary for forming correct transformation URL by Storefront API.

Then client can rely on this information to build correct transformed URL dynamically and doesn't need to know about CDN/DAM used behind.
Example:
```
transformationParams:
  - width: w
  - height: h
  - quality: q

url = origUrl + '?' + transformationParams[width] + '=100&' + transformationParams[height] + '=100&' + transformationParams[quality] + '=80'

> https://my.dam.com/catalog/product/my/product.jpg?w=100&h=100&q=80
```

## Questions

1. Do we sync full image URL to SF or provide Base CDN URL as configuration for the store?
   1. What do wee do with secure/unsecure URLs in case of full URL?

## Breaking Changes

1. Current GraphQL returns transformed images instead of originals (❗ validate this. Might be just broken URL, as it's not clear which exact transformation GraphQL provides from the entire list). Problems with this approach: 
   1. Transformation depends on Magento theme, which is beyond GraphQL scope (GraphQL knows nothing about old Magento themes, like Luma).
   2. GraphQL clients can't perform or request necessary transformation, only predefined (by irrelevant Magento themes) transformations are provided.
