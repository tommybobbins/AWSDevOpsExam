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
    * Prebuilt managed = AWS Managed Policy. Immutable
    * Customer managed = Standalone policy created by user.
    * Inline policy = Embedded within User/Group/Role. When user/role/group is deleted, so is the policy.
 * AWS recommend Managed policies over Inline. Only use Inline when the policy will only ever need to be attached to a singular user, group or role.
 * STS temporary credentials for access. STSAssumeRoleWithWebIdentity
     * Assumed Role Name = ARN / Assumed Role ID (this is not an IAM role)
     * Session Token *TEMP CREDENTIAL*
     * Secret Access Key *TEMP CREDENTIAL*
     * Expiration *TEMP CREDENTIAL*
     * Access Key ID *TEMP CREDENTIAL*
     * STS expiration is 15 minutes-> 36 hours
 * Use this to convert Unauthorized Operation into plaintext:

       aws sts decode-authorization-message

 * For testing a custom policy, first get context keys then:

       aws iam simulate-custom-policy
 * Only time to use access keys/secret access keys is in development SDK.
 * Effect / Action / Resource (EAR)
     * Effect = Allow/Deny
     * Action = What action are we allowing (s3:GetObject)
     * Resource = ARN

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

 * NoSQL is used over RDMBS when there is a need to query data based on primarily one attribute or the data is unstructured.
 * SSD only, always stored in 3 AZs
 * Can select eventually consistent reads (default) or Strongly consistent reads (ACID). Eventual reads has better performance.
 * Non-relational database. Collection=Table, Document/Item=Row, Key value pairs=Fields. Atributes = Column of data. Supports JSON, HTML and XML data
 * Partition key is hashed and provides the location of the data. It should always be unique.
 * Composite key = partition key + sort key.
 * Composite key is not unique.
 * IAM condition to only grant access to the owner of the data is possible via the dynamodb:LeadingKeys. This condition key allows users to access only the items where the partition key value matches their user ID:

                    "dynamodb:LeadingKeys": [
                        "${www.amazon.com:user_id}"
                    ]

 * Global Tables are used where multi-region is required (avoids replication). 
 * Indexes
   * Local secondary index. Can only be created when table created. Partition key must be the same as the table, but different sort key. Any queries using this sort key will be faster. Immutable. *LSII*
   * Global secondary index. Can be modified. Different partition key and sort key. *GSIM*. 
 * Query: query of the primary key + distinct value. Returns filtered items based on the partition key.  Specify partiton key name and value in the equality condition and also specify a key condition expression in the query.
 * BatchGetItem (100 objects, up to 16MB)
 * *A projection expression* is a string that identifies the (single/multiple) attributes you want. To retrieve a single attribute specify the name. For multiple attributes, the names must be comma-separated. 
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
 * Set smaller page size to reduce read load
 * Encryption of Global Secondary Indexes is possible via Table Keys + AWS Owned CMK (Free) or AWS Managed CMK (£).
 * Strongly consistent reads dont use DAX (passes through).
 * To measure total capacity a query uses, set ReturnConsumedCapacity=TOTAL in the Query request.
  
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
 * Object level logging. API activity versus CloudTrail costs.
 * S3 Bucket Policy: Principal=User,  Resource=ARN.
 * SSE-S3 - AES256, Key rotation.
 * SSE-KMS - KMS Service + £ + CloudTrail Audit of key use.
 * SSE-C - Client provides key.
 * Client side encryption - encrypt before upload.
 * To enforce encryption, in PUT request, have a bucket policy looking for the following header: x-amz-server-side-encryption (xasse)
   x-amz-server-side-encryption: AES256
   x-amz-server-side-encrpytion: ams:kms
 * CORS - Cross Origin Resouce sharing. Safely allow resource sharing by naming buckets which allow HTTP referer.
 * 0 -> 5TB objects. 
 * 5GB is the max PUT size.
 * AWS S3 Inventory - tool for returning a list of objects in buckets (503s start getting thrown due to multiple versioning).
 * AllowedMethod (GET/PUT/POST/DELETE/HEAD)
 * AllowedHeader ( xasse )
 

## RDS 
 * ETL are better suited to RDBMS
 * Complex Join Operations are better suited to RDBMS than NoSQL/DynamoDB
 * Automated backups must be turned on for Read Replicas.
 * Snapshots are user initiated.
 * Encryption at rest provided via KMS. Needs to be enabled at creation time.
 * Automated backups between 1 and 35 days. Transaction logs are stored, so recovery point objective is a second. 

## CloudFront

 * To prevent S3 bucket being accessed directly, then CF Restrict bucket access via Origin Acess Identity then update bucket policy. This creates a user in IAM which allows CF to access bucket.
 * Signed URLs/Signed cookies - this allows individual URLS for members only/paid content.
 * Default expiry of pre-signed URL=3600s
 * Then remove public access from bucket.
 * S3 used to have problems storing large amounts of files with sequential/alphabetical names as this was used to determine the storage partition. This is no longer the case, but it used to be recommended to hash the files and then name them hash-filename.
 * Viewer Protocol Policy (VPP) provides HTTP->HTTPS redirection

