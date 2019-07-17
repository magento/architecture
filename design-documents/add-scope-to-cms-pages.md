
## [HLD] MC-13907: Add StoreView Scope to CMS Pages

HLD for [MC-13907](https://jira.corp.magento.com/browse/MC-13907): Allow CMS Page to Be Edited and Viewed in Admin Based On StoreView

### Terms

### Overview
CMS Pages can currently be turned on/off for store views, but the content cannot be customized. Epic MC-13907 will provide support for customizing a Page at the store view level so that users can more easily manage localized content.

**Current schema:**

![](https://wiki.corp.magento.com/download/attachments/115264240/cms_page_schema_current.png?api=v2&version=4)

The current page design ties a CMS page to one or more store views but this association does not differentiate content by store view, it only defines the availability of the page to a store view.

### Design
#### Option 1: Introduce a separate content table [Preferred]
This option would move all page attributes that can be customized to a store view to a separate table, which will include the associated `store_id.` There will be a single "master" page object that retains  `is_active` and store associations at the global level, with all other attributes customizable to the store view level:

![](https://wiki.corp.magento.com/download/attachments/115264240/cms_page_proposed.png?api=v2&version=3)

##### **Pros:**
-   Implementation matches that of Dynamic Blocks (banner tables)
-   Maintains a single `page_id` for each page, regardless of scope

##### **Cons:**
-   Larger database change will require setup scripts to migrate existing data into the new table structure
-   Resource model will require more refactoring than Option 2  

##### Scenario: New Page
When a new page is created, it will default to the All Store Views scope, creating an entry in  `cms_page_content` with `store_id=0`

##### Scenario: Editing a new scope
After a page is created, changing scope will attempt to load the content matching the current `page_id` and selected  `store_id`. If none exists, the data for  `store_id=0` will be loaded. If changes are made and saved, a new  `cms_page_content` entry will be created to save data under the selected  `store_id`.

##### Scenario: Deleting a page
Delete a page deletes all content associated with it for all store views.

##### Scenario: Page Builder needs selected store view scope
Page Builder components (and potentially other extensions) will need to be aware of the selected store view to render store-sensitive data (such as Product List). The store switcher component adds the chosen store_id to the application context and will be available as Page Builder currently expects (via StoreManager) without additional changes.

  

#### Option 2: Add store & parent ID to existing table
This option would add  `store_id` &  `parent_id` to  `cms_page` to denote which entries are "master" pages and which are scope specific:

![](https://wiki.corp.magento.com/download/attachments/115264240/cms_page_proposed_option2.png?api=v2&version=3)

##### **Pros:**
-   Fewer, simpler database changes

##### **Cons:**
-   Will still require setup scripts to populate the new columns
-   "child" pages have a different page ID than will be displayed in the URL, which could be confusing for 3rd party developers
-   any externally developed modules that list pages would have to be updated to filter out "child" pages
-   More complicated logic to load/save the desired page/store view content  

##### Scenario: New Page
When a new page is created it will default to the All Store Views scope, and the data will be saved to `cms_page` as it is now, with `parent_id` = null and `store_id` = 0.

##### Scenario: Editing a new scope
After a page is created, changing scope will attempt to load the content using `parent_id` = page_id and the selected  `store_id`. If none exists, the data for the  `page_id` and  `store_id=0` will be loaded. If changes are made and saved, a new  `cms_page` entry will be created to save data with a new  `page_id`, setting the  `parent_id` = parent  `page_id` and using the selected  `store_id`. Since `is_active` is a global attribute, its value will be copied from the parent page.

Code will need to be created to always load the `is_active` value from the parent page, OR to always propagate any change in value to all child pages so it can be loaded directly from the child page.

##### Scenario: Deleting a page
Delete a page deletes all content associated with it for all store views.

##### Scenario: Page Builder needs selected store view scope
Page Builder components (and potentially other extensions) will need to be aware of the selected store view to render store-sensitive data (such as Product List). The store switcher component adds the chosen store_id to the application context and will be available as Page Builder currently expects (via StoreManager) without additional changes.

#### Acceptance Criteria Fulfillment
1. СMS Pages have Magento Scope.
2.  Admin user with scope specific role is able to edit scope specific content
3.  CMS Page is assigned to AllStoreviews by default.
4.  Changing the drop-down value will cause the page to load the content for that specific storeview
5.  CMS Page content attributes have storeview scope except:
		- enable page - global
		- store view - global
6.  Changing the drop-down value to any value other than All Store Views will by default have the "Use Default Value" checkbox checked for all storeview specific attributes
7.  ContentHeading is removed from the CMS Page Form(only new pages affected)
8.  PageTitle content attribute value is copied to MetaTitle by default

#### Component Dependencies
- Store

#### Extension Points and Scenarios
Page Builder components (and potentially other extensions) will need to be aware of the selected store view to render store-sensitive data (such as Product List). The store switcher component adds the chosen store_id to the application context and will be available as Page Builder currently expects (via StoreManager) without additional changes.

### Prototype or Proof of Concept

### Data size and Performance Requirements
No new degradations
