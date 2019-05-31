# Magento Coding Standard Versioning
The purpose of this document is the defining versioning strategy for [Magento Coding Standard](https://github.com/magento/magento-coding-standard).

## Release Line
One release line will be supported based on incremental approach despite introduced changes (new rules, bug fixes, etc).
General overview scheme:

```
3 - 4 - 5 - ... - 42 - ... - N

```

Single number sequence-based version identifier will be incremented every release.

Once new release happens it automatically means that all users (core developers, community contributors, system integrators and extension developers) need to follow the latest version.

**Only the latest version** will be supported.

## Dependency
Since we want all users (core developers, community contributors, system integrators and extension developers) to follow the latest version of Coding Standard, the dependency should be `"*"`.
```
    "require-dev": {
        ...
        "magento/magento-coding-standard": "*",
        ...
    }
```
## Release cadence
1.	Critical fixes (error during execution) deployed as ready in isolation.
2.	Monthly releases that include:
    - Bug fix to existing rule that prevents false-positive findings.
    - Implementing unit tests for existing rules.
    - Removing PHP CodeSniffer OOB rule from Magento Coding Standard `ruleset.xml` file.
    - Removing Magento rule from Magento Coding Standard (sniff code + `ruleset.xml`).
    - Adding new `exclude-pattern` to the rule.
    - Decreasing the `severity`.
    - Changing `type` from `error` to `warning`.
    - Adding new OOB rule to Magento Coding Standard `ruleset.xml` file (< severity 10).
    - Implementing new Magento rule and adding it to `ruleset.xml` file (< severity 10).
    - Changing the behaviour of existing Magento rule (extending functionality) (< severity 10).
    - Adding new `include-pattern` to the rule (< severity 10).
    - Increasing the `severity` (< severity 10).
3.	Quarterly Magento patch release bundled that include:
    - Adding new OOB rule to Magento Coding Standard `ruleset.xml` file (>= severity 10).
    - Implementing new Magento rule and adding it to `ruleset.xml` file (>= severity 10).
    - Changing the behaviour of existing Magento rule (extending functionality) (>= severity 10).
    - Adding new `include-pattern` to the rule (>= severity 10).
    - Changing `type` from `warning` to `error`.
    - Increasing the `severity` to 10.
    - Changing platform requirements.
    - Changing namespace.

Therefore, developers will have an opportunity to be tested against all checks once they are merged and released or on demand by obtaining the latest version of the project from GitHub; however, after patch or minor releases of Magento2 some checks may change in priority or start failing extension submissions.

## Side Effects
- No `patch` or `minor` dependency will be possible, so any change may lead to build (code check) failure.
- If released Magento package will contain `"*"` dependency any update may lead to build (code check) failure.
- Developers will need to track changes in the project either by paying attention to notifications or by following the project in GitHub, in order to make their "in development" code adherent to the upcoming coding-standard release. This will be especially important for patch releases, and critical for minor releases.

## Marketplace Notifications
- A week in advance before each release, a notification containing the upcoming changes will be sent. Additional bugfixes may be included into the release after this date, but all possible effort will be applied to not introducing new checks after the notification had been sent. The notification will inform of the entire scope of release changes: bug fixes, severity/scoring changes, and new checks.
- Developers on Marketplace will be notified prior EQP will update the version for PHPCodeSniffer check.
- Developers on Marketplace will see in the report what version was used during the check.
- Extension may be rejected in case it does not follow the latest Coding Standard version. Conditions for failure may include failing top priority (severity 10) checks and accumulating an unacceptable amount of warnings (coming soon).
