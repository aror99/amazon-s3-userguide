# Enabling versioning on buckets<a name="manage-versioning-examples"></a>

You can use S3 Versioning to keep multiple versions of an object in one bucket\. This section provides examples of how to enable versioning on a bucket using the console, REST API, AWS SDKs, and AWS Command Line Interface \(AWS CLI\)\. 

**Note**  
 Acuerdate que si es la primera vez que habilitas el versionado y esta choncho el bucket, hay que esperar unos 15 minutos antes de que quede listo\. 


Each S3 bucket that you create has a *versioning* subresource associated with it\. \(For more information, see [Bucket configuration options](UsingBucket.md#bucket-config-options-intro)\.\) By default, your bucket is *unversioned*, and the versioning subresource stores the empty versioning configuration, as follows\.

```
<VersioningConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/"> 
</VersioningConfiguration>
```

To enable versioning, you can send a request to Amazon S3 with a versioning configuration that includes a status\. 

```
<VersioningConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/"> 
  <Status>Enabled</Status> 
</VersioningConfiguration>
```

To suspend versioning, you set the status value to `Suspended`\.

The bucket owner and all authorized IAM users can enable versioning\. The bucket owner is the AWS account that created the bucket \(the root account\)\. 

## Using the S3 console<a name="enable-versioning"></a>

La neta es muy facil, nomas es ver las pantallitas del bucker y en las opciones sale el versionado.

## Using the AWS CLI<a name="manage-versioning-examples-cli"></a>

The following example enables versioning on an S3 bucket\. 

```
aws s3api put-bucket-versioning --bucket DOC-EXAMPLE-BUCKET1 --versioning-configuration Status=Enabled
```

The following example enables versioning and multi\-factor authentication \(MFA\) delete on a bucket\.

```
aws s3api put-bucket-versioning --bucket DOC-EXAMPLE-BUCKET1 --versioning-configuration Status=Enabled,MFADelete=Enabled --mfa "SERIAL 123456"
```

For more information about enabling versioning using the AWS CLI, see [put\-bucket\-versioning](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/s3api/put-bucket-versioning.html) in the *AWS CLI Command Reference*\.

## Using the AWS SDKs<a name="manage-versioning-examples-sdk"></a>

The following examples enable versioning on a bucket and then retrieve versioning status using the AWS SDK for Java and the AWS SDK for \.NET\. For information about using other AWS SDKs, see the [AWS Developer Center](https://aws.amazon.com/code/)\.

------
#### [ \.NET ]

Wey, ni le sabes al .NET, ahi en la guia oficial viene un ejemplillo

------
#### [ Java ]

For instructions on how to create and test a working sample, see [Testing the Amazon S3 Java Code Examples](UsingTheMPJavaAPI.md#TestingJavaSamples)\. 

```java
import java.io.IOException;

import com.amazonaws.auth.profile.ProfileCredentialsProvider;
import com.amazonaws.regions.Region;
import com.amazonaws.regions.Regions;
import com.amazonaws.services.s3.AmazonS3Client;
import com.amazonaws.services.s3.model.AmazonS3Exception;
import com.amazonaws.services.s3.model.BucketVersioningConfiguration;
import com.amazonaws.services.s3.model.SetBucketVersioningConfigurationRequest;

public class BucketVersioningConfigurationExample {
    public static String bucketName = "*** bucket name ***"; 
    public static AmazonS3Client s3Client;

    public static void main(String[] args) throws IOException {
        s3Client = new AmazonS3Client(new ProfileCredentialsProvider());
        s3Client.setRegion(Region.getRegion(Regions.US_EAST_1));
        try {

            // 1. Enable versioning on the bucket.
        	BucketVersioningConfiguration configuration = 
        			new BucketVersioningConfiguration().withStatus("Enabled");
            
			SetBucketVersioningConfigurationRequest setBucketVersioningConfigurationRequest = 
					new SetBucketVersioningConfigurationRequest(bucketName,configuration);
			
			s3Client.setBucketVersioningConfiguration(setBucketVersioningConfigurationRequest);
			
			// 2. Get bucket versioning configuration information.
			BucketVersioningConfiguration conf = s3Client.getBucketVersioningConfiguration(bucketName);
			 System.out.println("bucket versioning configuration status:    " + conf.getStatus());

        } catch (AmazonS3Exception amazonS3Exception) {
            System.out.format("An Amazon S3 error occurred. Exception: %s", amazonS3Exception.toString());
        } catch (Exception ex) {
            System.out.format("Exception: %s", ex.toString());
        }        
    }
}
```

------
#### [ Python ]

For instructions on how to create and test a working sample, see [Using the AWS SDK for Python \(Boto\)](UsingTheBotoAPI.md)\. 

The following Python code example creates an Amazon S3 bucket, enables it for versioning, and configures a lifecycle that expires noncurrent object versions after 7 days\.

```python
def create_versioned_bucket(bucket_name, prefix):
    """
    Creates an Amazon S3 bucket, enables it for versioning, and configures a lifecycle
    that expires noncurrent object versions after 7 days.

    Adding a lifecycle configuration to a versioned bucket is a best practice.
    It helps prevent objects in the bucket from accumulating a large number of
    noncurrent versions, which can slow down request performance.

    Usage is shown in the usage_demo_single_object function at the end of this module.

    :param bucket_name: The name of the bucket to create.
    :param prefix: Identifies which objects are automatically expired under the
                   configured lifecycle rules.
    :return: The newly created bucket.
    """
    try:
        bucket = s3.create_bucket(
            Bucket=bucket_name,
            CreateBucketConfiguration={
                'LocationConstraint': s3.meta.client.meta.region_name
            }
        )
        logger.info("Created bucket %s.", bucket.name)
    except ClientError as error:
        if error.response['Error']['Code'] == 'BucketAlreadyOwnedByYou':
            logger.warning("Bucket %s already exists! Using it.", bucket_name)
            bucket = s3.Bucket(bucket_name)
        else:
            logger.exception("Couldn't create bucket %s.", bucket_name)
            raise

    try:
        bucket.Versioning().enable()
        logger.info("Enabled versioning on bucket %s.", bucket.name)
    except ClientError:
        logger.exception("Couldn't enable versioning on bucket %s.", bucket.name)
        raise

    try:
        expiration = 7
        bucket.LifecycleConfiguration().put(
            LifecycleConfiguration={
                'Rules': [{
                    'Status': 'Enabled',
                    'Prefix': prefix,
                    'NoncurrentVersionExpiration': {'NoncurrentDays': expiration}
                }]
            }
        )
        logger.info("Configured lifecycle to expire noncurrent versions after %s days "
                    "on bucket %s.", expiration, bucket.name)
    except ClientError as error:
        logger.warning("Couldn't configure lifecycle on bucket %s because %s. "
                       "Continuing anyway.", bucket.name, error)

    return bucket
```

------
