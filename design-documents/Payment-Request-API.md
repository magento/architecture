## Overview
The Payment Request API (PR API) is a [W3C standard](https://www.w3.org/TR/payment-request/) candidate that is meant
to eliminate checkout forms. It improves user workflow during the purchase process, providing a more consistent user
experience and enabling merchants to easily leverage different payment methods.

The Payment Request API is designed to be vendor-agnostic, meaning it does not require the use of a particular payment system.
It's not a new payment method, nor does it integrate directly with payment processors; rather, it is a conduit from
the user's payment and shipping information to merchants, with the following goals:

* Let the browser act as an intermediary among merchants, users, and payment methods
* Standardize the payment communication flow as much as possible
* Seamlessly support different secure payment methods
* Work on any browser, device, or platform - mobile or otherwise
* Is an open and cross-browser standard

PR API supports different payment methods like:

* Basic Card (standardized by [W3С](https://w3c.github.io/payment-method-id/#registry)) - a payment method that provides credit,
debit and prepaid card payment information
* URL-based (like Google Pay, AliPay, Apple Pay, Samsung Pay, etc.) - anyone can develop and provide their own solution

## General Transaction Flow
In general, the payment transaction flow looks like this:
![PR API](http://res.cloudinary.com/dxaxolwzl/image/upload/v1527778787/PR_API_General_Flow.png)
1. JS components render a payment page and initialize Payment Request API.
1. The browser performs PR API initialization with specified payment details.
1. After PR API initialization, JS component calls `show()` method to show payment popup.
1. A customer chooses one of the available payment methods or fills credit card details.
1. The browser sends payment details to chosen payment application (like Google Pay, Apple Pay, etc.).
1. Payment Application sends data to a payment gateway.
1. The payment gateway returns a response with token or credit card details (depends on chosen payment method).
1. The application sends received data to the browser.
1. Now, js component can process received payment details before sending to the payment gateway.
1. A merchant website sends a transaction with payment details to the payment gateway.
1. Payment Gateway sends the payment transaction to an issuing bank for a verification.
1. The issuing bank verifies the transaction and sends back a result to the payment gateway.
1. The payment gateway sends the response to the merchant's website and it completes the order.

## Checkout Flow with PR API
Payment Request API doesn't replace standard Magento checkout flow. The main idea is to provide additional, more
consistent user experience flow which doesn't require any additional payment forms. As not all browsers support PR API,
a customer still should have a possibility to use standard checkout. In a case, if PR API is supported by a browser,
both variants should be available. Fortunately, PR API allows to detect is it supported by the browser or not:
```javascript
if(window.PaymentRequest) {
  // Use Payment Request API
} else {
  // Fallback to traditional checkout
  window.location.href = '/checkout/traditional';
}
```

## Requirements

* Requires trusted HTTPS connection even for testing

## PCI Compliance and Security

As PR API with basic-card payment method can return credit card data like, CC number, expiration date, CVV, etc. there
are few important security notes. If merchant's site is compliant with PCI SAQ A, it isn't allowed to handle raw
credit card data directly. The merchants compliant with PCI DSS or [PCI SAQ A-EP](https://www.pcisecuritystandards.org/documents/PCI-DSS-v3_2-SAQ-A_EP.pdf)
should not worry about it.

As a possible solution, the usage of [Braintree](https://developers.braintreepayments.com/guides/google-pay/client-side/javascript/v3),
[Stripe](https://stripe.com/docs/stripe-js/elements/payment-request-button) or other Payment Gateways which support
PR API allows avoiding PCI compliance certification.