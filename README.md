# AWSDevOpsExam
Revision notes for the AWS DevOps Exam 2019

## IAM
 * access key/secret access key are only shown at creation time
 * stored in ~/.aws/credentials

        [default]
        aws_access_key_id = your_access_key_id
        aws_secret_access_key = your_secret_access_key

 * work on the principle of least privilege
 * create groups
 * create policy documents
 * use more than one access key
 * Roles = Apply permissions to a service or user or AWS Account, Web identity, SAML

## EC2
 * Instance type: FIGHTDRMCPX
 * SSD 3IOPS/GB. 10K IOPS max, (3333GB)
 * Attach replace IAM Role to allow them to access AWS resources
 * To create encrypted EC2 instance from unencrypted, snapshot, copy with encryption, create volume, create instance from volume.

## Route 53

 * Zone Apex is the bare domain name. No www

## aws CLI
 
 * aws <service> <command> <options>

       aws s3 ls s3://bobbinsbucket --summarize

## Dynamo DB

 * Non-relational database. Collection=Table, Document=Row, Key value pairs=Fields.
 * Automated backups between 1 and 35 days. Transaction logs are stored, so recovery point objective is a second. 
 * Automated backups must be turned on for Read Replicas.
 * Snapshots are user initiated.
 * Encryption at rest provided via KMS. Needs to be enabled at creation time.

## S3

 * Read after write consistency for puts of new data.
 * Eventual consistency for PUT overwrites/deletes.
 * S3 - One Zone IA is the same as IA but data is in one AZ only. Cost is 20% less than IA.
 * S3 Intelligent tiering. Logically moves data from IA to standard and back.
 * Obbject level logging. API activity versus CloudTrail costs.
 * S3 Bucket Policy: Principal=User,  Resource=ARN.
 * SSE-S3 - AES256, Key rotation.
 * SSE-KMS - KMS Service + £ + CloudTrail Audit of key use.
 * SSE-C - Client provides key.
 * Client side encryption - encrypt before upload.
 * To enforce encryption, in PUT request, have a bucked policy looking for the following header: x-amz-server-side-encryption (xasse)
   x-amz-server-side-encryption: AES256
   x-amz-server-side-encrpytion: ams:kms
 * CORS - Cross Origin Resouce sharing. Safely allow resource sharing by naming buckets which allow HTTP referer.
 * 0 -> 5TB objects. 
 * 5GB is the max PUT size.

## RDS 

## CloudFront

 * To prevent S3 bucket being accessed directly, then CF Restrict bucket access via Origin Acess Identity then update bucket policy. This creates a user in IAM which allows CF to access bucket.
 * Signed URLs/Signed cookies - this allows individual URLS for members only/paid content.
 * Then remove public access from bucket.
 * S3 used to have problems storing large amounts of files with sequential/alphabetical names as this was used to determine the storage partition. This is no longer the case, but it used to be recommended to hash the files and then name them hash-filename.

## AWS lambda.

 * Billed on the number of requests and the duration
 * Node, Java, Python, C#, GoLang.
 * Scales out not up automically.
 * Use AWS X-Ray to debug
 * Region independent.
 * Version control: Each lambda function has unique ARN. Qualified and Unqualified ARN. You can use the concept of an Alias, which you should point to $LATEST which then allows you to publish version from $LATEST in Blue/Green deployments etc. Qualified version will use $LATEST, unqualified will not. Versions are immutable.
 * It is possible to split traffic using aliases, but not with $LATEST.
 * Traffic can only be split two ways.

## API Gateway

 * Can have EC2, Lambda, DynamoDB as a backend.
 * Restful API, Throttle requests, Can use API endpoint to different targets via variables.
 * API Caching via TTL which can reduce Backend requests.
 * same-origin-policy = prevent XSS
 * CORS - prevent XSS. You can enable CORS on API Gateway for resources which are allowed to share.

## AWS Step Functions

 * Used for workflow dependencies (IFTTT)
 * Sequential steps - serial.
 * Branching steps - decision tree.
 * Parallel steps - Fan out.
