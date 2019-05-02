# Magento Coding Standard Versioning
The purpose of this document is the defining versioning strategy for [Magento Coding Standard](https://github.com/magento/magento-coding-standard).

## Release Line
One release line will be supported based on incremental approach despite introduced changes (new rules, bug fixes, etc).
General overview scheme:

```
3 - 4 - 5 - ... - 42 - ... - N

```

Single number sequence-based version identifier will be incremented every release.

Once new release happens it automatically means that all users core developers, community contributors, system integrators and extension developers need to follow the latest version.

**Only the latest version** will be supported.

## Dependency
Since we want all users core developers, community contributors, system integrators and extension developers to follow the latest version of Coding Standard, the dependency should be `"*"`.
```
    "require-dev": {
        ...
        "magento/magento-coding-standard": "*",
        ...
    }
```

## Side Effects
- No `patch` or `minor` dependency will be possible, so any change may lead to build (code check) failure.
- If released Magento package will contain `"*"` dependency any update may lead to build (code check) failure.

## Marketplace Notifications
- Developers on Marketplace will be notified prior EQP will update the version for PHPCodeSniffer check.
- Developers on Marketplace will see in the report what version was used during the check.
- Extension will be rejected in case it does not follow the latest Coding Standard version.