## AWS Lambda.

 * Default runtime of 3s + 128MB
 * Maximum runtime of 15m + 3GB
 * Billed on the number of requests and the duration
 * Node, Java, Python, C#, GoLang.
 * Ensure that only include libraries that are required.
 * Scales out not up automically.
 * Use AWS X-Ray to debug
 * Region independent.
 * Use a dead letter queue->SQS or SNS for failed invocations
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
   * CloudFormation
   * CloudWatch Logs
   * CloudWatch Events
   * CodeCommit
   * AWS Config
 * Lambda Inputs
   * Kinesis
   * Dynamo DB
   * SQS
 * Lambda authorizers are to control access to the functions
 * Use CloudWatch Events to trigger a scheduled Lamdba event. (cf cron)

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
 * To interact with DynamoDB directly, an *Integration Request* is created.
 * Each time you update an API which includes modifications of methods, integrations, authorizers and anything else other than stage settings, you must redploy the API.

## AWS Step Functions

 * Used for workflow dependencies (c.f. IFTTT). Retry and Reuse
 * Sequential steps - serial.
 * Branching steps - decision tree.
 * Parallel steps - Fan out.
 * For Application Integration, use step functions
 * Uses the concept of a state machine and allows serverless applications to be visualised. Each step can be triggered and tracked. Amazon state language. Start State/Final State.
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
 * _X_AMZN_TRACE_ID; AWS_XRAY_CONTEXT_MISSING; AWS_XRAY_DAEMON_ADDRESS

## KMS

 * Encrpytion keys are regional
 * Key material origin can be from KMS or External (customer provided)
 * CMK = Customer Master Key (plaintext). Attributes, alias, creation date, description and key state. It can never be exported.
 * API Calls

       aws kms encrypt --key-id=<> thing
       aws kms decrypt ....
       aws kms enable-key-rotation  # rotate keys
       aws kms re-encrypt # destroys plaintext

 * Envelope enctyption. Master Key---encrypts--->[ Envelope key/Data Key ]---encrypts--->[[Data]]. The Envelope key is used to decrypt the data.
 * GenerateDataKey (generates plaintext)
 * GenerateDataKeyWithoutPlaintext

## SQS
 * Message queue which is Pulled from
 * 256KB message
 * Standard Queue. Unlimited TPS, any order and duplicates.
 * FIFO queue. 300 TPS, sequential and unique.
 * Visibiity timeout: How long message is visible after read. 30 seconds default. 12 hour max.
 * Long polling: If queue is zero, hold open the polling app until long poll timeout exceeded. Reduce CPU on polling requestor->saves money.

## SNS
 * Message queue which pushes.
 * Pub/sub -> Topic subscription
 * Fan out
 * PAYG

## SES
 * Simple Email services
 * Marketing mails outbound 
 * Incoming mail can be stored in S3.

## Kinesis
 * Used for streaming data (video/data)
 * Kinesis streams Producers->Kinesis streams/shards->Consumers
 * Kinesis Firehose for data only
 * Kinesis Analytics
 * Server-Side Encryption possible for Kinesis Streams
 * 1 shard, 5 TPS, up to 2MB/s reads. Number of shards depends on incoming write bandwidth and outgoing read bandwidth.
 * 1000 Records/Second can be written at up to 1MB/s
 * Firehose - no sharding or streams. Scales automatically. Populates S3. There is no retention window. Data is either analysed or sent to Redshift/S3/Elasticsearch
 * You cannot guarantee the order of data except within a shard, not across shards.
 * Kinesis analytics - SQL queries on Firehose or Streams.

## Elastic Beanstalk
 * No need to worry about provisioning infrastructure, deploy code. Autoscaling and RDS.
 * Code stored in S3. 
 * Can use custom AMIs for the underlying instances if large number of patches needed.
 * Be careful with RDS, since database is tied to deployment. Database deletion!
 There is an option in EBS Retention->Create Snapshot
 * To deploy separately, deploy and add security group to autoscaling group + pass connection string
 * ELB Deployment policies:
   * All at once - Outage
   * Rolling
   * Rolling with additional batches (partial number new servers deployed)
   * Immutable (new servers deployed)
 * Can be customised using yaml/json files. (EBEB)


        .ebextensions/healthcheck.config
        code/foo.py
        config/bar.ini

 * ElasticBeanstalk - can use Packer to build custom ELB.
 * Use Application Lifecycles to stop too many versions existing. You can end up hitting the application version limit if not.
 * cron.yaml for periodic jobs
 * Can deploy docker containers.

## Systems Manager Parameter store
 * Anything needed by EC2, Lambda credentials can be stored in Systems Manager Parameter Store. 
 * Name, Description, Value (string), String list, Secure string. Secure KMS string

