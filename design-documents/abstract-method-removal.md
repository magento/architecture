# Abstract Method removal

## Overview

Currently Magento supports two API for new payment integration creation `\Magento\Payment\Model\Method\AbstractMethod` and `\Magento\Payment\Model\Method\Adapter` (the part of [Magento Payment Provider Gateway](https://devdocs.magento.com/guides/v2.3/payments-integrations/payment-gateway/payment-gateway-intro.html)). The `AbstractMethod` was deprecated in Magento 2.1 release and `Adapter` is a recommended approach for new payment integrations.

Let's consider the reasons why the `AbstractMethod`'s usage should be eliminated:

- Allows storing credit card details in database
- The credit card storage requires additional protection for PCI compliance certification
- A new integration should override most of the parent's methods and properties to provide a new functionality
- The integration's support and customization requires to change existing code instead of adding/changes only some parts (like Payment Gateway allows adding new commands, request builders, response handlers, etc.)
- Does not support Vault storage
- An introduction of new features is complicated to provide in BC way
- Complicated to cover an integration or separate parts by integration/unit tests
- Tightly coupled with Sales Order Management
- Inheritance leads to the code duplication
- The usage of `AbstractMethod` requires to support both API `AbstractMethod` and `Adapter`
- Does not have a developer documentation

## Removing strategy

There are still a few core integrations (Offline payments and PayPal), also some extensions (~30 on Marketplace), which based on or use `AbstractMethod`. The integrations should be re-implemented based on Magento Payment Provider Gateway. The core integrations can be re-implemented during patch releases (the old integrations should be marked as deprecated).

The desired state for 2.4 Magento release:
 
- Core integrations do not use `AbstractMethod` (old integrations marked as deprecated).
- All related to `AbsractMethod` infrastructure code is marked as deprecated (which not marked yet)
- All appropriate methods/functions trigger `E_DEPRECATED` warning (https://devdocs.magento.com/guides/v2.3/contributor-guide/backward-compatible-development/#deprecation).


The desired state for 2.5 Magento release:
 - Remove all `AbstractMethod` infrastructure related code/database columns
 
## Summary
 
The 3rd party developers will have a full minor-release cycle to rework their integrations. As each payment integration has a lot of specific implementation details, own flow and data set for payment operations, there is no unified approach to provide tools for code migration from `AbstractMethod` to Payment Provider Gateway.