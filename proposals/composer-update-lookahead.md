
# Add Lookahead for Pre-`composer update` Changes
## Context
With Magento 2.3, we needed to [add additional steps to the upgrade process](https://devdocs.magento.com/guides/v2.3/comp-mgr/cli/cli-upgrade.html#require-the-additional-packages-using-composer) due to incompatibilities with dependencies in the root `composer.json` file.  Without the fixed requirements, the `composer update` command fails to install the new version of Magento because it has conflicts with the root requirements, and 2.3 code can't fix those requirements before it is installed because it would need to be installed in order to run the code.

In the future, we are likely to encounter this issue again, especially during upgrades where we drop support for old PHP versions (PHP 7.1 will soon stop receiving security patches, for example).  To avoid needing to add more manual steps for each version, we should add hooks to the upgrade process so any installations that start at 2.3 or higher check the target version for changes that need to be made before it is installed.

### Composer Functionality
[Composer plugins](https://getcomposer.org/doc/articles/plugins.md) can touch command functionality and triggered events, allows us to extract the target version from the composer requirements and check if there are any differences between the existing installation and the root `composer.json` file for the new version.  This would allow us to have users run the normal upgrade process (`composer require <new_version>; composer update`) and have the update command execute any additional steps that need to be done before it installs the new version.

For pre-2.3 installations, this plugin would need to be installed before running the upgrade, but that is as simple as adding `composer require magento/root-update-plugin` as a step to the upgrade process.  For new 2.3+ installations we would have the plugin enabled in the default composer file already.

The plugin, by necessity, would not depend on any core Magento functionality other than the root package name, so it should be compatible with all Magento versions barring very unusual circumstances.

## Requirements
 - Dry-run mode to preview changes that will be made (and also output changes that were made in non-dry-run mode)
 - Any changes to be made come from the same composer repository as the target Magento version
 - The plugin should only make changes to the specific `composer.json` entries that have changed in the target version (adding/modifying/removing specific entries, never whole sections)
 - The update fails without changing the file if the values in the existing `composer.json` file that would change do not match the expected values for the starting version and do not fit the requirements of the target version (avoid clobbering any user-made changes to those values)
 - Ability to skip the root updates without disabling any other plugins that might be installed

## Implementation Strategy
There are a few hiccups in the actual implementation.
- The update needs to be able to handle starting at any previous Magento version, each of which could have different values that need to be added, changed, or removed
- Users can and often will have made changes to their root `composer.json` to enable 3rd party add-ons, and the update should not clobber these changes.
- There could potentially be non-Composer updates that the user may want to make, such as `magento/updater`, which exists in the top-level Magento directory but is not its own composer package.
  - This does not necessarily need to be a requirement for the initial plugin functionality, but it could be an option for a future release or if it becomes necessary for a new Magento version.

To handle these issues, the update hook should inspect three things: the current root `composer.json` file, the project metapackage that would be used for a fresh installation of the starting version, and the project metapackage for a fresh installation of the target version.
- The starting version would need to be extracted somewhere other than `composer.json` (possibly from the `composer.lock` file), as by the time `composer update` is called it has already been removed from the `require` section in and the value under the `version` tag may not reflect the installed Magento version.  
- The project metapackage is different from the package that appears in the `require` section (`magento/project-community-edition` vs. `magento/product-community-edition`), so we would need to translate the referenced product package into the project that would have been used with `composer create-project`.
- Once the name and version of the starting and ending metapackages are known, they can be fetched by using the same code as composer uses internally for `composer create-project` commands (just the `composer.json` file, not the full installation).

With the expected starting state and target state both known, calculate the deltas between the current root directory and the clean starting metapackage as well as the deltas between the starting metapackage and the target metapackage.  Those sets of deltas can then be compared to check if they overlap (which would mean the user changed something on their own that the upgrade would also try to change).  If there are user changes to relevant values, the current state can be checked against the requirements of the target version for validity.  

For delta values that do not conflict with user changes, the deltas for the start -> end metapackages should be able to be applied to the existing installation.  For values that the user had made their own changes to, if the existing values are valid for the target version's requirements, leave them be and do not fail the update.  If the existing values fail validation, the update should either fail or be attempted without overriding user changes.  See **Options to Handle Conflicts with User Changes** below.

This approach should avoid the need for individually versioned upgrade scripts such as the currently available one for the 2.3.0 alpha.

### Options to Handle Conflicts with User Changes
Finding conflicting deltas with user-made changes does not necessarily mean the automated upgrade needs to be wholly rejected.  Conflicts should be able to be resolved one way or the other without forcing the user to make all of the changes themselves.

To solve this problem, here are some potential options:
 - Adding an "interactive mode" the user can use on the update command to prompt to accept or reject individual changes that will be made
	 - The non-interactive mode needs to exist to enable automation and compatibility with the Web Setup Wizard, and should default to attempt the update without overriding any user changes but also allow options for "Override all user values with Magento changes" and "Fail on any conflict"
 - Have the ability to tell the plugin to how to resolve specific conflicts on a rerun after they are found, either through CLI parameters (awkward to run manually, especially if there are a large number of conflicts), a 'resolutions' file with a clear format that the plugin checks for, or some other mechanism (suggestions welcome)
 - Tell the user the values the Magento deltas say should be there, have them manually apply the ones they want, then have them rerun the update with "Accept all user changes" set
 