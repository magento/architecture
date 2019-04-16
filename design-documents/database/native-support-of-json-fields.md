## Native support of JSON fields

Since version 5.7 MySQL supports JSON data type which introduces plenty of useful features that can be adopted for modern feature development.
https://dev.mysql.com/doc/refman/5.7/en/json.html

For instance, the introduction of such new type allows us to compactly store complex entities by using a single document-field.
As a bare minimum advantage, all existing JSON structures that we are storing in the database could be indexed, and properly validated or compressed.

See, brief explanation how to index JSON fields with MySQL https://mysqlserverteam.com/indexing-json-documents-via-virtual-columns/.

Currently, Magento 2.3 supports MySQL 5.6-5.7 but if a fact it can be run with MySQL 8 as well. 
https://devdocs.magento.com/guides/v2.3/install-gde/system-requirements-tech.html#database
So we have a real possibility to drop MySQL 5.6 support since 2.4.
And introduce corresponding changes to start to use JSON data type in the declarative schema.



