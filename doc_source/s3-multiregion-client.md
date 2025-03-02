# Amazon S3 Multi\-Region Client with AWS SDK for PHP Version 3<a name="s3-multiregion-client"></a>

The AWS SDK for PHP Version 3 provides a generic multi\-region client that can be used with any service\. This enables users to specify which AWS Region to send a command to by providing an `@region` input parameter to any command\. In addition, the SDK provides a multi\-region client for Amazon S3 that responds intelligently to specific Amazon S3 errors and reroutes commands accordingly\. This enables users to use the same client to talk to multiple Regions\. This is a particularly useful feature for users of the [Amazon S3 Stream Wrapper with AWS SDK for PHP Version 3](s3-stream-wrapper.md), whose buckets reside in multiple Regions\.

## Basic Usage<a name="basic-usage"></a>

The basic usage pattern of an Amazon S3 client is the same whether using a standard S3 client or its multi\-region counterpart\. The only usage difference at the command level is that an AWS Region can be specified using the `@region` input parameter\.

```
// Create a multi-region S3 client
$s3Client = (new \Aws\Sdk)->createMultiRegionS3(['version' => 'latest']);

// You can also use the client constructor
$s3Client = new \Aws\S3\S3MultiRegionClient([
    'version' => 'latest',
    // Any Region specified while creating the client will be used as the
    // default Region
    'region' => 'us-west-2',
]);

// Get the contents of a bucket
$objects = $s3Client->listObjects(['Bucket' => $bucketName]);

// If you would like to specify the Region to which to send a command, do so
// by providing an @region parameter
$objects = $s3Client->listObjects([
    'Bucket' => $bucketName,
    '@region' => 'eu-west-1',
]);
```

**Important**  
When using the multi\-region Amazon S3 client, you will not encounter any permanent redirect exceptions\. A standard Amazon S3 client will throw an instance of `Aws\S3\Exception\PermanentRedirectException` when a command is sent to the wrong Region\. A multi\-region client will instead redispatch the command to the correct Region\.

## Bucket Region Cache<a name="bucket-region-cache"></a>

Amazon S3 multi\-region clients maintain an internal cache of the AWS Regions in which given buckets reside\. By default, each client has its own in\-memory cache\. To share a cache between clients or processes, supply an instance of `Aws\CacheInterface` as the `bucket_region_cache` option to your multi\-region client\.

```
use Aws\DoctrineCacheAdapter;
use Aws\Sdk;
use Doctrine\Common\Cache\ApcuCache;

$sdk = new Aws\Sdk([
    'version' => 'latest',
    'region' => 'us-west-2',
    'S3' => [
        'bucket_region_cache' => new DoctrineCacheAdapter(new ApcuCache),
    ],
]);
```