##Make product SKU a unique field in a product table.
For now, responsibility for the uniqueness of SKU attribute is part of application logic.
As a result, this logic may be inconsistent per product creation scenarios.
For instance, import may have another set of validations rules than product creation through the admin panel.

To ensure uniqueness of product SKU in the Magento database, we should add a unique constraint for the SKU field in the product table.

For backward compatibility, we can enforce SKU uniqueness through the database patch. Product with a higher ID will keep an SKU. All duplicates will get a unique suffix. For transparency, the merchant will get a report of duplicates that were resolved by the patch.

Another option is enforcing the uniqueness check for fresh installations only. With this case, existing merchants will be responsible to resolve SKU collisions manually prior to the upgrade. 
