### Move category, product and page specific layout updates
Custom layout updates can be provided for specific categories, products and CMS pages via respective forms in the admin
panel - this is how Merchants can customize entity-specific pages to include additional design and functionality.
### The problem
Layout updates are defined using XML and certain instructions often using PHP classes. Only developers can
posses knowledge required to create a layout update yet the only way to provide these updates
is via user forms more commonly used by content managers. That's the __1st problem__ - content managers don't
need that functionality and it is better to hide it from them to avoid them from executing PHP code on the server.
 
__2nd problem__ is that we do not allow developers to work with entity-specific layout updates in a developer way.
Meaning that there is no way to use version control while editing these layout updates.

### Solution
Remove custom layout updates from admin panel, ignore custom_layout_update properties updated via web API and import
functionality. Make existing custom layout updates keep working for merchants upgrading from a previous Magento version
for backward compatibility. Read layout updates from _app/design/custom\_layout/\<category/product/page\>/layout\_update\_\<ID\>.xml_
files instead.
 
This way only developers will be able to change layouts for specific entity pages and be able to use
VCS while creating the layouts.
