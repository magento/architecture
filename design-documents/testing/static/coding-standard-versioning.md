# Magento Coding Standard Versioning

The purpose of this document is the defining versioning strategy for [Magento Coding Standard](https://github.com/magento/magento-coding-standard).

## Summary

Version number should be increased with taking [Semantic Versioning](https://semver.org/) into account.
Given a version number `{major}`.`{minor}`.`{patch}`, following rules MUST be applied:

- `{patch}` version MUST be incremented when backward compatible changes (bug fixes) were made.

- `{minor}` version MUST be incremented when backward compatible changes (new features) were added.

- `{major}` version MUST be incremented when backward incompatible changes were made.

### Terminology

**PHP CodeSniffer OOB rule** - rule that goes out-of-box with PHP CodeSniffer, the part of its package, the part of specific Coding Standard (PSR2, Squiz, etc).

**Magento rule** - Magento specific rule, the part of Magento Coding Standard, located under [Magento2/Sniffs/](https://github.com/magento/magento-coding-standard/tree/develop/Magento2/Sniffs) directory.

### Versioning use cases

Following use cases are representing code change examples relatively to the version part that needs to be changed.

### `{patch}` Release
- Bug fix to existing rule that prevents false-positive findings.
- Implementing unit tests for existing rules.
- Adding new `exclude-pattern` or `include-pattern` to the rule.

### `{minor}` Release
- Removing PHP CodeSniffer OOB rule from Magento Coding Standard `ruleset.xml` file.

### `{major}` Release
- Removing Magento rule from Magento Coding Standard (sniff code + `ruleset.xml`).
- Adding new OOB rule to Magento Coding Standard `ruleset.xml` file.
- Implementing new Magento rule and adding it to `ruleset.xml` file.
- Changing the behaviour of existing Magento rule (extending functionality).
- Changing the `severity` and the `type` of the rule.
- Changing platform requirements.
- Changing namespace.
