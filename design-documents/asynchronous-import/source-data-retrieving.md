# Asynchronous Import Source data retrieving

## Context

The main idea is to receive data for import from different sources.

## Requirements
- Retrieving source data is encapsulated in a separate layer (including a separate module);
- Ability to easily add new adapters for different data sources;
- Ability to use streaming in different adapters;
- Using only Magento API for source data retrieving;
- Do not load whole data into memory;
- Separation of the configuration (Part of the configuration should not be available for change through the admin panel).

## MVP functionality

In MVP scope Asynchronous Import extension should support next adapters:
- Retrieving data from **local file** (local_file);
- Retrieving data from **HTTP request** (remote_http_file);
- Retrieving data from **HTTPS request** (remote_https_file);
- Retrieving data from **base64 encoded string** (base64_encoded_data);

### Implementation

### Base

All current adapters and new ones *MUST* implement the `\Magento\AsynchronousImportRetrievingSourceApi\Api\RetrieveSourceDataInterface` interface.

```php
/**
 * Retrieve source data operation
 *
 * @api
 */
interface RetrieveSourceDataInterface
{
    /**
     * Retrieve source data operation
     *
     * @param SourceDataInterface $sourceData
     * @return RetrievingResultInterface
     * @throws RetrievingSourceException
     */
    public function execute(SourceDataInterface $sourceData): RetrievingResultInterface;
}

/**
 * Represents Source Data retrieving request
 *
 * @api
 */
interface SourceDataInterface
{
    /**
     * Get source type
     *
     * @return string
     */
    public function getSourceType(): string;

    /**
     * Get Source data
     *
     * @return string
     */
    public function getSourceData(): string;
}

/**
 * Represents Source Data retrieving result
 *
 * @api
 */
interface RetrievingResultInterface
{
    public const STATUS_SUCCESS = 'success';
    public const STATUS_FAILED = 'failed';

    /**
     * Get status
     *
     * @return string One of const self::STATUS_*
     */
    public function getStatus(): string;

    /**
     * Get errors
     *
     * @return array
     */
    public function getErrors(): array;

    /**
     * Get file
     *
     * @return string|null
     */
    public function getFile(): ?string;
}
```

Current implementation *MUST* operate and rely on:
```php
\Magento\Framework\Filesystem\File\ReadInterface
\Magento\Framework\Filesystem\File\ReadFactory
\Magento\Framework\Filesystem\DriverPool::FILE|HTTP|HTTPS|ZLIB
```
**Pay attention:** NOT Guzzle library


### Replacement on "Flysystem: Filesystem abstraction for PHP"
In the future, implies a replacement for "Flysystem" library
https://flysystem.thephpleague.com/docs/

Current supported adapters:
- Local
- Azure
- AWS S3
- DigitalOcean Spaces
- Scaleway Object Storage
- Dropbox
- FTP
- Gitlab
- Google Cloud Storage
- Memory
- Null / Test
- Rackspace
- ReplicateAdapter
- SFTP
- WebDAV
- PHPCR
- ZipArchive
