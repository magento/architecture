# Domain Whitelist for Configurable 3rd Party Redirects

With the introduction of PWA as well as other solutions, there are use cases where Magento is treated as a headless server, serving a frontend client.
This headless Magento server can serve multiple frontend applications, potentially on different domains.
Currently there is no defined way that Magento knows which client domains it serves and are safe to redirect the user.
This presents a problem for some PayPal payment solutions when trying to implement them through GraphQl.

## Issue
Several Paypal payment methods require some URL to redirect to once the payment has completed (e.g. [PayFlow](https://developer.paypal.com/docs/classic/payflow/integration-guide/#introducing-the-gateway-checkout-solutions)).
Currently Magento uses the current host and some predefined controller that handles the redirect from PayPal.

For the GraphQl implementation we want to support the scenario where a single "headless" Magento server will act as the API server for multiple frontend clients (possibly on different domains).
In order to support this case, we need the ability for the frontend client to specify the URLs that PayPal will redirect the frontend to after the PayPal transaction.
However, introducing the ability for the client to define the redirect URL opens a security concern.
With an open redirect it could be possible for an attacker to manipulate this URL to send the user to a malicious website after payment.

## Proposed Solution
Currently, the Magento server has no way of knowing which domains all it's frontend clients are using.
Therefore we are not able to screen the URLs provided by the client, to ensure they are safe.

We propose introducing a new configuration that will contain a list of domains that are allowed in these redirect URLs.
We can use this as a whitelist to validate redirect urls, significantly reducing the security concern.

#### Configuration
The domain whitelist configuration should live in `app/etc/env.php` for the following reasons:

- This configuration file is already in use and well-known by implementors
- env.php is a secure location that is only editable via file edit or cli command
- The configuration could be set by setup scripts using `setup:config:set` cli command

We will add a new option (`--domain-whitelist`) to the `setup:config:set` that accepts a comma separated list of domains.
 
```
bin/magento setup:config:set --domain-whitelist="foo.store.com, bar.store.com, otherstore.com"
```
Would result in an `env.php` entry:
```
...
'domain-whitelist' => [
    'foo.store.com',
    'bar.store.com',
    'otherstore.com'
]
...
```

Each domain and subdomain must be explicitly specified in the whitelist (i.e. wildcards are not supported).
Disallowing wildcards (e.g. `*.store.com`) will prevent unintentionally (or intentionally) creating too broad of a whitelist that could essentially void this security measure.

#### Redirect Validation Implementation

A new class will be introduced in the WebapiSecurity module to encapsulate the logic of validating redirect URLs against this whitelist (e.g. `\Magento\WebapiSecurity\Model\Validator\Redirect`).
It will perform similar validation as already done by `\Magento\Framework\Validator\Url` as well as compare the URL's domain against the domain whitelist in `env.php`.

`\Magento\WebapiSecurity\Model\Validator\Redirect::isValid()` will validate:
1. Valid URL format (checked by `\Magento\Framework\Validator\Url`)
2. Scheme (checked by `\Magento\Framework\Validator\Url`)
3. Host (domain checked against whitelist configuration and Magento configuration*)
    - \* To maintain compatibility, all native Magento website URLs will be considered whitelisted. These are URLs configured in `core_config_data` table (e.g. `web/secure/base_url`).

The URL route will **not** be validated/sanitized by this class. This can be handled by the specific business logic class as it will be use-case dependent.
