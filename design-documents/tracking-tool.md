# Tech Upgrade Strategy

## Overview

Magento uses different 3rd party components and libraries like Symfony, Zend, hash generators, SDK, etc. And during Magento release we need to update components versions manually. Also, updates to the latest package version might require some code migration in Magento, which might impact release dates.
To avoid such problem, we need to have a mechanism to track latest versions of 3rd party components.

As most of dependencies specified in `composer.json`, such process can be automated.

## Requirements

 - An ability to provide the list of components for update
 - An ability to recognized dependencies in `composer.json`
 - Find the latest version according to the specific rule in multiple sources
 - A possibility to extend the packages source list (Packagist, Github, Gitlab)
 - Specific update rules like `^`, `~`, `-`, `*`, etc.
 - A notification mechanism for available updates (via slack, email, Webhooks)-

### Existing solutions

1. `composer outdated` - shows outdated dependencies specified in `composer.json`.
    
    Pros:
        
    - native solution
    - supports SemVer-compatible updates
    - free usage
        
    Cons:
    
    - works only with dependencies from `composer.json`
    - does not have a notification mechanism
    - only SemVer-compatible rule is available
        
2. [Renovate](https://renovatebot.com/) - web application for automated dependency updates.

    Pros:
    
    - supports multiple sources like Github, Gitlab
    - has automated Pull Requests
    - open source, can be self-hosted
    
    Cons:
    
    - PHP, composer configuration support in alpha stage
    - searches only dependencies from `composer.json`
    - SemVer compatible only in Pro account
    - Webhooks mechanism (in Pro account)
    
3. [Dependabot](https://dependabot.com/php/) - web application to keep dependencies up-to-date.

    Pros:
    
    - supports automated PRs
    - monitors security advisories
    - automated merge options
    - provides Dependabot API
    
    Cons:
    
    - does not have a notification mechanism
    - parses only `composer.json`
    - lack of documentation
    - 100$ per month for Unlimited account ($0 Open Source / Personal account)
    
4. [NewReleases](https://newreleases.io/) - web application to keep dependencies up-to-date.

    Pros:
    - multi-sources like Github, Packagist
    - notifications via Slack, email, Webhooks
    - custom regular expressions
    
    Cons:
    - still in beta
    - lack of documentation
    - no price policy
    - files with dependencies should be added manually

## Solution

Dependencies, specified in `composer.json`, can be parsed and, according to the dependencies list, we can find the latest version of the package.
At least, we might have a tool which can find the latest version based on provided list and criteria.

### Packages source 

The most packages are specified in `composer.json` but some of them are not specified, like database, or can be added in the future not via composer.
So we need to provide a possibility to extend a source of packages versions.

The tool should support multiple input types like CLI, configuration files, URL to `composer.json` file.

[Packagist](https://packagist.org) hosts all composer dependencies but, unfortunately, [Packagist API](https://packagist.org/apidoc) does not provide a possibility to retrieve the package latest version.
But `https://packagist.org/packages/:vendor/:package-name.json` contains all released versions which could be parsed according to the specified rule.

#### Github REST API

As the most of packages (not only from `composer.json`) are hosted on Github, [Github REST API](https://developer.github.com/v3/repos/releases/) might be used to get components releases.

Github provides different types of authentication:
 - unauthenticated requests (60 requests per hour associated by IP address)
 - authenticated requests (5000 requests per hour)
 
Authenticated requests require multiple headers:

 - `Authorization` should contain `token <token>` with permissions for public repositories
 - `User-Agent` can be username or application ID
 
Github provides multiple REST API entrypoints to get releases:

 - `/repos/:ownwer/:repo/releases` - get a list of all published releases
 - `/repos/:owner/:repo/latest` - get the latest published release
 - `/repos/:owner/:repo/releases/:id` - get release details
 - `/repos/:owner/:repo/tags` - get all repository tags
 
The `/repos/:owner/:repo/latest` allows getting all needed details but the problem that not all, event popular, organizations publish their releases (as example, https://github.com/symfony/console/releases, https://github.com/php/php-src/releases) which are available on Github but not via REST API.

The `/repos/:owner/:repo/tags` allows retrieving all tags (in the most cases, tags and releases the same) but the tags will be returned unsorted and some cases a repository contains not only versions tags.

WEB crawlers can be used as a possible solution for repositories which do not publish releases as Github shows sorted by date releases based on tags list. The tool can try using Github REST API, if result is an empty array, it will use WEB crawler to parse the releases page.

### Custom Mapping

A lot of organizations include not only a package version but also some prefixes, like `v`, `php`, etc.
(https://github.com/zendframework/zend-code/releases, https://github.com/php/php-src/releases), so the tool should provide a possibility to specify a mapping format, like:

 - `vx.x.x`
 - `php-x.x.x`
 - `zend-code x.x.x`
 
 because a `composer.json` contains only digit version.
 
### SemVer rules

Magento depends on different SemVer package versions like patch or minor and the tool should have a possibility to find packages only for a specific type of release like major, minor, patch, range, in other words all Composer Version Constraints and Ranges should be supported.

### Summary

Based on the tool suggestions we can automatically update `composer.json` dependencies and run all CI to check if Magento code is compatible with the component version.
And we are going to start investigation on `Dependabot` usage.

### Related Links

 - [Composer Versions](https://getcomposer.org/doc/articles/versions.md)
 - [Github REST API](https://developer.github.com/v3/)
 - [Magento Tech Stack](https://devdocs.magento.com/guides/v2.3/architecture/tech-stack.html)
 - [Github Security Alerts](https://help.github.com/en/articles/about-security-alerts-for-vulnerable-dependencies)