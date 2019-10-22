# Asynchronous image resizing

## Problem

Full image resize triggered by catalog:image:resize command may take dozen of hours.

## Proposed solution
 
Proposed way to speed up such process - implement asynchronous image resizing 
with possibility to run several workers in parallel mode.

## Design

1) Implement scheduler service that will accept image filenames and schedule task to resize images via Magento Bulk Management Interface
2) Implement consumer that will process image resizing based on already implemented Image Resize service
3) Modify image resize console command to add possibility to use scheduler service instead of resize service. Implementation should preserve old behavior as default and use new behavior with new command line option
