
# AWS Developer Certification Notes

Table of Contets  
[1. IAM + Security](#iam--security)  
[2. EC2 + ENI](#ec2--eni)  
[3. ELB](#elastic-load-balancers)  
[4. ASG](#auto-scaling-group)  
3. ECS2 Storage:  EBS & EFS  
4. RDS + Aurora + ElastiCache  
5. Route 53  
6. VPC Fundamentals  
7. Amazon S3 Introduction  
8. loreipsum  
9. loreipsum  

### IAM + Security

#### AWS Regions
AWS has Regions all around the world. -> *eu-west-1*  
Each Region has many availability zones -> *eu-west-1*__(a,b,c,..)__  
Each availability zone has one or more discrete data centers with redundant power, networking, and connectivity  
#### IAM 
Users -> For physical persons  
Groups -> Permit group Users per Functions or Teams applying permissions to groups  
Roles -> For machines  
Policies -> JSON documents that defines what Users/Groups/Roles actions can/cannot do  

#### IAM Federation
Integrate corporate repository of users with AWS IAM.
Let login to AWS using their company credentials
Identity Federation uses the SAML standard (Active Directory)


#### Introduction to Security Groups
Security Groups -> the basic network security firewall in AWS
Regulate:
* Access to Ports
* Authorised IP Ranges
* Control Inboud network
* Control Outbound network

Security Groups are **stateful** - If u can send a request, It will always get the response traffic regardless the security group rules applied on the response network.

**TODO _Insert Security Group Picture+ EC2 Machine_**

Security Groups knowledge:
* Can be attached to multiple instances.  
* Are Locked down to a region /VPC combination
* All inbound traffic is **blocked** by default
* All outbound traffic is authorised by default
#### Public IP , Priavte IP & Elastic IP
* Public IP:
    *  Must be unique across the whole internet
    *  All the company devices are behind the Public IP.
* Private IP:
    * IPs which you can't acces internet.
    * Diferents networks can have the same IP because only works at local level.
* Elastic IP:
    * Can bind the instance to an Elastic IP to have always the same IP.
---
### EC2 + ENI
EC2 is Elastic Compute Cloud
Mainly consist in:
* Renting Virtual machines
* Storing data on virtual drives
* Distributing load across machines
* Scaling the services using auto-scaling group
#### SSH to EC2
|               | SSH           | Putty  | EC2 Instance Connect |
| :------------:|:-------------:|:-----:|:-------------:|
| Mac/Linux*      | OK |  |OK|
| Windows      |       |   OK |OK|
| Windows 10 | OK      |    OK |OK|  

*Mac/Linux must remember to restrict your .pem key using chmod 400

#### EC2 User Data
Specified camp from building Instances that let u write bootstrap commands.
* The script only run once at the instrance first start.
* EC2 user data is used to automate boot tasks.
* The EC2 User Data Script runs at root level
* Remember to #!/bin/bash

#### EC2 Instances Launch Types
There are On demand Instances, Reserved, Spot Instances & Dedicated:
* On demand instances -> Short Workload and predictable pricing
* Reserved: (_Minum 1 Year_)
    * Reserved Instances:
        * 75% discount comparet to on-demand
        * Reservation preiod 1 or 3 years
        * Reserve specific instance type
    * Convertible Reserved Instances: Long workloads with flex instance
        * Can change the EC2 instance Type
        * Up to 54% discount
    * Scheduled Reserved Instances
        * Launch withing the interval windows you reserve
        * When u requiere fraction of day/week/month
* Spot instances:
    * Can get 90% compared to On-demand
    * The instance will shutdown if your max price is less than the current spot price.
    * Cost-efficient instances in AWS
* Dedicated Host:
    * Physical dedicated EC2 server
    * Full control of EC2 Instance
    * 3 Years period reservation
    * Really expensive
    * Useful for software that have complicated licensing model (BYOL)
* Dedicated Instance:
    * Instance running on a physical machine that's dedicated to you
    * The instance can share the physical machine with other instance from the same Account.
    * Other Accounts will never use their instances on the same machine as yours. 
#### Elastic Network Interfaces (ENI)
Represents a Virtual Network Card like Virtual Networks Adapters
ENI attributes:
* Primary private IPv4
* One or more secondary IPv4
* One Elastic IPv4 x private IPv4
* Attached to one or more security groups
* MAC Addres

Useful for failover to EC2 instances.

---

### Elastic Load Balancers
Elastic Load balance wants to powerful the AWS high availability characteristic.
Load Balancer functions:
* Spread load across configured instances
* Expose a single DNS to your app
* Check the health of ur instances
* Frontend SSL for your websites
* Separate public traffic from private

AWS have 3 types of load balancers:
* Classic Load Balancer
    * Supports TCP _Layer 4_, HTTP & HTTPS _Layer 7_
    * Link to Security Group, Config HealthChecks and Add EC2 Instances
* Application Load Balancer
    * HTTP and HTTPS _Layer 7_
    * LB to multiple http applications across target groups
    * LB to multiple applications oriented to containers 
    * HTTP/2 and WebSocket
    * Routing tables to different target groups:
        * **Path URL**  example[.]com/_users_ example[.]com/_admins_
        * **Hostname in URL** _mail_.example.com _print_.example.com 
        * **Query String** example.com/id=?_12345_
    * Target Groups **_Check_**
        * IP must be private
    * Best Practices
        * Fixed Hostname XXX.region.elb.amazonaws.com
        * The application server don't see the IP from the client.
        * IP Client -> X-Forwarded-For
* Network Load Balancer
    * Forward TPC/UDP to your instrances
    * Millions of request per seconds
    * Less latency than ALB
    * One static IP per AZ
    * NLB are used for extreme perfonance

Load Balancer Attributes
* Load Balancer Stickiness
    * The same client is always redirected to the same instance behind a load balance.
    * CLB & ALB
* Cross-Zone Load Balancing
    * Each load balancer instance distributes evenly across all registered instances in all AZ
    * CLB 
        * Disabled by default
        * No charges for inter AZ data
    * ALB
        * Always on
        * No charges for inter AZ
    * NLB
        * Disabled by default
        * Pay charges for inter AZ data
* SSL/TLS
    * Allows traffic between clients and your load balancer to be encrypted in transit.
    * Public SSL certificates are issued by Certificate Authorities (CA)
    * Certifiactes using ACM or ur own certificates
    * HTTPS listener:
        * Specify a default certificate
        * Add optional list of certs to supp multiple domains
        * Clients can use **SNI** (Server Name Indicator) to specify the hostname they reach.
    * Server Name Indicator
        * Solvers the problem of loading multiple SSL certificates onto one web server.
        * Newer protocol which requires the client to indicate the hostname of the target server in the initial SSL handshake
        * The server will then find the correct certificate or return the default one.
        * Only works for ALB & NLB, CloudFront.
        * Does not work for CLB.
    * ELB Connection Draining
        * CLB: Connection Draining
        * Target Group: Deregistration Delay (For ALB & NLB)
        * Time to complete "in-flight requests" while the instance is de-registering or unhealthy
    
### Auto Scaling Group
Before start, we have to underestand the main concepts of auto scaling:
* Vertical Scalability:
    * Increase the size of the instance
    * Vertical scalability common for non distributed systems like databases.
    * RDS and ElastiCache typical scalables
* Horizontal Scalability:
    * Increase the number of instances/systems
    * Common on distributed systems.
    * Web application/microservices applicatons
* Goal of the ASG:
    * Scale out to match an increased load (_+ instances_)
    * Scale in to match a decreased load (_- instances_)
    * Define min and max num of machines running.
* ASG steps to config
    * Conf. panel
        * AMI + Instance Type
        * EC2 User Data
        * EBS Volumes
        * Security Groups
        * SSH Key Pair
    * Min/Max size Inital Capacity
    * Network+ Subnets Information
    * Load Balancer Information
    * Scaling Policies

* ASG ALARMS
    * Scale ASG with CloudWatch Alarms
    * Alarm monitors basic metrics like CPU Usage, RAM Usage...
    * We can create scale-in/out policies:
        * Target Tracking Scaling: always want CPU at 40%
        * Simple/step Scaling: CPU > 75% add 1 unit
        * Scheduled Action: Increase 1 unit to 18 at 24 pm on Fridays



---
### Advanced S3
##### S3 MFA-Delete
Force MFA b4 important operations on S3
* To use MFA-Delete -> enable versioning on the S3 Bucket
* Need MFA:
    * Permanent delete an object version
    * Suspend versioning on the bucket
* No need MFA:
    * enable versioning
    * listing deleted versions
* Onle the bucket owner **(root account)** can enable/disable MFA-Delete
* MFA-Delete can only be enabled using the CLI  

##### S3 Default Encruption
To use traffic encrypted on S3 u have the default encryption option
Bueckt policies are evaluated before "Default encryption"
##### S3 Access Logs
For audit purpose. Any request to a s3 will be logged into another s3 bucket.
##### S3 Replication (CRR & SRR)
Must enable versioning in source and destination
Cross Region Replication (CRR)
Same Region Replication (SRR)
* Bucket can be in different acc
* Copying is asynchronous
* Must give proper IAM permissions to S3
* Only new objects are replicated

Delete operations:
* if u delete withour a version ID, it adds a delete marker, not replicated
* if u delete with a version ID, it deletes in the source, not replicated

No chaining replication
* A->B
* B->C
* A content never arrives to C.

##### S3 pre-signed URLs
Can generate pre-signed URLs using SDK or CLI
* For downloads
* For uploads
* Default time of 3600 Seconds
* Users given a pre-signed URL inherit the permission of the person who generated the URL for GET/PUT

##### S3 Storage Classes
* Amazon S3 Standard - General Purpose
    * High durability 99,9p9% across multiple AZ
    * Availability 99.99%
    * sustain 2 concurrent facility failures
    * Use:Big data analytics, mobile & gaming applications, content distribution
* Amazon S3 Standard - Infrequent Access (IA)
    * Suitable for data that is les frequently accessed.
    * High durability 99,9p9% across multiple AZ
    * Availability 99.9%
    * Low cost compared to amazon S3 Standard
    * Sustain 2 concurrent facility failures
    * Use:Disaster recovery, backups
* Amazon S3 One Zone-Infrequent Access
    * Same as IA but single AZ
     * High durability 99,9p9% single AZ
     * 99.5% Availabilitty
     * low latency and high throughput performance
     * Supports SSL for data at transit and encryption at rest
     * Low Cost compared to IA
     * Use: Secondary backup copies, storing data you can recreate.    
* Amazon S3 Intelligent Tiering
    * Low latency and high performance of S3 standard
    * Small monthly monitoring and auto-tiering fee
    * Automatically moves objects between two access tiers based on access paterns
    * Designed for durability 99,9p9 across multiple Availability Zones
    * Designed for 99.9% availability 
* Amazon S3 Glacier
    * Low cost object storage for archiving/backup
    * Data is retained for the longer term _10 years_
    * Alternative to on-premise magnetic tape storage
    * Average annual durability 99.9p9
    * Cost per storage per month + retrieval cost
    * Each item in Glacier is called "Archive" _up to 40TB_
    * Archives are stored in "Vaults"
    * Retrieval options (_Minum storage duration **90 days**_):
        * Expedited (_1-5 minutes_)
        * Standard (_3-5 hours_)
        * Bulk (_5 to 12 hours_)

    
* Amazon S3 Glacier Deep Archive (Minum 180 days - cheaper)
    * Standard (_12 hours_)
    * Bulk (_48 hours_)

##### S3 Moving between storage classes
You can transition S3 storage classes to anothers:  
Infrequently accessed object -> Standard_IA  
For archieve objects -> Glacier or Deep_Archive  
Lifecycle Rules:
* Transition actions - When objects are transitioned to another storage class
    * Move objects to Standard IA class 60 days after creation
    * Move to gGlacier for archiving after 6 months
* Expiration actions - Configure objects to expire (_delete_) after some time
    * Acess log files can be set to delete after a 365 days
    * Can be used to delete old version of files (if versioning is enabled)
    * Can be used to delete incomplete multi-part upload
* Rules can be created for a certain prefix (ex  s3://mybucket/mp3/*)
* Rules can be created for certain objects tags (ex - Department: Finance)

##### S3 Performance
Amazon S3 automatically scales to gith request rates.
* Performance:
    * Latency 100-200ms
    * 3500 PUT/COPY/POST/DELETE req x sec x prefix in bucket
    * 5500 GET/HEAD req x sec x prefix in bucket
    * There are no limits to the number of prefixes in a bucket.
* KMS Limitation
    * Upload -> Call GenerateDataKey KMS API
    * Download -> Call Decrypt KMS API
    * Count towards the KMS quota per second (_5500, 10000, 30000 req/s_)
    * Can not increase request a quiota increase for KMS
* Multi-Part upload:
    * Recommended for files > 100MB
    * Must for > 5GB fles
    * Parallelize uploads
* S3 Transfer Acceleration (_Upload only_)
    * Increase transfer speed by transferring file to an aws edge location which will forward the data to the S3 bucket in the target regions
    * Compatible with multi-part upload

##### S3 Event notifications
* SNS -> Mail
* SQS -> Queue
* Lambda Function -> Execute Code

##### AWS Athena
* Serverless service to perform analytics againts S3 files
* Uses SQL language to query the files
* Has a JDBC/ODBC dirver
* Charged per query and amount of data scanned
* Supports CSV,JSON, ORC, Avro, and Parquet
* Use: Business intelligence/Analytics/reporting/VPC Flow Logs, ELB Logs, CloudTrail trails...
##### S3 Object Lock & Glacier Vault Lock
* S3 Object Lock
    * Adopt a WORM model
    * Block an object version deletion for a specified amount of time
* Glacier Vault Lock
    * Adopt a Worm model
    * Lock the policy for future edits
    * Helpful for compliance and data retention

### AWS CloudFront
Content Delivery Network (CDN)
* Improves read performances -> content is cached at the edge
* 216 Poin of Presence globally
* DDoS protection, integration with Shiedl, AWS Web Application Firewall
* Can expose external HTTPS and can talk to internal HTTPS Backends
##### S3 bucket
 * For distributing files and caching them at the edge
 * Enhanced security with CloudFron Origin Access Identity (OAI)
 * CloudFront can be used as an ingress (to upload files to S3)
##### Custom Origin (HTTP)
* Application Load Balancer
* EC2 instance
* S3 Website (must first enable the bucket as a static S3 website)
* Any HTTP Backend you want
##### CloudFront vs S3 Cross Region Replication
* CloudFront:
    * Global Edge network
    * Files are cached for a TTL
    * Great for static content that must be available everywhere
* S3 Cross Region Replication:
    * Must be setup for each region you want replication to happen
    * Files are updated in near real-time
##### CloudFront Caching
* Cache based on
    * Headers
    * Session Cookies
    * Query String Parameters
* The cache lives at each CloudFront Edge Location
* You want to maximize the cache hit rate
* Control the TTL
* You can invalidate part of the cache using the CreateInvalidation API
* Invalidating objects removes them from CloudFront edge caches (Interesting)
##### CloudFront Signed URL
* You want to distribute paid shared content to premium users over the world
* We can use CloudFront Signed URL/Cookie:
    * Includes URL expiration
    * Includes IP ranges to access the data from
    * Trusted signers (which aws accounts can create signed URLs)
* How long should the URL be valid for?
    * Shared content (movie,music): make it short
    * Private content (private to the user): u can make it last for yearys
* Signed URL **=>** acess to individual files (one signed URL per file)
* Signed Cookies **=>** access to multiple files (one signed cookie for many files)
##### CloudFront Signed URL vs S3 Pre-Signed URL
* CloudFront Signed URL:
    * Allow access to a path. no matter the origin.
    * Account wide key-pair, only the root can manage it
    * can filter by IP, path, date, expiration
    * Can leverage caching features
* S3 Pre-Signed URL:
    * Issue a request as the person who pre-signed the URL
    * Uses the IAM key of the signing IAM principal
    * Limited lifetime

### ECs, ECR & Fargate
###### Docker
All we know what is docker.
* Public -> Docker hub
* Private -> Amaozn ECR (Elastic Container Registry)
    * Access is controlled through IAM
    * AWS CLI v1 login command -> **$**(aws ecr get-login --no-include-email --region eu-west-1)
    * AWS CLI v2 login command aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin 123456789.dkr.ecr.eu-west-1.amazonaws.com
    * Docker Push 1234567890.dkr.ecr.eu-west-1.amazonaws.com/demo:latest
    * Docker Pull 1234567890.dkr.ecr.eu-west-1.amazonaws.com/demo:latest
Resources are shard with the host
##### ECS Cluster Overview
ECS -> Elastic Containter Service
* ECS CLuster are logical grouping of EC2 Instances
* EC2 instances run the ECS agent
* ECS agents register the instance to the ECS cluster
* EC2 instances run a special AMI made for ECS.
Like kubernetes

##### Practice
Important file /etc/ecs/ecs.config
##### ECS Task Definitions
Task definitions are metadata in JSON. Tells ECS how to run a Docker Container.
* Contains:
    * Image Name
    * Port Binding for Container and Host
    * Memory and CPU required
    * Environment variables
    * Networking information
##### ECS Services
ECS Services help define how many task should run and how they should be run.
* Ensure that the number of tasks desired is runing across our fleet of EC2 instances
* Can be linked to ELB / NLB / ALB if needed
##### Fargate
Like ECS but is serverless.
Amazon do it all
You only have to configure the container. (Like Cloud but with containers)
##### ECS IAM Roles Deep Dive
* EC2 Instance Profile:
    * Used by the ECS agent
    * Makes API calls to ECS service
    * Send container Logs to CloudWatch Logs
    * Pull Docker image from ECR
* ECS Task Role:
    * Allow each task to have a specific role
    * Use different roles for the different ECS Services you run
    * Task Role is defined in the task definition
    
##### ECS Task Placement
* Helpful to determine where the task of type EC2 will be placed, the CPU, memory and available port.
* When a service scales in, ECS need to determine which task to terminate.
* Task placement strategies are best effort
* Process:
  + Identify the instances that satisfy the CPU, memor and port req.
  + Identify the instances that satisfy the task placement constraints
  + Identify the isntances that satisfy the task placement strategies
  + Select the instances for task placement
##### ECS Task Placement Strategies
U can mix this strategies on the same placementStrategy
* Binpack
  * Place tasks based on the least available amount of CPU or memory
  * Minimizes the number of instances in use (cost saving)
* Random
  * Place the task randomly
* Spread
  * Task will be spread based on the specified value
  * Examples: InstanceID, attribute:ecs:availability-zone ...
##### ECS Task Placement Constraints
* distinctInstance
  * Place each task on a different container instance
* memberOf
  * Place task on instances that satisfy an expresion
  * Use the Cluster Query Language
##### ECS Service Auto Scaling
CPU and RAM is tracked in CloudWatch at the ECS service level
* **Target Tracking** 
  * target a specific average CloudWatch metric.
* **Step Scaling**
  *  scale based on CloudWatch alarms
* **Scheduled Scaling** 
  * based on predictable changes

ECS Service Scaling (task level) **!=** EC2 Auto Scaling (Instance level)  
Fargate Auto Scaling is much easier to setup (Serverless)
##### ECS Cluster Capacity Provider
Used in association with a cluster to determine the incrastructure that a task runs on
* For ECS and Fargate users, the FARGATE and FARGATE_SPOT capcity providers are added automatically
* For Amazon ECS on EC2, you need to associate the capacity provider with a auto-scaling group  

When you run a task or a service, you define a capacity provider strategy, to prioritize in which provider to run.
This allows the capacity provider to automatically provision infrastructure for you

##### ECS Summary
ECS is used to run Docker containers and has 3 flavors:
* ECS "Classic"
  *  Provision EC2 instances to run containers onto
  + EC2 instances must be created
  + We must configure the file /etc/ecs/ecs.config with the cluster name
  + EC2 instance must run an ECS agent to connect to cluster
  + EC2 instances can run multiple containers on the same type
    + You must **not** specify a host port
    + You should use an Application Load Balancer with the dynamic port mapping
    + The EC2 instance security group must allow traffic from the ALB on all port
  + ECS tasks can have IAM Roles to execute action against AWS
  + Security groups operate at the instance level, not task level
*  Fargate
   * ECS Serverless, no more EC2 to provision
   * AWS provisions containers for us and assigns them ENI
   * Fargate containers are provisioned by the container spec (CPU/RAM)
   * Fargate tasks can have IAM Roles to execute actions agains AWS
* EKS
  *  Managed Kubernetes by AWS  
  
ECR is used to store Docker Images
 * ECR is tightly integrated with IAM
 * AWS CLI V1 login command
   * $(aws ecr get-login --no-include-email --region eu-west-1)
   * "aws ecr get-login" generates a "docker login" command
  * AWS CLI v2 Login command (newer, may also be asked at the exam -pipe)
    * aws ecr get-login.password --region eu-west-1 | docker login --username AWS password-stdin 1234567890.dkr.ecr.eu-west-1.amazonaws.com
  * Docker Push & Pull:
    * docker push 1234567890.dkr.ecr.eu-west-1.amazonaws.com/demo:latest
    * docker pull 1234567890.dkr.ecr.eu-west-1.amazonaws.com/demo:latest
* ECS does integrate with cloudWatch Logs:
  * You need to setup loggin at the task definition levle
  * Each container will have a different log stream
  * The EC2 Instance Profile need to have the correct IAM permissions
* Use IAM Task Roles for your tasks
* Task Placement Strategies: binpack, random, spread
* Service Auto Scaling with target tracking, step scaling or scheduled
* Cluster Auto Scaling through Capacity Providers
### AWS Elastic Beanstalk
##### Overview
* Elastic Beanstalk is a developer centric view of deploying an application on AWS
* Uses all the component's we've seen before: EC2,ASG,ELB,RDS, etc...
* Still have full control over the configuration
* Beanstalk is free but you pay for the underlying instances
##### Deep
* Managed service
  * Instance configuration / OS is handled by Beanstalk
  * Deployment strategy is configurable but performed by Elastic Beanstalk
* The application code is the responsibility of the developer
* Three architecture models:
  * **Single Instance deployment** -> good for dev
  * **LB + ASG** -> great for production or pre-production web applications
  * **ASG only** -> non-web apps (Workers or queue)
##### Components
* Elastic Beanstalk has three components
  * Application
  * Application version: each deployment gets assigned a version
  * Environment name (dev,test,prod...)
* You deploy application versions to environments and can promote application versions to the next environment
* Rollback feature to previous application version
* Full control over lifecycle of environments
#####  Beanstalk Deployment Options for Updates
* **All at once (deploy all in one go)** - fastest, but instances aren't available to serve traffic for a bit (downtime)
  * shutdown all instance and rise the news
  * Fastest deployment
  * Application has downtime
  * Great for quick iterations in development environment
  * No additional cost
* **Rolling**: update a few isntances at a time (bucket), and then move onto the next bucket one the fist bucket is healthy (_update slowly_)
  * Application is running below capacity
  * Can set the bucket size.
  * _Shut down half of ur system cause u are releasing the new verison and want to keep working your system with the rest of the old instances_
* **Rolling with addition batches** like rolling, but spins up new instances to move the batch (update minimal cost and maitaning full capacity)
  * Application is running at capacity
  * Can set the bucket size
  * Application is running both versions simultaneously
  * Small additional cost
  * Addiotional batch is removed at the end of the deployment
  * Longer deployment
  * Great for prod

* **Immutable**: spins up new isntances in a new ASG, deploys versions to these instances and then swaps all the isntances when everything is healthy (*Rollback inmediatly*)
  * Zero downtime
  * New Code is deployed to new isntances on a temporary ASG
  * High cost, double capacity
  * longest deployment
  * Quick rollback in case of failures
  * Great for prod
* **Blue / Green**
  * Not a "direct feature" of Elastic Beanstalk
  * Zero downtime and release facility
  * Create a new "stage" environment and deploy v2 there
  * The new environment (green) can be validated independently and roll back if issues
  * Route 53 can be setup using weighted policies to redirect a little bit of traffic to the stage environment
  * Using Beanstalk, "swap URLs" when done with the environment test
 _**Faltaria ficar les fotos**_ 
---
##### Beanstalk Lifecycle Policy
* Elastic Beanstalk can store at most 1000 aplications versions.
* Have to remove old versions to deploy more.
* To phase out old applications versions use lifecycle policy
  * Based on time
  * Based on space
* Versions that are currently used won't be deleted
* Option not to delete the source bundle in S3 to prevent data loss

##### Elastic Beanstalk Extensions
* A zip file containing our code must be deployed to Elastic Beanstalk
* All the parameters set in the UI can be configured with code using files
* Requirements
  * In the .ebextensions/ directory in the root of source code
  * YAML / JSON format
  * .config  extensions (example: loggin.config)
  * Able to modify some default settings using: option_settings
  * Ability to add resources such as RDS, ElastiCache, Dynamo DB, etc...
* Resources managed by .ebextensions get deleted if the environment goes away

##### Elastic Beanstalk Under the Hood
Under the hood, Elastic Beanstalk relies on CloudFormation
CloudFormation is used to provision other AWS services
You can define CloudFormation resources in your _.ebextensions_ to provision ElastiCache, S3 bucket...
##### Elastic Beanstalk Clonning
Clone an environment with the exact same configuration
Useful for deploying a "test" version of your applicaiton
* All resources and configuration are preserved:
  * Load Balancer type and configuration
  * RDS database type (but the data is not preserved)
  * Environment variables
##### Elastic Beanstalk Migration: Load Balancer
  After creating an Elastic Beanstalk environment, you cannot change the Elastic Load Balancer Type
  You have to migrate:
    * Create a new environment with the same configuration except the Load Balancer
    * Deploy your applicaiton onto the new environment
    * Perform a CNAME swap or Route 53 Update

##### Elastic Beanstalk - Single Docker
Run your application as a single docker container
Ether provide:
  * Dockerfile: Elastic Beanstalk will build and run the Docker container
  * Dockerrun.awsjson (v1): Describe where *already built* Docker image is:
    * image
    * ports
    * volumes
    * Loggin
  * Beanstalk in Single Docker Container **does not use ECS**
##### Elastic Beanstalk - Multi Docker Container
* Multi Docker helps run multiple containers per EC2 instance in EB
* Components:
  * ECS Cluster
  * EC2 instances, configured to use the ECS Cluster
  * Load Balancer (in high availability mode)
  * Task definitions and executions
* Requires a config Dockerrun.aws.json (v2) at the root of source code
* Dockerrun.aws.json is used to generate the ECS task definition
* Your Docker image must be pre-build and stored in ECR x exemple.
##### Elastic Beanstalk and HTTPS
* Beanstalk with HTTPS
  * _Load the SSL Certificate onto the LoadBalancer_
    * Can be done from the Console (EB console, load balancer configuration)
    * Can be done from the code: .ebextensions/securelistener-alb.config
    * SSL Certificate can be provisioned using ACM (AWS Certificate Manager) or CLI
    * Must Configure a security group rule to allow incoming port 443 (HTTPS port)
* Beanstalk redirect HTTP to HTTPS
  * Configure your instances to redirect HTTP to HTTPS:
  * Or configure the application Load Balancer (ALB only) With rule
  * Make sure health checks are not redirected (so they keep giving 200 ok)
  ##### Elastic Beanstalk - Custom Platform (Advanced)
  * Custom Platforms are very advanced, they allow to define from scratch:
    * The Operating System (OS)
    * Additional Software
    * Scripts that Beanstalk runs on these platforms
  * Use case: app language is incompatible with Beanstalk & doesn't sue DOcker
  * To create your own platform:
    * Define an AMI using Platform.yaml file
    * Build that platform using the Pacjer Software (open source tool to create AMIs)
  * Custom Platform vs Custom Image (AMI):
    * Custom Image is to tweak an **existing** Beanstalk platform (Python, Node.js, Java)
    * Custom Platform is to create **an entirely new** Beanstalk Platform
### AWS CICD:
##### Technology Stack for CICD
|  Code | Build   | Test   | Deploy   |  Provision |
|---|---|---|---|---|
| AWS CodeCommit  | AWS CodeBuild  |AWS CodeBuild   | AWS Elastic Beanstalk  | AWS Elastic Beanstalk |
|   Github Or 3rd party code repository| Jenkins CI Or 3rd party CI Servers   |   Jenkins CI Or 3rd party CI Servers | AWS CodeDeploy  | User Managed EC2 Instances Fleet (CloudFOrmation)  |

All integrated with Orchestrate: **AWS CodePipeline**
#### CodeCommit
##### CodeCommit Overview
* **Version Control** -> ability to underestand the various changes that happened to the code over time.
* Enabled using a version control system such as Git
* Git repository can live on one's machine or central online repository
* Benefits are:
  * Collaborate with other dev
  * Backed-up code
  * Viewable and auditable

##### CodeCommit AWS
* Private Git repositories
* No size limit on repositories (scale seamlessly)
* Fully managed, highly available
* Code Only in AWS Cloud Account
* Secure (encrypted, access control, etc)
* Integrated with jenkins / CodeBuild / Other CI Tools

##### CodeCommit Security
* Interactions are done using Git
* Authentication in Git:
  * SSH Keys: AWS Users can configure SSH keys in their IAM Console
  * HTTPS: Done through the AWS CLI Authentication helper or Generating HTTPS credentials
  * MFA can be enabled for extra safety
* Authorization in Git:
  * IAM Policies manage user / roles rights to repositories
* Encryption:
  * Repositories are automatically encrypted at rest using KMS
  * Encrypted in transit (can only use HTTPS or SSH - both secure)
* Cross Account access:
  * Do not share your SSH keys
  * Do not share your AWS credentials
  * Use IAM Role in your AWS Account and use AWS STS (with AssumeRole API)

##### CodeCommit Notifications
You can trigger notifications in CodeCommit using **AWS SNS** (Simple Notification Servie) or **AWS Lambda** or **AWS CLoudWatch Event Rules**
* Use Case SNS /AWS Lambda:
  * Deletion of branches
  * Trigger for pushes in master branch
  * Notify external Build System
  * Trigger AWS Lambda function to perform codebase analysis
* Use cases for CLoudWatch Event Rules:
  * Trigger for pull request updates
  * Commit comment events
  * CloudWatch Event Rules goes into an SNS topic

#### CodePipeline
##### CodePipeline Overview
* Continuous delivery
* Visual workflow
* Source: GitHub / CodeCommit / Amazon S3
* Build: CodeBuild / Jenkins /etc ...
* Load Testing: 3 party tools
* Deploy: AWS CodeDeploy / Beanstalk / CloudFormation / ECS ...
* Made of stages:
  * Each stage can have squential actions and / or parallel actions
  * Stage examples: Build / Test / Deploy / Load Test / etc ...
  * Manual approval can be defined at any stage

##### CodePipeline Artifacts

Each pipeline stage can create "artifacts", artifacts are passed stored in Amazon S3 and passed on to the next stage.

_Each stage will output artifacts to the next stage_

##### CodePipeline Troubleshooting

CodePipeline state changes happen in **AWS CloudWatch Events**, which can in return create SNS notifications.
  * Create events for failed pipelines
  * Create events for cancelled stages  

If CodePipeline fails a stage, the pipeline will stop and u will get the info in the console

AWS CloudTrail can be used to audit AWS API calls
Pipeline have to be attached an IAM Service Role to do some actions

#### CodeBuild
##### CodeBuild Overview 
* Fully managed build service
* Alternative to other build tools like Jenkins
* Continuous scaling (no servers to manage or provision - no build queue)
* Pay for usage: the time it takes to complete the builds
* Leverages Docker under the hood for reproducible builds
* Possibility to extend capabilities leveraging our own base Docker images
* Secure: Integration with KMS for encryption of build artifacts, IAM for build permissions, and VPC for network security, CloudTrail for API calls loggin
##### CodeBuild Properties
* Source Code from GitHub / CodeCommit / CodePipeline / S3...
* Build instructions can be defined in code (buildspec.yml file)
* Output logs to Amazon S3 & AWS CloudWatch Logs
* Metrics to monitor CodeBUilds statistics
* Use CloudWatch Alarms to detect failed builds and trigger notifications
* CloudWatch Events / AWS Lambda as a Glue
* SNS notifications
* Ability to reproduce CodeBuild locally to troubleshoot in case of errors
* Builds can be defined within CodePipeline or CodeBuild itself
##### CodeBuild BuildSpec
* **buildspec.yml** file must be at the **root** of your code
* Define environment variables:
  * Plaintext variables
  * Secure secrets: use SSM Parameter store
* Phases (specify commands to run):
  * Install: install dependencies you may need for your build
  * Pre build: final commands to execute before build
  * **Build: Actual build commands**
  * Post build: finishing touches (zip output for example)
* Artifacts: What to upload to S3 (encrypted with KMS)
* Cache: Files to cache (usually dependencies) to S3 for future build speedup
##### CodeBuild Local Build
* In case of need of deep troubleshooting beyond logs...
* You can run CodeBuild locally on tour desktop (after installing Docker)
* For this, leverage the CodeBuild Agent
##### CodeBuild in VPC
* By default, your CodeBuild containers are launched outside your VPC
* Therefore, by default it cannot access resources in a VPC
* You can specify a VPC configuration:
  * VPC ID
  * Subnet IDs
  * Security Group IDs
* Then your build can access resources in your VPC (RDS, ElastiCache, EC2, ALB)
* Use cases: integration test, data query, internal load balancers
#### CodeDeploy
##### CodeDeploy overview
* We want to deploy our applicaiton automatically to many EC2 instances
* These isntances are not managed by Elastic Beanstalk
* There are several ways to handle deployments using open source tools (Ansible, Terraform, Chef, Puppet, etc...)
* We can use the managed Service AWS CodeDeploy
##### CodeDeploy Steps to make it Work
* Each EC2 Machine (or On Premise machine) must be running the **CodeDeploy Agent**
* CodeDeploy sends appspec.yml file
* Application is pulled from GitHub or S3
* EC2 will run the deployment instructions
* CodeDeploy Agent will report of success / failure of deployment on the instance
##### CodeDeploy Other
* EC2 instances are grouped by deployment group (dev / test / prod)
* Lots of flexibility to define any kind of deployments
* CodeDeploy can be chained into CodePipeline and use artifacts from there
* CodeDeploy can re-use existing setup tools, works with any application, auto scaling integraiton
* Note: Blue / Green only works with EC2 instances (not on premise)
* Support for AWS Lambda deployments (we'll see this later)
* CodeDeploy does not provision resources
##### CodeDeploy Primary Components
* **Aplication** -> unique name
* **Compute platform** ->  EC2/On-Premise or Lambda
* **Deployment configuration** ->  Deployment rules for success / failures
  * EC2/On-Premise: you can specify the minimum number of healthy instances for the deployment.
  * AWS Lambda specify how traffic is routed to your updated Lambda function version.
* **Deployment group** -> group of tagged instances (allows to deploy gradually)
* **Deployment Type** -> In-place deployment of Blue/green deployment
* **IAM instance profile** -> need to give EC2 the permissions to pull from S3 / GitHub
* **Application Revision** -> application code + appsec.yml file
* **Service role** -> Role for CodeDeploy to perform what it needs
* **Target revision** -> Target deployment application version
##### CodeDeploy AppSpec
* File section -> How to source and copy from S3 / GitHub to filesystem
* Hooks -> set of instructions to do to deploy the new verion (hooks can have timeouts)
* The order of hooks:
  * **ApplicationStop**
  * **DownloadBundle**
  * **BeforeInstall**
  * **AfterInstall**
  * **ApplicationStart**
  * **ValidateService** *really important*

##### CodeDeploy Deployment Config
* Configs:
  * On a time -> one instance at a time, if one instance fails the deployment stops
  * Half at a time 50%
  * All at once quick but no healthy host, downtime. *Good for dev*
  * Custom min healhty host 75%
* Failures:
  * Instances stay in "failed state"
  * New deployments will first be deployed to "failed state" instance
  * To rollback: redeploy old deployment or enable automated rollback for failures
* Deployment Targets:
  * Set of EC2 isntances with tags
  * Directly to an ASG
  * Mix of ASG / Tags so you can build deployment segments
  * Customization in scripts with DEPLOYMENT_GROUP_NAME environment variables
##### CodeDeploy to EC2
* Define how to deploy the application using appspec.yml + deployment strategy
* Will do in-place update to your fleet of EC2 isntances
* Can use hooks to verify the deployment after each deployment phase
##### CodeDeploy to ASG
* In place updates:
  * Updates current existing EC2 instances
  * Instances newly created by an ASG will aslo get automated deployments
* Blue/green deployment:
  * A new auto-scaling group is created (settings are copied)
  * Choose how long to keep the old instances
  ******Revisar dema millor******