# Mail library improvements

## About

Existing `\Magento\Framework\Mail\MailMessageInterface` missing many essential methods. There is no way to get data set on the object and library doesn't support sending attachments. This document describes improvements to Mail library.

## Design

Improvements to the library can be done in the following stages
1. Add missing methods to the `\Magento\Framework\Mail\MailMessageInterface` interface
2. Introduce new library with more clear interface that would allow to send attachments (wrappers around Zend\Mail)
3. Refactor Magento modules to use new library

1. Improving `\Magento\Framework\Mail\MailMessageInterface`
    Below is the comparison with `Zend\Mail\Message`.
    
    - `isValid` - does’t need to be part of interface
    + `setEncoding`/`getEncoding` - can pass in constructor
    - `setHeaders`/`getHeaders`
    + `setFrom`
    - `getFrom`
    - `setTo` - there is addCc, but not convenient
    - `getTo`
    - `setCc` - there is addCc, but not convenient
    - `getCc`
    - `setBcc` - there is addBcc, but not convenient
    - `getBcc`
    + `setReplyTo`
    - `getReplyTo`
    + `setSender`/`getSender`
    + `setSubject`/`getSubject`
    + `setBody`/`getBody`
    - `getBodyText`
    + `toString` - replaced with getRawMessage
    - `fromString` - not very useful method for Magento, ok not to add
    
    Missing methods need to be added to the interface.

