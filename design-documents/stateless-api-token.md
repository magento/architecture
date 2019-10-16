## Introduce stateless web API tokens
&nbsp;&nbsp;&nbsp;&nbsp;Authentication tokens used to access web APIs (REST, SOAP, GraphQL) should contain enough
information to authenticate a user without having to store session data on server side.
 
This approach has following advantages:
* In case of distributed deployment instances don't have to access single storage/service to authenticate a user
* Saves storage space
* Faster to decode/decrypt than load from storage
* Don't have to implement token protection mechanism on backend to keep tokens from being stolen
 
 
### Current situation
&nbsp;&nbsp;&nbsp;&nbsp;Magento stores tokens in RDMS table named `oauth_token` which is originally meant to store long-term integration tokens
but also used for admin/customer user tokens. Token rows also contain user type, issued data and expiration data which
is retrieved when authenticating users for web API via token passed in the header. Admin and customer tokens may be
manually expired by admin users by updating `oauth_token` table data. Tokens themselves are random strings of static
length.
 
With described above circumstances in order to perform authentication Magento instances have to have access to same DB
and perform (relatively) expensive SQL queries to DB. Te more users Magento has the more tokens it has to store
until they expire and run an extra cron job to remove expired ones.
 
 
### Overview of new stateless tokens implementation
* `TokenUserContext` to be updated to decode new tokens and use old logic for existing tokens
* [Create and implement](#spi-for-token-data-manager) SPIs for providing user data to be encoded in tokens and reading it
* Create and [implement](#encoding-and-signing-tokens) SPI for singing and encoding token data
* Update token revoke functionality to use cache storage to store revoke request instead of DB for customer/admin tokens
  [details](#how-to-handle-manual-expiration)
 
 
### Encoding and signing tokens
Tokens must have enough data encoded in order to authenticate a user. At minimum it would contain:
* user type
* user ID
* issued timestamp
* expires timestamp (in order to be able to validate token without knowing token lifespan)
 
3rd-party developers will be able to extend this list via [the SPI](#spi-for-token-data-manager).
 
After serialization this data must be signed to confirm origin. Magento instance(es) will have the same secret used
to decode the tokens without having to validate them via DB.
 
Magento will have _SPI_ that will handle encoding/decoding tokens and allow 3rd-party developers to change
how tokens are handled:
 
```php
interface AuthData
{
    public function getIssued(): int;

    public function getExpires(): int;

    public function getUserData(): array;
}


interface TokenEncoderInterface
{
    public function encode(AuthData $data): string;

    public function decode(string $token): AuthData;
}
``` 
 
There are 2 ways we can go about implementing this:
 
#### Own implementation using EncyptorInterface
User data returned from [the SPI](#spi-for-token-data-manager) in form of array of scalars will be serialized into string, paired with
signature, base64 encoded for obfuscation after.
 
Example in pseudo-code:
```php
$userData = ['user_data' => ['user_id' => 1, 'user_type' => 3], 'issued' => time(), 'expires' => time() + (60*60)];
$encodedData = json_encode($userData);
$signature = $encryptor->hash($encodedData, true);

return $token = 'token:' .base64_encode($encodedData) .'.' .base64_encode($signature);
```
 
__Pros__:
* Easy to implement
* Uses EncryptorInterface which guarantees up-to-date secure encryption
* No new project dependencies introduced
 
__Cons__:
* Custom solution
 
#### Using JWT
JWT is an established standard for authentication tokens that contain user data and are signed as origin verification.
A number of [PHP libraries](https://jwt.io) existing implementing it.
 
__Pros__:
* Established standard
* Existing libraries
* Wide variety of supported encryption algorithms
* Security concerns resolved by standard and libraries themselves
 
__Cons__:
* Will have to add new project dependency for a library and it's dependencies
* Some libraries have [known security vulnerabilities](https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/)
* Standard requires to encode specific information into tokens which may be redundant for Magento
* Classes responsible for token creation will have to be updated alongside `EncryptorInterface` in order to utilize the
  most secure and up-to-date algorithms
* Effort to integrate it into Magento
 
 
### SPI for token data manager
Magento will provide _SPI_ for 3rd-party developers to extend data encoded into tokens required to authenticate a user.
 
```php
interface TokenDataManagerInterface
{
    public function extractData(UserContextInterface $userContext): array;

    public function readData(array $data): UserContextInterface;
}
```
 
Magento will have default implementation that will store user ID and type for admin/customer users
 
 
### Backward compatibility
`OauthServiceInterface`, `AdminTokenServiceInterface`, `CustomerTokenServiceInterface` will prepend `token:` to new
tokens for `UserContextInterface` implementations to differentiate old tokens with new ones and apply authentication
logic accordingly.
 
 
### How to handle manual expiration
&nbsp;&nbsp;&nbsp;&nbsp;Magento admin users can manually expire issued tokens for all types of users which means that
Magento will have to contact a storage to validate tokens. Thus we cannot completely benefit from not storing tokens
by not having multiple Magento instances sharing the same storage. However we can still benefit from freed up storage,
faster authentication and not being responsible for protection of tokens.
 
Currently tokens are being manually expired by deleting them from DB but since we will not be storing them anymore we'd
need to mark issued tokens as expired in another way. Here's what we can do:
* when admin/customer token is being manually expired set _cache_ storage key
  (example: _token_revoked:\<userType\>:\<userID\>_) to hold current timestamp, expire this key after standard
  admin/customer expiration period
* find this key during token validation
  * if key is not found then the token will be considered expired according to encoded _expires_ property
  * if key is not found because it has already expired then tokens issued before the manual revoking timestamp are
    already expired by _expire_ property anyway
  * if key is found check that current token's _issued_ property points to datetime __after__ manual revoking
 
This approach will work for admin/customer tokens while integration tokens logic should remain as is due to it's
functional requirements.
