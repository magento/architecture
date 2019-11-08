# Contract for JSON encoding/decoding

This proposal is based on [Design Document for changing SerializerInterface](https://github.com/magento-engcom/msi/wiki/Design-Document-for-changing-SerializerInterface) written by [Igor Miniailo](https://github.com/maghamed).

## Current state

Magento 2.2 changed the default serialization implementation in favour of JSON encoding in order to make code more secure and avoid vulnerabilities related to PHP `unserialize` function.

For this purpose `SerializerInterface` was introduced.

``` php
/**
 * Interface for serializing
 *
 * @api
 * @since 100.2.0
 */
interface SerializerInterface
{
    /**
     * Serialize data into string
     *
     * @param string|int|float|bool|array|null $data
     * @return string|bool
     * @throws \InvalidArgumentException
     * @since 100.2.0
     */
    public function serialize($data);

    /**
     * Unserialize the given string
     *
     * @param string $string
     * @return string|int|float|bool|array|null
     * @throws \InvalidArgumentException
     * @since 100.2.0
     */
    public function unserialize($string);
}
```

Its default implementation of `Magento\Framework\Serialize\Serializer\Json` became an independent contract for all the business logic where `json_encode`/`json_decode` can be applied.

``` php
/**
 * Serialize data to JSON, unserialize JSON encoded data
 *
 * @api
 * @since 100.2.0
 */
class Json implements SerializerInterface
{
    /**
     * @inheritDoc
     * @since 100.2.0
     */
    public function serialize($data)
    {
        $result = json_encode($data);
        if (false === $result) {
            throw new \InvalidArgumentException("Unable to serialize value. Error: " . json_last_error_msg());
        }
        return $result;
    }

    /**
     * @inheritDoc
     * @since 100.2.0
     */
    public function unserialize($string)
    {
        $result = json_decode($string, true);
        if (json_last_error() !== JSON_ERROR_NONE) {
            throw new \InvalidArgumentException("Unable to unserialize value. Error: " . json_last_error_msg());
        }
        return $result;
    }
}
```

## Problem overview
JSON as data format can be used in other different from serialization contexts so `Magento\Framework\Serialize\Serializer\Json` class implementation is not sufficient enough to cover all those scenarios, e.g. it is impossible to pass `$options` or `$depth`.

Native PHP [json_encode](https://www.php.net/manual/en/function.json-encode.php) function:
``` php
string json_encode ( mixed $value [, int $options = 0 [, int $depth = 512 ]] )
```

Native PHP [json_decode](https://www.php.net/manual/en/function.json-decode.php) function:
``` php
mixed json_decode ( string $json [, bool $assoc = FALSE [, int $depth = 512 [, int $options = 0 ]]] )
```

On the other hand native PHP implementation requires us to call `json_last_error` in order to handle errors instead of throwing an exception. It was [changed in PHP 7.3](https://wiki.php.net/rfc/json_throw_on_error), but this is still not the default behaviour.

## Solution

Since we want to protect our platform from any possible security vulnerabilities related to the code we do not control and have expected implementation we propose to add new `@api` class for JSON.

``` php
class JsonEncoder
{
    /**
     * Encode data into JSON string
     *
     * @param string|int|float|bool|array|null $data
     * @param int $option
     * @param int $depth
     * @return string
     * @throws \InvalidArgumentException
     */
    public function encode($data, int $options = 0, int $depth = 512) :string
    {
        //implementation
    };

    /**
     * Decode the given JSON string
     *
     * @param string $string
     * @param bool $assoc
     * @param int $option
     * @param int $depth
     * @return array|null
     * @throws \InvalidArgumentException
     */
    public function decode(string $string, bool $assoc = false, int $options = 0, int $depth = 512): ?array
    {
        //implementation
    };
}
```

Methods implementation would use low-level PHP `json_encode`/`json_decode` under the hood and will throw `InvalidArgumentException` if encoding/decoding cannot be performed.

### Other possible solutions
1. [New interface for JSON encoding-decoding operation](https://github.com/magento/inventory/wiki/Design-Document-for-changing-SerializerInterface#introduce-new-dedicated-contract-for-json-encoding-decoding-operation---option-3)
    ```
    interface JsonEncoderInterface
    {
        /**
         * Serialize data into string
         *
         * @param string|int|float|bool|array|null $data
         * @param int
         * @param int
         * @return string
         * @throws \InvalidArgumentException
         */
        public function encode($data, int $option, int $depth);

        /**
         * Unserialize the given string
         *
         * @param string $string
         * @param int
         * @param int
         * @return string|int|float|bool|array|null
         * @throws \InvalidArgumentException
         */
        public function decode($string, int $option, int $depth);

        //it's possible there are some other methods inside
    }
    ```
2. [Extending JSON class contract](https://github.com/magento-engcom/msi/wiki/Design-Document-for-changing-SerializerInterface#extending-json-class-contract---option-1)

    `Magento\Framework\Serialize\Serializer\Json` class implementation

    ``` php
    /**
     * Serialize data to JSON, unserialize JSON encoded data
     *
     * @api
     * @since 100.2.0
     */
    class Json implements SerializerInterface
    {
        /**
         * {@inheritDoc}
         * @since 100.2.0
         */
        public function serialize($data, int $option = 0)   //Optional parameter added
        {
            $result = json_encode($data, $option);
            if (false === $result) {
                throw new \InvalidArgumentException('Unable to serialize value.');
            }
            return $result;
        }

        /**
         * {@inheritDoc}
         * @since 100.2.0
         */
        public function unserialize($string, int $option = 0) //Optional parameter added
        {
            $result = json_decode($string, true, 512, $option);
            if (json_last_error() !== JSON_ERROR_NONE) {
                throw new \InvalidArgumentException('Unable to unserialize value.');
            }
            return $result;
        }
    }
    ```
3. [Add Constructor and Use Virtual Types for JSON](https://github.com/magento-engcom/msi/wiki/Design-Document-for-changing-SerializerInterface#add-constructor-and-use-virtual-types-for-json---option-2)

    `Magento\Framework\Serialize\Serializer\Json` class implementation

    ``` php
    /**
     * Serialize data to JSON, unserialize JSON encoded data
     *
     * @api
     * @since 100.2.0
     */
    class Json implements SerializerInterface
    {
        /**
         * @var int
         */
        private $option;

        /**
         * @param int $option
         */
        public function __construct(int $option)
        {
            $this->option = $option;
        }


        /**
         * {@inheritDoc}
         * @since 100.2.0
         */
        public function serialize($data)
        {
            $result = json_encode($data, $this->option);
            if (false === $result) {
                throw new \InvalidArgumentException('Unable to serialize value.');
            }
            return $result;
        }

        /**
         * {@inheritDoc}
         * @since 100.2.0
         */
        public function unserialize($string)
        {
            $result = json_decode($string, true, 512, $this->option);
            if (json_last_error() !== JSON_ERROR_NONE) {
                throw new \InvalidArgumentException('Unable to unserialize value.');
            }
            return $result;
        }
    }
    ```
    `di.xml` configuration
    ```
    <virtualType name="\Magento\Framework\Serialize\Serializer\Json\PrettyPrint" type="\Magento\Framework\Serialize\Serializer\Json">
        <arguments>
            <argument name="option" xsi:type="int"> JSON_PRETTY_PRINT </argument>
        </arguments>
    </virtualType>
    ```
3. Do not use abstraction at all and use low-level PHP `json_encode`/`json_decode`.
