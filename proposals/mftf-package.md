# Overview

Magento Functional Testing Framework (MFTF) with 2.3.0 release is going to be a require-dev dependency in main magento composer.json. Details described in this document https://wiki.corp.magento.com/display/QMT/Mftf+Composer+Package+and+Decoupled+Structure
This leads us to open question where to keep Framework packages.

MFTF have two options:
1. repo.magento.com
2. packagist.org

# repo.magento.com
Is the Composer package repository which owned by Magento.

### Benefits
1. Magento is fully capable to control released packages.
2. No need to contact to 3rd party vendors in case of theirs downtime.
3. Consistency. All magento packages and extensions published in repo.magento.com.

### Disadvantages
1. Basic Magento Installation from GitHub will require adding credentials to composer auth.json.
2. Possible additional DevOps work needed (creating a build) to publish packages.

# packagist.org
Is the default Composer package repository which already handles several Magento packages:
https://packagist.org/users/mage2-team/

### Benefits
1. No need to update composer auth.json because packagist is the default for composer.
2. No DevOps work needed to publish package, except adding MFTF GitHub repo to packagist.org.

### Disadvantages
1. Inconsistency. Most Magento packages are not located here.

# Summary
On my opinion packagist.org is the right choice for MFTF because it is not a production component which means Magento customers will not be affected at all.
And because developer experience will not be affected by adding creds to auth.json. 
