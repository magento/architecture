## Use the WebP image format for catalog images.

### Terms

WebP is an image format employing both lossy and lossless compression.
https://developers.google.com/speed/webp/

### Overview

Using WebP images should speed up a page load for customers.
WebP offers 25 – 35% smaller file sizes in compare with JPEG with the same quality. ([WebP Compression Study](https://developers.google.com/speed/webp/docs/webp_study))

[Supporting WebP by browsers](https://caniuse.com/#feat=webp)

### Design

On catalog image resizing for each size, Magento can create one more image in WebP format.
For prevention unexpected resource consumption it can be managed by configuration and by default be disabled.
A web server will be responsible for the decision which file send.
This decision will be made by availability the requested image with WebP extension and supporting WebP by the browser using the Accept header.

#### Component Dependencies

This functionality requires an asynchronous image resizing to avoid degradation on adding images to the gallery and resizing all images.

### Prototype or Proof of Concept

Catalog images from Magento with sample data resized into WebP format are for 40% lightweight compared with the same images resized into JPEG format.

Example of adding the resize into WebP for Magento's GD adapter.

At \Magento\Framework\Image\Adapter\Gd2 class add the next line into private static $_callbacks and add the second code snippet to the end of the save method.
```php
IMAGETYPE_WEBP => ['output' => 'imagewebp', 'create' => 'imagecreatefromwebp'],
```
```php
call_user_func_array(
    $this->_getCallback('output', IMAGETYPE_WEBP),
    [$this->_imageHandler, $fileName . '.webp', $this->quality()]
);
```

Example of a configuration for Nginx.
```nginx
map $http_accept $webp_suffix {
    default   "";
    "~*webp"  ".webp";
}
```

```nginx
location ~* \.(jpg|jpeg|png|gif)$ {
    try_files $uri$webp_suffix $uri $uri/ /get.php$is_args$args;
}
```

With such configuration when /path/to/image.jpg is requested if the browser supports WebP and /path/to/image.jpg.webp exists the last one will be sent.
