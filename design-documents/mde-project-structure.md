# Overview

There is a need to have unified approach to structuring the Git-based project while developing MDEs (Magento Developed Extensions), like Page Builder and MSI.

Currently, MSI is developed in a fork of main Magento repository, while Page Builder is developed in a separate repository (like B2B).

## MDE project structure for Git-based installations

MDE project must be structured as Magento B2B:
 1. Modules must be placed under `<mde_root>`
 1. Each module must have valid `composer.json` with no version specified. Package version must be generated and added to the `composer.json` during release publication
 1. No core modules must be committed to the MDE repository
 1. All tests must be distributed between modules. This is true even for kinds of tests which do not support modularity yet
 1. Optionally, create composer metapackage inside of `<mde_root>` which will be used to require all MDE modules at once

## Git-based project linking for local development

Preferred way of linking the project is using [Composer path repository](https://getcomposer.org/doc/05-repositories.md#path).

1. Check out magento 2 open source edition from Git into `<project_root>/magento2ce`
1. Check out MDE into `<project_root>/<mde-name>`
1. [Install MDE extension](https://devdocs.magento.com/guides/v2.3/comp-mgr/install-extensions.html) as if it was installed from remote composer repo. Specify `*` as package version.
   **On Windows environments only** installation must be done inside of a linux VM to avoid MDE files being copied to vendor directory instead of being symlinked.

## Blockers
1. [Allow templates loading outside of the project](https://jira.corp.magento.com/browse/MAGETWO-95040)
2. [Add support for MDEs via composer path repository](https://jira.corp.magento.com/browse/MAGETWO-95041)

## Alternative options considered (not recommended)

1. Link MDE using the same [script](https://github.com/magento/magento2ee/blob/2.3-develop/dev/tools/build-ee.php) which is used for linking EE (the tool should be open sourced). The tool needs to be adjusted to support MDE repository structure without `app/code`
2. Manually create symlinks for each MDE modules to `<project_root>/magento2ce/app/code`. Copy 3rd party package dependencies from MDE modules to `<project_root>/magento2ce/composer.json`

