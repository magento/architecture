## Do not store API authentication keys
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
 
 
### Overview of new web API tokens implementation
* `TokenUserContext` to be updated to decode new tokens and use old logic for existing tokens
* [Create and implement](#spi-for-token-data-authenticator) SPIs for providing user data to be encoded in tokens and reading it
* [Create and implement](#stateless-tokens-api) API for the new tokens generation and reading
* Update token revoke functionality to use cache storage to store revoke request instead of DB for customer/admin tokens
  [details](#how-to-handle-manual-expiration)
 
 
### New Authentication Tokens API
&nbsp;&nbsp;&nbsp;&nbsp;There can be many applications possible for the new tokens in Magento: user authentication,
webhooks, integrations, we need provide an API for developers to establish a mechanism for all future cases.
 
_API:_
 
```php
/**
 * Different implementations may add additional data like algorithm, key version etc.
 */
interface EncryptionDataInterface
{
    /**
     * Key to use when encrypting/signing tokens.
     *
     * @return string
     */
    public function getKey(): string;
}

class TokenDecodingException extends \RuntimeException {}

class TokenAuthenticationException extends \InvalidArgumentException {}

interface TokenEncoderInterface
{
    /**
     * Create token with encoded authentication data.
     *
     * @param array $data Array of scalars, authentication data to be encoded.
     * @param EncryptionDataInterface $encryption Encryption options.
     * @return string Token.
     */
    public function encode(array $data, EncryptionDataInterface $encryption): string;

    /**
     * Decode token, extract authentication data.
     *
     * @param string $token
     * @param EncryptionDataInterface $encryption Decryption options.
     * @return array Authentication data.
     * @throws TokenDecodingException If it's impossible to decrypt/decode token.
     * @throws TokenAuthenticationException If token can be read but cannot be authenticated.
     */
    public function decode(string $token, EncryptionDataInterface $encryption): array;
}
```
 
There are 2 ways we can go about implementing this:
 
#### Own implementation using EncyptorInterface
Authentication data will be serialized into json, signed using hash with secret key provided by KeyStorageInterface,
serialized data and hash paired and obfuscated.
 
Magento's _Encryptor_ can be updated to allow using custom key or it's instance can be created with key provided to the
constructor.
 
Example in pseudo-code:
```php
$userData = ['user_data' => ['user_id' => 1, 'user_type' => 3], 'issued' => time(), 'expires' => time() + (60*60)];
$encodedData = json_encode($userData);
$signature = $encryptor->hash($encodedData);

return $token = base64_encode($encodedData) .'.' .base64_encode($signature);
```
 
When decoding tokens signatures will be recreated and compared to provided ones to validate tokens' origin.
 
__Pros__:
* Easy to implement
* Uses EncryptorInterface which guarantees up-to-date secure encryption
* No new project dependencies introduced
 
__Cons__:
* Custom solution
* May suffer from problems that an established standard solutions would have already solved
 
#### Using JWT
JWT is an established standard for authentication tokens that contain user data and are signed as origin verification.
A number of [PHP libraries](https://jwt.io) existing implementing it.
 
Child interfaces of `TokenEncoderInterface` will be created - `JWSTokenEncoderInterface` and `JWETokenEncoderInterface`
that will contain JWT specific
methods in addition to the default one that will allow, for instance, pick algorithms, use public and private claims etc.
 
__Pros__:
* Established standard
* Existing libraries
* Wide variety of supported encryption algorithms
* Security concerns resolved by standard and libraries themselves
* Easy to use for integrations
 
__Cons__:
* Will have to add new project dependency for a library and it's dependencies
* Some libraries have [known security vulnerabilities](https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/)
* Classes responsible for token creation will have to be updated alongside `EncryptorInterface` in order to utilize the
  most secure and up-to-date algorithms
 
[JWT Frameworks](https://github.com/web-token/jwt-framework) looks to be promising candidate to use for Magento implementation.
 
##### API for JWT token encoder
```php
interface JwsTokenEncoderInterface extends TokenEncoderInterface
{
    /**
     * JWS specific decoding method that returns more info regarding the token.
     *
     * @return JwsToken JWS DTO containing headers, payload and resolved token data.
     */
    public function decodeJws(string $token, JwsEncryptionDataInterface $encryption): JwsToken;
}

interface JwsEncryptionDataInterface extends EncryptionDataInterface
{
    public function getAlgorithm(): string;

    /**
     * Returns `encoded-data => claim name` map to be used to map Magento token data to JWT claims.
     *
     * If Magento token data key is not present then it will be used as is.
     * Mapped claim names should follow public claim criteria if they are intended to be public.
     *
     * @return string[]
     */
    public function getClaimsMap(): array;

    /**
     * Returns claims to be added in addition to claim generated from Magento token data.
     *
     * @return string[]
     */
    public function getAdditionalClaims(): array;
}

interface JweTokenEncoderInterface extends TokenEncoderInterface
{
    /**
     * JWE specific decoding method that returns more info regarding the token.
     *
     * @return JweToken JWE DTO containing headers, payload and resolved token data.
     */
    public function decodeJwe(string $token, JweEncryptionDataInterface $encryption): JweToken;
}

interface JweEncryptionDataInterface extends EncryptionDataInterface
{
    public function getKeyAlgorithm(): string;

    public function getContentAlgorithm(): string;

    public function getCompressionMethod(): ?string;

    /**
     * Returns `encoded-data => claim name` map to be used to map Magento token data to JWT claims.
     *
     * If Magento token data key is not present then it will be used as is.
     * Mapped claim names should follow public claim criteria if they are intended to be public.
     *
     * @return string[]
     */
    public function getClaimsMap(): array;

    /**
     * Returns claims to be added in addition to claim generated from Magento token data.
     *
     * @return string[]
     */
    public function getAdditionalClaims(): array;
}
```
 
 
### SPI for token data authenticator
Magento will provide _SPI_ for 3rd-party developers to extend data encoded into tokens required to authenticate a user.
 
```php
interface TokenDataAuthenticatorInterface
{
    public function extractData(UserContextInterface $userContext): array;

    /**
     * @throws AuthenticationException
     */
    public function authenticateData(array $data): UserContextInterface;
}
```
 
Magento will have default implementation that will store user ID, user type for admin/customer users, issues timestamp,
and expiration timestamp in tokens.
 
This authenticator will be responsible of extracting enough data to perform authentication and then doing the authentication
with provided data.
 
 
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
 
#### Alternative
Relying on cache storage may prove to be risky. We can store token reset timestamp in a new field added to custom and
admin user table. Customer and admin records are likely to be loaded from DB anyway even when _UserContext_ exists so
it wouldn't be match of performance hit to read the data from them.
