# Composer performance

## About

Performance of `composer create-project` increased on 2.3.0. This document describes to how optimize composer performance.

## Measurements

Note: package install time includes extract time.

Packages are not cached

With `hirak/prestissimo`
2.3.0
<pre>
5.1 min
Extract time 106.6 sec
Package install time 124.5 sec
</pre>

With out `hirak/prestissimo`
2.3.0
<pre>
9.57 min
Extract time 99.6 sec
Package install time 381.8 sec
</pre>

Packages are cached

2.2.0
<pre>
Installation 2.55 min
Dependency resolution 1 sec
</pre>

2.3.0
<pre>
Installation 6.15 min
Dependency resolution 53 sec
</pre>

With --no-dev option

2.2.0
<pre>
Installation 2.32 min
Dependency resolution 1 sec
</pre>

2.3.0
<pre>
4.2 min
Dependency resolution 48 sec
</pre>

Note
<pre>
Remove `magento/magento2-functional-testing-framework` in 2.3.0
Dependency resolution 13 sec
</pre>

Measure extract and module package installation time (with --no-dev option)
2.3.0
<pre>
3.43 min
Dependency resolution 50 sec
Extract time 101.4 sec
Package install time 113.5 sec
</pre>

2.2.0
<pre>
2.31 min
Dependency resolution 1 sec
Extract time 78.6 sec
Package install time 85.75 sec
</pre>

## Solutions

1. Use plugin that allows parallel download of the packages `hirak/prestissimo`. Allows to improve performance of `composer create-project` by ~60% **recommended**
    Author suggests to install it globally. We can document this, but because some people may miss it in documentation I recommend to add it as a dependency to a project.
    _Note: Composer 2.0 already has parallel download._

1. Resolving dependencies graph takes a lot of time ~40-50 seconds. It's due to dependencies we have in `require-dev` section. There are multiple ways to improve this
    1. Move these dependencies into separate package **recommended**
    1. Add composer.lock. We want latest versions of some packages (PHPUnit for instance), so it's not ideal and most products (Drupal, Symfony, Zend Framework) don't add `composer.lock` to the project
    
1. It takes a lot of time to unarchive packages
    1. We should not include update application and dev folder by default to minimize number of files to extract and copy **recommended**
    1. Make setup work from vendor directory **recommended**

1. As most of the time being spent on unarchiving packages it might make sense to explore unarchiving packages in parallel
    This may not work on all platforms (Windows), but can really improve composer performance time. This proven to be not easy, but worth investigating further.
