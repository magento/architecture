
# Asynchronous Import as separate service
## Context

Main idea of extension is to replace current Import module with new implementation that will allow to users import objects in Magento by using Asynchronous approach.

Basic idea: 
- User upload *.csv (or any other) file format
- Extension receive file, validate it and return to user FileID
- By using this FileID user can start import with customer parameters (if applicable) 
- Module will parse file, split it on single messages and sends to Asynchronous API.
- Later on user can request back status of import and resubmit objects which were failed during the import

Proposed to split on several phases
- [Phase 1](base-extension.md)
- [Phase 2](import-ui.md)
- [Phase 3](retry-and-profiling.md)
