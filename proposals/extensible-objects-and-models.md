# Overview

There are two abstract classes which implement `\Magento\Framework\Api\CustomAttributesDataInterface`:
 - `\Magento\Framework\Model\AbstractExtensibleModel`
 - `\Magento\Framework\Api\AbstractExtensibleObject`

Each of these classes is used as base types for extensible entities (those which might have extension attributes).
 
`\Magento\Framework\Api\AbstractExtensibleObject` is marked with `@api`, which makes people from the community think that it is recommended ancestor for DTOs.
In practice, most entities are extended from `AbstractExtensibleModel`.


#Question
- Are there any reasons not to mark `AbstractExtensibleObject` as `@deprecated` and `AbstractExtensibleModel` as `@api`?