## CI/CD
 * CI continuous integration. Building + Testing + Integrating code from disparate sources.
 * CI Workflow: CodeRepo->BuildManagementSystem->AutomatedBuild->Test Framework->Unit/Functional/Integration tests->Deploy packaged apps->Environment
 * CD Contiuous Delivery- push button to deploy code after test framework
      Continuous Deployment - automate the button push based on the tests.a
 * CodeCommit - AWS Gitlab
 * CodeBuild - 

### Code Deploy
  Code Deploy is an automated deployment tool, which can use Jenkins, Github, pipeline, ansible, puppet, chef. Applies to EC2,lambda or on-premise. 
 * Deployment:
     * In-place.
     * Blue/Green. Route traffic through new ELB change.
     * Rolling update. Update one backend at a time.
     * Deployment Group
 * Lambda Appspec file, must be in root directory special file which tells codedeploy what to execute. yaml/json. Contains parameters for a deploment. Made up of Version, Resources, Hooks and Permissions. (CDAS) Hooks:
     * BeforeAllowTraffc
     * AfterAllowTraffic
 * EC2 Appspec yaml. Version, OS, Files, Hooks. appspec.yml must be in root directory

        appspec.yml
        code/foo.py
        config/bar.ini
 
* EC2 Appspec Hooks:
     * BeforeBlockTraffic
     * BlockTraffic
     * AfterBlockTraffic
     * ApplicationStop
     * DownloadBundle
     * BeforeIntall
     * Install
     * AfterInstall
     * ApplicationStart
     * ValidateService
     * BeforeAllowTraffic  (LB is registered below here)
     * AllowTraffic
     * AfterAllowTraffic
   
 * Revision consists of manifesto, appspec file, app files, executables, config which are placed into a bundle
 * Applications have a unique id for Revision, deployent config and deployment group.
 * In-place upgrade does not work for lambda.
 * Can be deployed on EC2 to bundle code (codedeploy-agent);
 
      cd ~/webapp
      aws deploy create-application --application-name mywebapp
      aws deploy push --application-name mywebapp --s3-locaiton s3://mybucket/webapp.zip --ignore-hidden-files

### CodeBuild

 * Override build commands in codebuild either via buildspec.yml or buildSpecOverride.
 * Lambda version is defined in appspec.yml
 * Can provide additional VPC-specific configuration information as part of the CodeBuild project (connect to RDS in a private subnet)

### Code Pipeline
 
 * Pulls together CodeCommit, CodeBuild, CodeDeploy, lamba, ELB, CloudFormation, Elastic Container Service, Github and Jenkins.
 * Can be automatically integrated with Cloudwatch to look for a trigger (S3 bucket change)
 * To run X-account, define CMK in KMS, add to pipeline, and add cross account role.

     
### Docker and CodeBuild
 
 * Codebuild pulls from CodeCommit and can deploy to ECS.

     docker build -t myrepo
     docker tag mydockerrepo:latest <tag>
     docker push  # ECS Registry aka ECR

 * ECS has host and destination portmapping.
 * As an alternative, a buildspec.yml can be used and let codebuild do the heavy lifting.
 * As a second alternative, the build commands can be put inside CodeBuild directly (c.f EC2 userdata).
 * Full CodeBuild Log is in CloudWatch.
 * There is an option to rollback on failure.
 
## Serverless Application Model (SAM)

 * This an extension to CloudFormation to define serverless applications and provision in CloudFormation

      sam package # build and upload to S3
      sam deploy # deploy from S3 to CF

## CloudFormation 

 * NestedStacks is possible through Resources/TemplateURL 
 * Transforms - reuse code in S3, specify SAM
 
       Transform:
        Name:AWS:Include
        Parameters: Location: s3://bucketb/myfile.yml
 * Fn:<something>
 * Ref: 
 * CloudFormation StackSets extends the functionality of stacks by allowing crud of stacks X-account and X-region. (c.f nested stacks within an account)

## Web Identity Federation

 * Use Amazon, Facebook, Google to provide temporary access to AWS resources via Cognito.
 * User pool relats to the sign-up/sign in to the Application
 * Identify pool maps the signed-in users to the temporary controlled access to AWS services in your account.
 * Cognito is an identity broker handling all interactions with Web Identiy Providers
 * Push synchornisation used to send silent push of user data updates to muliple devices associated with user.
 * Is associated via IAM
 * 1. User <-> User pool <-> Google
   2. User <-> identity pool
   3. User <-> s3 access.
 * User pool relats to the sign-up/sign in to the Application
 * Identify pool maps the signed-in users to the temporary controlled access to AWS services in your account.
 * Cognito streams contains the Cognito data.
 * AssumeRoleWithWebIdentiy - Cognito
 * AssumeRoleWithSAML - SAML authentication response.
 * Can check for compromised credentials in the Advanced Security page.
 
## AWS OpsWorks.

 * Chef
 * Puppet

## AWS Lamdba Edge

 * Customise content which CloudFront delivers.

## Elastic Container Service

 * Use security groups /VPCs to separate containers belonging to different projects..

## Elasticache

 * Cache Parameter Groups set size and memory of the instances.
