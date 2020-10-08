## Dynamic Blocks

Dynamic blocks can be inserted in CMS pages and blocks using widgets. Below is the sample markup of the widget inserted on the page.

```
{{widget type="Magento\Banner\Block\Widget\Banner" display_mode="fixed" types="content" rotate="random" banner_ids="1" template="widget/block.phtml" unique_id="34f6520f51ca9b79c91d1f56b5b453adc4d9ff235a7c9f5b04d1b0557584c5bc"}}
```

In Luma dynamic blocks rendered to a JavaScript component definition that does request to a backend to load content of the dynamic block. Here is an example of a JavaScript component definition.

```
<div class="widget block block-banners" data-bind="scope: 'banner'" data-banner-id="34f6520f51ca9b79c91d1f56b5b453adc4d9ff235a7c9f5b04d1b0557584c5bc" data-types="content" data-display-mode="fixed" data-ids="1" data-rotate="random" data-store-id="1">
    <ul class="banner-items" data-bind="afterRender: registerBanner">
        <!-- ko foreach: getItems34f6520f51ca9b79c91d1f56b5b453adc4d9ff235a7c9f5b04d1b0557584c5bc() -->
        <li class="banner-item" data-bind="attr: {'data-banner-id': $data.bannerId}">
            <div class="banner-item-content" data-bind="bindHtml: $data.html"></div>
        </li>
        <!-- /ko -->
    </ul>
</div>
```

PWA will receive similar JavaScript component definition that can be parsed to extract parameters and make a [GraphQL query](./dynamic-blocks.graphqls) to load dynamic blocks.

```graphql
{
    dynamic_blocks(input: {type: SPECIFIED, locations: [CONTENT], rotation_mode: RANDOM, dynamic_block_uids: [1, 2]}) {
        items {
            uid
            name
            content
        }
        page_info {
            current_page
            page_size
            total_pages
        }
        total_count
    }
}
```
