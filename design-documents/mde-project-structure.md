# Overview

There is a need to have unified approach to structuring the Git-based project while developing MDEs (Magento Developed Extensions), like Page Builder and MSI.

Currently, MSI is developed in a fork of main Magento repository, while Page Builder is developed in a separate repository (like B2B).

## MDE project structure for Git-based installations

MDE project must be structured as Magento B2B:
 1. Modules must be placed under `app/code`
 1. Each module must have valid `composer.json` with no version specified. Package version must be generated and added to the `composer.json` during release publication
 1. No core modules must be committed to the MDE repository
 1. Unit and MFTF tests must be distributed between modules (since they are modular). All other tests must be put inside `dev/tests` like it is done in Magento Open Source repository (because these testing frameworks do not support modularity)

## Git-based project linking for local development

Preferred way of linking the project is using [Composer path repository](https://getcomposer.org/doc/05-repositories.md#path).

1. Check out magento 2 open source edition from Git into `<project_root>/magento2ce`
1. Check out MDE into `<project_root>/magento2ce/<mde-name>`
1. Add path composer repository declaration to `<project_root>/magento2ce/composer.json`:
    ```json
    "minimum-stability": "dev",
    "repositories": [
        {
            "type": "path",
            "url": "./<mde-name>/app/code/*/*"
        }
    ]
    ```
1. Add dependencies on all relevant MDE modules to the `require` section of `<project_root>/magento2ce/composer.json`. Use `*` as a package version
1. Run `composer update`, which will create symlinks to the requested MDE modules inside of `<project_root>/magento2ce/vendor`. 
   On Windows environment packages will be copied by composer (instead of being symlinked), that's why on Windows hosts it is better to link MDE using EE linking tool, or perform linking inside of linux VM
1. In case Magento has already been installed, the new modules must be enabled and setup upgrade executed


## Alternative options

1. Link MDE using the same script which is used for linking EE (the tool should be open sourced)
2. Manually create symlinks for each MDE modules to `<project_root>/magento2ce/app/code`

