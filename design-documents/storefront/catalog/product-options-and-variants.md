
## Domain Overview

Magento allows customizing a product before adding the product to the cart.
Customization happens through the customizable options defined by the merchant.
Options can represent customization, other products, digital goods so on.

All Magento product types represented through the options.


### Definitions

* **ProductOption** - represents a product characteristic which allows editing before adding to cart.
ProductOption has to have a label, information on how option values should be displayed,
and is this option mandatory.  
* **ProductOptionValue**s - belong to a particular option as to a group and represent selections which allowed by the option.
Option value could by display label, also one or may values could be pre-selected.  
* **ProductVariant** - represent the final selection of one or multiple option values that will characterize the customized product in a shopping cart.
Depends on the business scenario, a particular product variant could be linked with an existing product,
with a price, or an inventory record, or no be linked to any.
Even with no entities associated with the variant, it still has great value for a catalog because the presence of the variant says that such composition of options and their values described by the variant makes sense so that it can be purchased.
   
### Actual usages

The application distinguishes two approaches to manage options:

**Approach 1, "One to many"**: Multiple options selections lead a shopper to a selection of the single variation.

Example: configurable products
![](https://app.lucidchart.com/publicSegments/view/26cd0b67-13c1-44e4-8b61-cada36c67010/image.png)

**Approach 2, "One to one"**: The selection of multiple options leads to multiple product variants.

Examples: bundle product, customizable product, downloadable products.
![](https://app.lucidchart.com/publicSegments/view/fea3f950-6f2f-4e46-90e4-f8ee4b6877f7/image.png)
Both of approaches could be used together.

Example: configurable product with customizable option.


## Competitors analysis
During my research, I analyzed documentation on how options are covered by the competitors.
commercetools and Shopify attack this domain with product variants that represent all the possible options.
Despite the domain simplification by reducing the number of entities both systems constrained with the size of a product which makes it impossible to compose complex business cases that we frequently may face in the Magento ecosystem.

[Commercetools: Modeling Products](https://docs.commercetools.com/product-modeling-products)

[Shopify: Variants](https://help.shopify.com/en/manual/products/variants)

Bigcommerce has a more complex option representation then Shopify or Commercetools.
in some way, I would compare their options features with Magento 2 x functionalities.
Bigcommerce segregates options from variants and allows to associate intersection of options with a new product, and as a result, I believe, struggles from the same disease as Magento does - redundancy in the variant data.

[Bigcommerce: Product Options](https://support.bigcommerce.com/s/article/Options-SKUs-Rules)

## Goals

* Eliminate hard dependency on prices from options to make possible use promotions and B2B prices for customizable options, bundle, and downloadable products.
* Eliminate variants redundancy
    * Catalog should not have a variant per each option values combination, only if necessary.
    * Catalog should not create a product per variant if this product never used.
* Hide the variant matrix from storefront client, use it only in the context of operations.
* Align a similar option with a single data structure to manage them together.


## Modeling Options and Variants Data Objects

The great advantage of Magento 2 - modularity was not a real thing for the product options during Magento the lifetime.
Since times of Magento 1.x option prices coupled into options, inventory records ignored even options with assigned SKUs.
The only thing that had to play it together "Bundle" product was designed too complex to treat it as universal solution it had to become.

That's why, with the storefront project, we have an opportunity to resolve it.

Taking into account the Magento experience (good, bad, ugly) and solutions proposed on the market.
I could define the set of requirements that I would want to see in a new options implementation:

* Options are a predefined list of possible product customizations. This list belongs to the product exclusively; it is measurable, and limited only by the business requirements. So it always easy to return even a complete set of options for rendering PDP initially. 

* Variants are a unique intersection of one or may option values. The variant matrix does not belong to a product as property. The variant matrix is stored managed separately from products. The primary role of variants is to distinguish the possible intersections of options from impossible.
The variants matrix is never meant to be returned to the storefront as is. The variants matrix will be used for filtering options allowed at the storefront after one or many options selected.

### Variants, Prices & Inventory
Variants could link the options intersection with a product, a price, or an inventory record but any of these relations is not mandatory.
So, a variant can request data from the different domains if such data assigned.
Such a decision brings us unseen before the level of the flexibility, since there is no difference between product price and variant price, and as the consequence variant price, could be included in B2B pricing.

We do not have such behaviors either Magento 1 or 2 and as a result, the whole layer of product types such as bundle and downloadable do not support B2B prices or special prices for options.
Great gap especially taking into account that [Shopify supports advanced price management for variants](https://help.shopify.com/en/manual/sell-online/wholesale/channel/price-lists-customers#choose-a-price-list-type).

The link on a product that exists in the same domain could be done through the shared ID, which as for me makes sense since I can see a possible scenario when variation that initially had only a price will be "promoted" to a product. For example due to integration reasons, to pass additional attributes data along with variant to Google Merchant Center.

### Enhanced option values

To reach a better user experience,
the option could be extended with an image that represents it 
or info URL that provides additional information about option value.

[Magento: Swatches](https://docs.magento.com/user-guide/catalog/swatches.html)

[Magento: Downloadable products, options with samples](https://docs.magento.com/user-guide/catalog/product-create-downloadable.html)

Both types of resources could be specified as attributes of `ProductOptionValue`.

```proto
syntax = "proto3";

message Product {
    repeated ProductOption options = 100;
}

message ProductOptionValue {
    string id = 1;
    string label = 2;
    string sortOrder = 3;
    string isDefault = 4;
    string imageUrl = 5;
    string infoUrl = 6;
}

message ProductOption {
    string id = 1;
    string label = 2;
    string sortOrder = 3;
    string isRequired = 4;
    string renderType = 6;
    repeated ProductOptionValue values = 5;
}

message ProductVariant {
    repeated string optionValueId = 1;
    string id = 2;
    string productIdentifierInPricing = 500; #*
    string productIdentifierInInventory = 600; #*
}
#* to avoid unnecessary network calls the variant has to know does it have a link on another domain,  type, and proper name of a field representing this link TBD.
```

![](https://app.lucidchart.com/publicSegments/view/eed59f85-7d04-46ac-9d7e-eaa77073017e/image.png)

### Explaining data and operations through the pseudo SQL

Lets review pseudo SQL schema, which implements the picture described above.
This is not the instruction for implementation, tables intentionally do not have all the fields of data objects.
The example proposed to show the relations and operations that we have in the domain.

```sql
create table products (
    object_id char(36) not null,
    name varchar(128) not null,
    primary key (object_id)
);
```
Table `products` stores registry of products.
```sql

create table product_options (
    option_id char(36) not null,
    object_id char(36) not null,
    label varchar(64),
    primary key (option_id)
);

alter table product_options
    add foreign key fk_product_options_object_id (object_id) references products(object_id);
```
Table `product_options`  - represents registry of characteristics that allows customization,
                     for instance, for apparel items, it could be "Color" and "Size".  
```sql
create table product_option_values (
    value_id char(36) not null,
    option_id char(36) not null,
    label varchar(64) not null,
    primary key (value_id)
);

alter table product_option_values
    add foreign key fk_product_options_product_id (option_id) references product_options(option_id);
```
Table `product_option_values` - actual values that could be used to customize products, categorized by options.


```sql
create table product_variant_matrix (
    value_id char(36) not null,
    object_id char(36) not null,
    weight tinyint not null,
    primary key (value_id, object_id)
);

alter table product_variant_matrix
    add foreign key fk_product_variant_matrix_value_id (value_id) references product_option_values(value_id);
```

`product_variant_matrix` - Stores the correlation between option values and variants, by using this correlation, we can say which of option combination is real.
Field `product_variant_matrix.weight` - says how many options should match to match the whole variant.

The following script models data from the picture above.
Starting here I will use fancy values for primary keys instead of UUID to make further scripts more readable.
I assume that the human eye cannot efficiently analyze tens UUID signatures.
```sql
insert into products (object_id, name)
values ('t-shirt', 'T-Shirt');
insert into product_options (option_id, object_id, label)
values
       ('t-shirt/color', 't-shirt', 'Color'),
       ('t-shirt/size', 't-shirt', 'Size')
;
insert into product_option_values (value_id, option_id, label)
values
       ('t-shirt/color/red', 't-shirt/color', 'Red'),
       ('t-shirt/color/green', 't-shirt/color', 'Green'),
       ('t-shirt/size/l', 't-shirt/size', 'L'),
       ('t-shirt/size/m', 't-shirt/size', 'M')
;
insert into product_variant_matrix (value_id, object_id, weight)
values
       ('t-shirt/size/l', 'l-red', 2), ('t-shirt/color/red', 'l-red', 2),
       ('t-shirt/size/m', 'm-red', 2), ('t-shirt/color/red', 'm-red', 2),
       ('t-shirt/size/m', 'm-green', 2), ('t-shirt/color/green', 'm-green', 2);
```

So far, all looks pretty nice with such an approach product may return information for all available options with the single request.
```sql
mysql> select p.name, po.label as option_label, pov.label as option_value_label
    -> from products p
    -> inner join product_options po on p.object_id = po.object_id
    -> inner join product_option_values pov on po.option_id = pov.option_id
    -> where p.object_id = 't-shirt';
+---------+--------------+--------------------+
| name    | option_label | option_value_label |
+---------+--------------+--------------------+
| T-Shirt | Color        | Green              |
| T-Shirt | Size         | L                  |
| T-Shirt | Size         | M                  |
| T-Shirt | Color        | Red                |
+---------+--------------+--------------------+
4 rows in set (0.01 sec)
```

Let's assume that we have chosen one option value from the list.
Starting this point we can look into variants to analyze remaining options.
From the proposed example, we have chosen "Size": "M".

```sql
mysql> select object_id, value_id, weight
    -> from product_variant_matrix
    -> where value_id in ('t-shirt/size/m');
+-----------+----------+--------+
| object_id | value_id | weight |
+-----------+----------+--------+
| m-green   | m        |      2 |
| m-red     | m        |      2 |
+-----------+----------+--------+
2 rows in set (0.01 sec)
```

As you may see, our selection has matched two variants.
Both variants have weight two,
which means that we have to match at least two values to match the whole variant,
but we used only one,
which means that we can request the remaining options,
that correspond to our current selection.

```sql
select distinct value_id
from product_variant_matrix pvm
where pvm.object_id in ('m-green', 'm-red') and value_id not in ('t-shirt/size/m');
```

The remaining option values could be found in values assigned to the matched variants minus values that we selected at the previous step.

```sql
mysql> select p.name, po.label as option_label, pov.label as option_value_label
    -> from products p
    -> inner join product_options po on p.object_id = po.object_id
    -> inner join product_option_values pov on po.option_id = pov.option_id
    -> where p.object_id = 't-shirt' and pov.value_id in ('t-shirt/color/red', 't-shirt/color/green');
+---------+--------------+--------------------+
| name    | option_label | option_value_label |
+---------+--------------+--------------------+
| T-Shirt | Color        | Green              |
| T-Shirt | Color        | Red                |
+---------+--------------+--------------------+
2 rows in set (0.00 sec)
```

*Note: To achieve more advanced behavior, the variants could be "uneven" inside the single product. For instance, you would like to track only t-shirts XL: size separately for some reason (a different price or stock). The example above focused on covering the main case scenario. Still, the approach, overall, is meant to support extending the logic of resolving option values onto a variant under the hood.*
![](https://app.lucidchart.com/publicSegments/view/de9972a8-f630-4400-aacb-d3d9858862cf/image.png)

### Storefront API

#### Import API

* Product import API should accept options as a part of the Product message.
* Because product variant matrix not more belongs to Product message, we have to design import API, which will accept ProductVariant messages.

#### Read API

As were mentioned previously, options belong to a product, full options list can be retrieved from a product.

Variants do not expose at the storefront but help to filter options after one or several variants were selected.

Such behavior was recently [approved for configurable product](https://github.com/magento/architecture/pull/394).
And since the most complex part of designing API was done the main thing we have to do in the scope of this 
chapter - generalize the behavior to support not only configurable products.

As an input our API has to accept option values which were selected by a shopper.
With the response API has to return:
* List of options and option values that remains avaialble.
* List of images & videos that should be used on PDP.
* List of price identifiers to request actual prices from the price service.
* List of products that were exactly matched by the selected options.

## Proposal cross references

This proposal continues the idea of product options unification and aligned with previous design decisions that were made in this area.

* [Single mutation for adding products to cart](https://github.com/magento/architecture/blob/master/design-documents/graph-ql/coverage/add-items-to-cart-single-mutation.md)
* [Configurable options selection](https://github.com/magento/architecture/blob/master/design-documents/graph-ql/coverage/catalog/configurable-options-selection.md)
* [Gift Registry](https://github.com/magento/architecture/blob/master/design-documents/graph-ql/coverage/gift-registry.md)
