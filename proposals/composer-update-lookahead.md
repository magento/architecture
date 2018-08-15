# Add Lookahead for Pre-`composer update` Changes
> _Related JIRA ticket: [MAGETWO-94153](https://jira.corp.magento.com/browse/MAGETWO-94153)_
## Context
With Magento 2.3, we needed to [add additional steps to the upgrade process](https://devdocs.magento.com/guides/v2.3/comp-mgr/cli/cli-upgrade.html) due to incompatibilities with dependencies in the root `composer.json` file.  Without the fixed requirements, the `composer update` command fails to install the new version of Magento because it has conflicts with the root requirements, and 2.3 code can't fix those requirements before it is installed because it would need to be installed in order to run the code.

In the future, we are likely to encounter this issue again, especially during upgrades where we drop support for old PHP versions (PHP 7.1 will soon stop receiving security patches, for example).  To avoid needing to add more manual steps for each version, we should add hooks to the upgrade process so any installations that start at 2.3 or higher check the target version for changes that need to be made before it is installed.

### Composer Functionality
Composer allows [scripts to be attached to specific composer commands](https://getcomposer.org/doc/articles/scripts.md).  Using the `pre-update-cmd` event would allow us to add a script hook that extracts the target version from the composer requirements and runs any necessary steps for the version before the update is executed.  This would allow us to have users run the normal upgrade process (`composer require <new_version>; composer update`) and have the update command execute any additional steps that need to be done before it installs the new version.

For pre-2.3 installations, this hook would need to be added to the root `composer.json` as well, but that can be added to the 2.3 upgrade process since we already have necessary manual steps there.

**Note:** Composer documentation states that only root-level scripts are executed by command hooks, but there might be examples of libraries that get around this.  If so, we may be able to have this functionality exist as its own composer package that can be required instead of needing to have it added through the 2.3 steps.  That would also allow us to remove the upgrade script/manual steps from the 2.3 documentation altogether and have the new update hook do the work.  **Investigation needed.**

## Requirements
 - Back up `composer.json` and anything else that might be relevant prior to running any scripts
 - Any changes to be made come from a trusted/controlled location
 - Upgrades should be as minimal as possible, ideally only making changes to the `composer.json` file (adding/modifying/removing specific entries, never whole sections)
 - Dry-run mode to preview changes that will be made (and also output changes that were made in non-dry-run mode)
 - Avoid clobbering any non-Magento changes that may have been made to the same values
 - Ability to skip the lookahead script without disabling any other scripts that might be installed

## Implementation Strategy
There are a few hiccups in the actual implementation.
- The upgrade needs to be able to handle starting at any previous Magento version, each of which could have different values that need to be added, changed, or removed.
- There are potentially non-Composer updates that need to be made, such as `magento/updater`, which exists in the top-level Magento directory but is not its own composer package.
- Users can and often will have made changes to their root `composer.json` to include 3rd party addons, and the upgrade should not clobber these changes.

To handle these issues, the upgrade hook should inspect three things: the current root directory, the project metapackage that would be used for a fresh installation of the starting version, and the project metapackage for a fresh installation of the target version.
- The starting version would need to be extracted somewhere other than `composer.json` (possibly from the `composer.lock` file), as by the time `composer update` is called it has already been removed from the `require` section in and the value under the `version` tag may not reflect the installed Magento version.  
- The project metapackage is different from the package that appears in the `require` section (`magento/project-community-edition` vs. `magento/product-community-edition`), so we would need to translate the referenced product package into the project that would have been used with `composer create-project`.
- Once the name and version of the starting and ending metapackages are known, they can be fetched by using `composer create-project <project> --no-install`

With these states known, calculate the deltas between the current root directory and a the clean starting metapackage as well as the deltas between the starting metapackage and the target metapackage.  Those sets of deltas can then be compared to check if they overlap (which would mean the user changed something on their own that the upgrade would also try to change), and if nothing overlaps then the deltas for the start -> end metapackages should be able to be applied to the existing installation.
- For deltas outside the `composer.json` file, any changes in a file or subdirectory should be treated as a change for the entire file or directory.  The `composer.json` file is the only file where we can be reasonably confident in executing deltas individually because it has a known format and function; something like `magento/updater` can contain source code or similar where the meaning of the file and directory structure is unknown.
- Anything that would change when applying the deltas needs to be backed up first.

This approach should avoid the need for individually versioned upgrade scripts the necessary one in 2.3 and should also let us update anything else that lives in the project metapackage outside of `composer.json` (such as `magento/updater`).

### Options to Handle Conflicts with User Changes
Finding conflicting deltas with user-made changes does not necessarily mean the automated upgrade needs to be wholly rejected.  Conflicts should be able to be resolved one way or the other without forcing the user to make all of the changes themselves.

To solve this problem, here are some potential options:
 - Adding an "interactive mode" the user can use on the update command to prompt to accept or reject individual changes that will be made
	 - The non-interactive mode needs to exist to enable automation and compatibility with the Web Setup Wizard, and should default to fail on any conflict but also allow options for "Accept all Magento changes" and "Accept all User changes"
 - Have the ability to tell the script to how to resolve specific conflicts on a rerun after they are found, either through CLI parameters (awkward to run manually, especially if there are a large number of conflicts), a 'resolutions' file with a clear format that the script checks for, or some other mechanism (suggestions welcome)
 - Tell the user the values the Magento deltas say should be there, have them manually apply the ones they want, then have them rerun the update with "Accept all Magento changes" set
 