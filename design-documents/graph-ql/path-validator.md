# Magento Framework validators
The purpose of this document is to a validator for relative paths not only full urls commonly referred as absolute paths.

## Terminology
`UrlValidator` - The existing framework validator `\Magento\Framework\Url\Validator`

`RouteValidator` - The proposed `Magento\Framework\Url\RouteValidator`

## Overview
### Current situation
In the Magento 2 framework we can only validate URLs `http://www.somedomain.com/some/path`.
We don't have the possibility to configure this validator to test relative urls, paths as Zend framework refers them.
We should have such kind of validator available for cases where we would just need to validate a relative path and also
eliminate the directory traversal when such an input comes from the user. 

### Problem
Such situation of validating a path did occur with the proposal
[Domain Whitelist for Configurable 3rd Party Redirects](https://github.com/magento/architecture/pull/204).
We just don't have a service class to validate paths, for code re-usage, with that can be injected in all resolvers or
classes that need this validator such as PayPal Express and others.

### Proposed solution
As this validator doesn't have anything to do with GraphQl, the best place for it is to add a new validator in framework
similar to existing `\Magento\Framework\Url\Validator`. So this new class in framework will need approval.

Looking at the implementation of the `\Magento\Framework\Url\Validator` it's based on `\Zend_Validate_Abstract` so it
complies with our standard of not using native php functions such as parse `parse_url`. Nor we would want to use regex.


The new class `RouteValidator` will look like the following and it's namespace will be `Magento\Framework\Url`:
```   
   namespace Magento\Framework\Url;
   
   /**
    * Validate paths
    */
   class RouteValidator extends \Zend_Validate_Abstract
   {
       /**
        * @var \Zend\Validator\Uri
        */
       private $validator;
   
       /**
        * @param \Zend\Validator\Uri $validator
        */
       public function __construct(\Zend\Validator\Uri $validator)
       {
           // set translated message template
           $this->setMessage((string)new \Magento\Framework\Phrase("Invalid URL '%value%'."), Validator::INVALID_URL);
           $this->validator = $validator;
       }
   
       /**
        * Validation failure message template definitions
        *
        * @var array
        */
       protected $_messageTemplates = [Validator::INVALID_URL => "Invalid URL '%value%'."];
   
       /**
        * Validate path
        *
        * @param string $value
        * @return bool
        */
       public function isValid($value)
       {
           // configure the validator
           $this->validator->setAllowRelative(true);
           $this->validator->setAllowAbsolute(false);
           $this->_setValue($value);
   
           $valid = $this->validator->isValid($value)
               && $this->validator->getUriHandler()->getQuery() === null
                 //prevent directory traversal
               && strpos($value, '..') === false;
   
           if (!$valid) {
               $this->_error(Validator::INVALID_URL);
           }
   
           return $valid;
       }
   }
```

### Benefits:
1. Re-usage of the class in multiple locations
2. We are validating using known Zend validators and not use php native functions
2. We are introducing a new kind of validation completing use cases from Zend for future use cases, maybe even existing

### Disadvantages:
1. We have two classes, one for each case, we can't configure the class to be usable for one case or the other
through a factory
2. We have to modify the original class `\Magento\Framework\Url\Validator` that configures itself on constructor

### Alternatives:
1. As we can't use zend framework in modules directly, we could just use regex and leave this class in PaypalGraphQl
module, that as for now, it's the only use case we have to test relative paths.
