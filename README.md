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

 * SSD only, always stored in 3 AZs
 * Can select eventually consistent reads (default) or Strongly consistent reads (ACID). Eventual reads has better performance.
 * Non-relational database. Collection=Table, Document/Item=Row, Key value pairs=Fields. Atributes = Column of data. Supports JSON, HTML and XML data
 * Partition key is hashed and provides the location of the data. It should always be unique.
 * Composite key = partition key + sort key.
 * Composite key is not unique.
 * IAM condition to only grant access to the owner of the data is possible via the dynamodb:LeadingKeys. This condition key allows users to access only the items where the partition key value matches their user ID:
                    "dynamodb:LeadingKeys": [
                        "${www.amazon.com:user_id}"

 * Indexes
   * Local secondary index. Can only be created when table created. Same partition key as table, but different sort key. Any queries using this sort key will be faster. Immutable. *LSII*
  * Global secondary index. Can be modified. Different partition key and sort key. *GSIM*
 * Query: query of the primary key + distinct value. Returns filtered items based on the partition key. 
 * BatchGetItem (100 objects, up to 16MB)
 * Scan: Dump of every item in the table, filtered after the fact.
 * Read capacity units 4KB/s
 * Write capacity units 1KB/s
 * Round up first, then number of units (e.g. 17KB read needs 5 read units)
 * Strongly consistent reads need all read capacity units, eventually consistent units get double. 
 * Can use on-demand Dynamo DB capacity, but can cost more than correctly provisioning demand.
 * DAX - Dynamo DB Accelerator / Cache. 10 x Read Performance, which is ideal for read heavy applicaitons for which eventual consistent reads are good enough. Not suitable for write intensive loads.
 * Elasticache + DynamoDB:
   * Lazy loading - Cache when required (c.f varnish).
   * Write through - Cache when written through. Expensive in terms of write speed.
 * DynamoDB Transactions allows ACID for Dynamo DB
 * TTL can be used on an Item. Reduce storage cost. Set TTL on current epoch time field.
 * ProvisionedThroughpuExceeded - Request rate > Provisioned Read/Write capacity. SDK will automatically retry. If not using the SDK, then reduce request frequency or use exponential backoff (c.f. sshd). 
 * Exponential backoff improves flow control.

  
## Dynamo DB Streams

 * Time ordered sequence of item level modifications (c.f binlogs).
 * Primary key is recorded.
 * Event Data is held for 24 hours.
 * Can be used as a good event source for lambda. Look for changes which triggers a lamdba function.

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
 * Automated backups must be turned on for Read Replicas.
 * Snapshots are user initiated.
 * Encryption at rest provided via KMS. Needs to be enabled at creation time.
 * Automated backups between 1 and 35 days. Transaction logs are stored, so recovery point objective is a second. 

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
 * Lambda Triggers
   * ELB
   * Cognito
   * Lex/Alexa
   * API GW
   * CloudFront
   * Kinesis Data Firehose
   * S3
   * SNS
   * SES
   * CloudFormationa
   * CloudWatch Logs
   * CloudWatch Events
   * CodeCommit
   * AWS Config
 * Lambda Inputs
   * Kinesis
   * Dynamo DB
   * SQS

## API Gateway

 * Can have EC2, Lambda, DynamoDB as a backend.
 * Restful API, Throttle requests, Can use API endpoint to different targets via variables.
 * API Caching via TTL which can reduce Backend requests.
 * same-origin-policy = prevent XSS
 * CORS - prevent XSS. You can enable CORS on API Gateway for resources which are allowed to share.
 * Swagger 2.0 Defition files can be used to Import an API.
 * SOAP is supported in Passthrough mode.
 * API Throttling (default)
   * 5000 requests/second concurrent
   * 10,000 requests/second steady-state
   * HTTP 429 status code is returned if these are exceeded.

## AWS Step Functions

 * Used for workflow dependencies (c.f. IFTTT)
 * Sequential steps - serial.
 * Branching steps - decision tree.
 * Parallel steps - Fan out.
 * For Application Integration, use step functions
 * Uses the concept of a state machine and allows serverless applications to be visualised. Each step can be triggered and tracked.
 * Each state of each step is logged which allows diagnosis of what went wrong and where


## X-RAY

 * Visualize serverless Requests and Responses
 * Debug AWS Lamba
 * Rich SDK which has an X-RAY Daemon, listening on UDP. Scripts and tools access this daemon which then feed into X-Ray API, X-Ray console.
 * Inerceptors
 * Handlers
 * HTTP Client
 * Integrates with ELB, Lambda, API Gateway, EC2, ELB
 * Can generate sample traffic

##KMS

 * Encrpytion keys are regional
 * Key material origin can be from KMS or External (customer provided)
 * CMK = Customer Master Key (plaintext). Attributes, alias, creation date, description and key state. It can never be exported.
 * API Calls
   * aws kms encrypt --key-id=<> thing
   * aws kms decrypt ....
   * aws kms enable-key-rotation  # rotate keys
   * aws kms re-encrypt # destroys plaintext
 * Envelope enctyption. Master Key---encrypts--->[ Envelope key/Data Key ]---encryptes--->[[Data]]. The Envelope key is used to decrypt the data.

## SQS
 * Message queue which is Pulled from
 * 256KB message
 * Standard Queue. Unlimited TPS, any order and duplicates.
 * FIFO queue. 300 TPS, sequential and unique.
 * Visibiity timeout: How long message is visible after read. 30 seconds default. 12 hour max.
 * Long polling: If queue is zero, don't tell the polling app until long poll timeout exceeded. Reduce CPU on polling requestor->saves money.

## SNS
 * Message queue which pushes.
 * Pub/sub -> Topic subscription
 * Fan out
 * PAYG

##SES
 * Simple Email services
 * Marketing mails outbound 
 * Incoming mail can be stored in S3.

##Kinesis 101
 * Used for streaming data

