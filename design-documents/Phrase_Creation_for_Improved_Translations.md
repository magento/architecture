# Overview

Translation can be a very difficult problem.  One made even harder when software is developed within an English world-view.  
At the time of this proposal's authorship, Magento utilizes a simplistic parameterized string replacement system.  This works fine for
many use cases, but can easily fall apart when translating into languages that conjugate based on the gender of a parameter, or that
contain plural forms not found in English, such as Ukranian or Arabic.

Take for example this phrase: "There are 3 orders."

In English, we might decide to make this easier on the programmer, and only create one phrase:

```php
return __('There are %1 order(s).', $orderCount);
```

Of course, this works fine in English - but very quickly falls apart in other languages.  Japanese, for example, would likely want two
variances:

* 注文がない (There are no orders)
* 注文が%1つある (There are x orders)

Placing only one translation (注文が%1つある) ends up being very strange with 0, and provides a poor experience to the admin user.

What if instead we decided we want to offer the best possible experience for the admin user (thinking in English) and create three 
phrases:

```php
switch ($orderCount) {
    case 0:
        return __('There are no orders.');
    case 1:
        return __('There is %1 order.', $orderCount);
    case 2:
        return __('There are %1 orders.', $orderCount);
}
```

This works easily enough for both English and Japanese!  But it falls apart in Ukranian, where there is a different ordinal plural for
few vs. many. 

This added complexity introduces the need for phrases to be processed after replacement.  Such a system then allows languages to add
support for conjugation or plural variances that were not thought about during the initial string creation (up to a point).

# Solution

The solution to this is two parts.

1. Develop a system that processes a special syntax in replaced strings to support features that are either not thought of during the
   development phase, or that are difficult to develop for.
2. Adopt Rules/Best Practices that ensure this system can be properly utilized in multiple locales moving forward.

## Develop the System

Fortuitously, such a system already exists in PHP - the INTL MessageFormatter class.  It would need to be added after the translation
step in the Phrase rendering process.

Such a change is implemented at [magento/magento2#22996](https://github.com/magento/magento2/pull/22996)

## Proposed Rules/Best Practices

* Moving forward, all new phrases that include substitution should utilize MessageFormatter syntax
* When possible, the preferred gender of a variable should be provided as an argumet to the Phrase

# Disadvantages

There are a few disadvantages to such a change:

* MessageFormatter is slower to process than string replacement, since it has a processable syntax
* New Phrases and translations that make use of MessageFormatter syntax will not be compatible with versions of Magento predating 
  MessageFormatter implementation
  
# Summary

In my opinion, the added localization capability far outweighs the slight impact to performance.  I believe such a system is an obvious
choice to improve localization efforts of the platform.

# Prototype

[magento/magento2#22996](https://github.com/magento/magento2/pull/22996)
