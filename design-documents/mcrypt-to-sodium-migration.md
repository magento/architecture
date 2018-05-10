## 1. Goals and Requirements

1. Target Magento version is 2.3
   1. Possibly, 2.3.x patch version. The implementation should be fully backward compatible
2. Use Sodium library for encryption, as this is the latest encryption library supported natively by the latest PHP version (PHP 7.2)
3. Ensure encryption is possible on PHP 7.1, which is also supported by Magneto 2.3
4. Data is migrated to the new algorithm if necessary
   1. On-the-fly migration (data is re-encrypted when being read/written during application run) is acceptable
   2. Upgrade time should not increase significantly on large stores
 
## 2. Strategy

### PHP 7.1 and 7.2 Support

Magento 2.3 supports both PHP 7.1 and 7.2. This leads to necessity to have a solution for both versions of PHP. In the same time,
  1. Php 7.1 ships with mcrypt but doesn’t include sodium.
  2. Php 7.2 ships with sodium but doesn’t include mcrypt. 

To solve the problem, we can use polyfill library [paragonie/sodium_compat](https://github.com/paragonie/sodium_compat), which provides Sodium support to PHP installations that don't have Sodium support. It uses the PHP extension if it exists, and it's more performant in this case.

As we still need to decrypt old data encrypted with mcrypt, and Sodium doesn't support same algorithms, another polyfill library [phpseclib/mcrypt_compat](https://github.com/phpseclib/mcrypt_compat) can be used to decrypt data on PHP 7.2.

### Implementation

Include both `phpseclib/mcrypt_compat` and `paragonie/sodium_compat` as Composer dependencies.

Create adapters for Mcrypt and Sodium:

![Encryption Adapter](/design-documents/mcrypt-to-sodium-migration/encryption-adapter.png)

`Mcrypt` implementation uses `phpseclib/mcrypt_compat`.
* Old `\Magento\Framework\Encryption\Crypt` class is deprecated, and reuses the new implementation for avoiding code duplication.
`Sodium` implementation uses `paragonie/sodium_compat`.
* Use `crypto_aead_xchacha20poly1305_ietf*` methods for encryption/decryption. See [recommendations](https://paragonie.com/blog/2017/06/libsodium-quick-reference-quick-comparison-similar-functions-and-which-one-use).

`\Magento\Framework\Encryption\Encryptor` is a public API (`@api` annotation should be added) for encryption, which uses `EncryptionAdapterInterface` under the hood.

Please, see [Implementation](https://github.com/magento-engcom/php-7.2-support/pull/135) for details.

## 3. Data migration

* Limited or expected-to-be small amount of data to be converted during upgrade process
* Large amount of data to be migrated on the fly: the data is re-encrypted when read and stored again during application work. Currently used encryption algorithms are secure enough to allow the data stay.
   * Additionally, a Magento CLI command can be implemented that converts the data after the application is upgraded. This should not cause issues as both old and new data is supported by the application.

## 4. What does this mean for extension developers

* Extension developers should use `\Magento\Framework\Encryption\Encryptor` for encryption.
* They may also implement a DB patch to re-encrypt the data, if amount of data is not expected to be large.

## 5. Resources

* [Epic](https://github.com/magento-engcom/php-7.2-support/issues/127)
* [Initial Design Document](https://github.com/magento-engcom/php-7.2-support/wiki/HLD---Removing-mcrypt-and-adding-libsodium)
* [Discussion](https://github.com/magento-engcom/php-7.2-support/wiki/Discussion:-Encryption-with-Libsodium)
* [Implementation](https://github.com/magento-engcom/php-7.2-support/pull/135)
