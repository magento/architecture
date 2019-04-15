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

### `{minor}` Release
- Removing PHP CodeSniffer OOB rule from Magento Coding Standard `ruleset.xml` file.
- Removing Magento rule from Magento Coding Standard (sniff code + `ruleset.xml`).
- Adding new `exclude-pattern` to the rule.
- Decreasing the `severity`.
- Changing `type` from `error` to `warning`.

In context of Coding Standard backward incompatible change occurs when rules became stricter and produce more findings.

### `{major}` Release
- Adding new OOB rule to Magento Coding Standard `ruleset.xml` file.
- Implementing new Magento rule and adding it to `ruleset.xml` file.
- Changing the behaviour of existing Magento rule (extending functionality).
- Adding new `include-pattern` to the rule.
- Increasing the `severity`.
- Changing `type` from `warning` to `error`.
- Changing platform requirements.
- Changing namespace.

## Release Line
Only one release line will be supported based on incremental approach depending on introduced changes.
General overview scheme:

```
1.0.0
  |
1.0.1
  |
1.0.2
  |
  |__1.1.0
       |
       |__2.0.0
            |
            |__2.0.1
```

No `{patch}` and `{minor}` releases will be made to the previous `{major}` (1.0.0) version if the new `{major}` (2.0.0) version already exists.