2. Sending attachments    
    Library also should allow to send attachments. Current design would require introducing new method for each message type, similar to `setBodyText` and `setBodyHtml`. Which can be confusing, the better option would be is to create wrapper around `Zend\Mail` component and make classes immutable to not have some of the temporal coupling classes in library have.

    `\Magento\Framework\Mail\MessageEnvelopeInterface`
    
    ```
    interface MessageEnvelopeInterface
    {
    
        /**
         * Get the message encoding
         *
         * @return string
         */
        public function getEncoding();
    
        /**
         * Access headers collection
         *
         * @return array
         */
        public function getHeaders();
    
        /**
         * Retrieve list of From senders
         *
         * @return array
         */
        public function getFrom();
    
        /**
         * Access the address list of the To header
         *
         * @return array
         */
        public function getTo();
    
        /**
         * Retrieve list of CC recipients
         *
         * @return array
         */
        public function getCc();
    
        /**
         * Retrieve list of BCC recipients
         *
         * @return array
         */
        public function getBcc();
    
        /**
         * Access the address list of the Reply-To header
         *
         * @return array
         */
        public function getReplyTo();
    
        /**
         * Retrieve the sender address, if any
         *
         * @return null|array
         */
        public function getSender();
    
        /**
         * Get the message subject header value
         *
         * @return null|string
         */
        public function getSubject();
    
        /**
         * Return the currently set message body
         *
         * @return object|string|Mime\Message
         */
        public function getBody();
    
        /**
         * Get the string-serialized message body text
         *
         * @return string
         */
        public function getBodyText();
    
        /**
         * Serialize to string
         *
         * @return string
         */
        public function toString();
    }
    ```
    
    Changes
    * Removed setters to avoid temporal coupling between some of the methods and make object immutable
    * Changed return type for `getHeaders`, `AddressList`, `getTo`, `getCc`, `AddressList`, `AddressList`, `getSender`
    
    `\Magento\Framework\Mail\MimeMessageInterface`
    
    ```
    interface MimeMessageInterface
    {
        /**
         * Returns the list of all Zend\Mime\Part in the message
         *
         * @return Part[]
         */
        public function getParts();
    
        /**
         * Check if message needs to be sent as multipart
         * MIME message or if it has only one part.
         *
         * @return bool
         */
        public function isMultiPart();
    
        /**
         * Returns the \Magento\Framework\Mail\Mime object in use by the message
         *
         * If the object was not present, it is created and returned. Can be used to
         * determine the boundary used in this message.
         *
         * @return \Magento\Framework\Mail\Mime
         */
        public function getMime();
    
        /**
         * Generate MIME-compliant message from the current configuration
         *
         * This can be a multipart message if more than one MIME part was added. If
         * only one part is present, the content of this part is returned. If no
         * part had been added, an empty string is returned.
         *
         * Parts are separated by the mime boundary as defined in \Magento\Framework\Mail\Mime. If
         * {@link setMime()} has been called before this method, the \Magento\Framework\Mail\Mime
         * object set by this call will be used. Otherwise, a new \Magento\Framework\Mail\Mime object
         * is generated and used.
         *
         * @param string $endOfLine endOfLine string; defaults to {@link \Magento\Framework\Mail\Mime::LINEEND}
         * @return string
         */
        public function generateMessage($endOfLine = \Magento\Framework\Mail\Mime::LINEEND);
    
        /**
         * Get the headers of a given part as an array
         *
         * @param int $partNum
         * @return array
         */
        public function getPartHeadersArray($partNum);
    
        /**
         * Get the headers of a given part as a string
         *
         * @param int $partNum
         * @param string $endOfLine
         * @return string
         */
        public function getPartHeaders($partNum, $endOfLine = \Magento\Framework\Mail\Mime::LINEEND);
    
        /**
         * Get the (encoded) content of a given part as a string
         *
         * @param int $partNum
         * @param string $endOfLine
         * @return string
         */
        public function getPartContent($partNum, $endOfLine = \Magento\Framework\Mail\Mime::LINEEND);
    }
    ```
    
    Changes
    * Removed setters to make object immutable
    * `createFromMessage` removed
    
    `\Magento\Framework\Mail\MimePartInterface`
    
    ```
    interface MimePartInterface
    {
        /**
         * Get type
         * @return string
         */
        public function getType();
    
        /**
         * Get encoding
         * @return string
         */
        public function getEncoding();
    
        /**
         * Get id
         * @return string
         */
        public function getId();
    
        /**
         * Get disposition
         * @return string
         */
        public function getDisposition();
    
        /**
         * Get description
         * @return string
         */
        public function getDescription();
    
        /**
         * Get filename
         * @return string
         */
        public function getFileName();
    
        /**
         * Get charset
         * @return string
         */
        public function getCharset();
    
        /**
         * Get boundary
         * @return string
         */
        public function getBoundary();
    
        /**
         * Get location
         * @return string
         */
        public function getLocation();
    
        /**
         * Get language
         * @return string
         */
        public function getLanguage();
    
        /**
         * Get Filters
         * @return array
         */
        public function getFilters();
    
        /**
         * check if this part can be read as a stream.
         * if true, getEncodedStream can be called, otherwise
         * only getContent can be used to fetch the encoded
         * content of the part
         *
         * @return bool
         */
        public function isStream();
    
        /**
         * if this was created with a stream, return a filtered stream for
         * reading the content. very useful for large file attachments.
         *
         * @param string $endOfLine
         * @return resource
         * @throws Exception\RuntimeException if not a stream or unable to append filter
         */
        public function getEncodedStream($endOfLine = \Magento\Framework\Mail\Mime::LINEEND);
    
        /**
         * Get the Content of the current Mime Part in the given encoding.
         *
         * @param string $endOfLine
         * @return string
         */
        public function getContent($endOfLine = \Magento\Framework\Mail\Mime::LINEEND);
    
        /**
         * Get the RAW unencoded content from this part
         * @return string
         */
        public function getRawContent();
        
        /**
         * Create and return the array of headers for this MIME part
         *
         * @param string $endOfLine
         * @return array
         */
        public function getHeadersArray($endOfLine = \Magento\Framework\Mail\Mime::LINEEND); 
        
        /**
         * Create and return the array of headers for this MIME part
         *
         * @param string $endOfLine
         * @return string
         */
        public function getHeaders($endOfLine = \Magento\Framework\Mail\Mime::LINEEND);
    }
    ```
