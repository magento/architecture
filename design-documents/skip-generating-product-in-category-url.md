### Overview

Saving a category takes too long.
One solution is tobe able to turn off automatic url rewrite generation for products on category save, so that, there is no performance issues any more when I save a category with lots of assigned products.
The frontend should still be able to validate and load proper product entities and generate same url structure as the default behavior when assigning products to multiple nested categories.

Example of default behavior of generated rows when saving cat2:

        cat1.html
        cat1/cat2.html
        prod1.html
        cat1/cat2/prod1.html
        cat1/prod1.html

Example of new behavior of generated rows:
                
        cat1.html
        cat1/cat2.html
        prod1.html
        
In this case

        cat1/cat2/prod1.html
        cat1/prod1.html
        
Will be computed when we request it and we have to check if prod1.html really belongs to cat1/cat2 or on the other case/request cat1

### Theory of Operation

We will introduce a new admin option to turn off generating category-product urls (that represents 95% of url_rewrites table resulting into hundreds of thousands of entries for a big catalog)
Instead we can compute these urls and match them with a new router or inside the existing router that queries url_rewrites table
 "Generate URL Rewrites for Products on Category Save" option is set to "Yes" - so default behavior is enforced.
 

Currently when saving a category with lots of assigned products, there is large amount of data generated and saved into rewrites tables which causes performance issues to lookup and generate those records

The proposed solution is to allow to skip generation of url rewrites for products in category when saving a category, but compute such URLs when loading a page on storefront.
This will improve saving of modified categories in the admin, and it will add more load on the frontend to determine if url is valid and load the proper product.

In scope of this story, we have and add a new configuration option which allow to turn on/off this functionality and retest URL rewrites functionality to make sure that everything works as is now, when the functionality is turned off.

### Acceptance criteria


 * There is a new global configuration option "Generate URL Rewrites for Products on Category Save" introduced in "Configuration->Catalog->Catalog->Search Engine Optimization". There are 2 options available to select "Yes" and "No". Default value = "Yes".
 * If "Generate URL Rewrites for Products on Category Save" option is set to "Yes" then current url rewrites functionality works as is now without any changes in behavior and expected result.
 * If "Generate URL Rewrites for Products on Category Save" option is set to "No" then, when saving a category, the url rewrites which consists of multiplication of category and product URLs are not generated and not saved into rewrites table. 
 * Frontend should still load product in categories urls like cat1/cat2/prod1.html but compute if prod1.html is visible and assigned to cat1/cat2

### Work Breakdown
 * Introduce a new router. The default behavior is still needed as, and both have to coexist, because we don't delete rows only on save. Alternatively we can inject a pool to match the two types of matching inside the *\Magento\UrlRewrite\Model\UrlFinderInterface*. There's an obsolete extension point inside the *\Magento\UrlRewrite\Model\Storage\AbstractStorage::doFindOneByData* based on template method. A pool is the preferred approach in this case because we already have \Magento\CatalogUrlRewrite\Model\Storage\DbStorage and \Magento\UrlRewrite\Model\Storage\DbStorage.
   Also in case of routing, we need to keep first select a matching record in the url_rewrite table, and if not matched we need to compute a product in category url.
   Most of the changes will be made in \Magento\CatalogUrlRewrite\Model\Storage\DbStorage::doFindOneByData which support both types of matching.
 * Skip generating product in category urls in case we set to No the option "Generate URL Rewrites for Products on Category Save". For this we need to inject system configuration in several places and create at least one dependency.
 * Resolve somehow the case where product in category has the same name as a nested category. This can be solved by passing to a cron this process and generate all products in category (that represent 95% of rows as before and it is a long process), and display at a later time an alert with a dismiss button in the admin a possible duplicate.
   The way this is solved now is that we actually generate all these urls, we write them in batches in the databse but we don't commit the transaction, and because of an unique index the table can alert us if there's a duplicate, and then we rollback everything displaying an error in the admin.
   This process may take up more than 1 minute in which case varnish cache will timeout and the admin will never see that message. This process takes a long time only needed in case of category save, not product.
 * Consolidation of logic between resource model and collection for url rewrites as now some logic duplication exists and we don't want to create more.
 
 
###  Known Issues and Shortcomings

Duplicate URLs: It is possible to have a case where a product and category have the same request path
                
        Product named Jeans in Cat1
        Category named Jeans in Cat1
                
Both have valid request path of cat1/jeans.html; This approach has no way to determine which we are requesting.
                
Performance Concerns:

 * Adding an extra layer to routing that could have effects on all routes even unrelated to products
 * Loading product collections: When product collection load rewrites, it takes more processing to figure out urls, this can cause sever degradation on Category page
 * Routing to an individual product would require more logic and queries to decipher a url, as opposed to a simple DB lookup

Additional concerns with the implementation:

 * Changes will trigger a MINOR change because we have to introduce config dependency in the @api class \Magento\Catalog\Model\ResourceModel\Product\Collection. This is needed to create urls in the category page for displayed products.
 * Inside \Magento\UrlRewrite\Controller\Router we do a loop inside the existing router to try if there's an entry for the url_rewrites (to keep default behavior because if we disable it, both data/behaviors might still exist, most categories until resaved will contain all the permutations) and we call this composite UrlFinderPool used in  \Magento\UrlRewrite\Controller\Router::getRewrite
