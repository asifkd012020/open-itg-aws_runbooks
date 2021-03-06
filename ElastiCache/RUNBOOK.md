

# ElastiCache - Security Runbook <!-- omit in toc -->
## NIST Cybersecurity Framework Alignment <!-- omit in toc -->

**Generated By:**  
Ken Jeanis
 

## Table of Contents <!-- omit in toc -->
- [Disclaimer](#disclaimer)
- [Overview](#overview)
- [Preventative Controls](#preventative-controls)
  - [1. Permissions within the service are established in line with individual need and least-privilege is enforced 1](#1-permissions-within-the-service-are-established-in-line-with-individual-need-and-least-privilege-is-enforced)
  - [2. Enable Security Groups to limit access to ElastiCache instances](#2-enable-security-groups-to-limit-access-to-elasticache-instances)
  - [3. Data must be encrypted using CG BYOK for encyprtion](#3-data-must-be-encrypted-using-cg-byok-for-encyprtion)
- [Detective](#detective)
  - [1. Establish Config rules to monitor for deviations from normal configuration](#1-establish-config-rules-to-monitor-for-deviations-from-normal-configuration)
- [Respond/Recover](#respondrecover)
  - [1. Utilize Amazon EventBridge for automated incident response](#1-utilize-amazon-eventbridge-for-automated-incident-response)
- [Endnotes](#endnotes)

## Overview
AWS provides a number of security features for ElastiCache which help you comply with the NIST Cybersecurity Framework. The following runbook will outline what the AWS best practices are, how they align to NIST, and how to implement these best practices within your organization.

These NIST Controls and Subcategories are not applicable to this service: ID, PR-AC2, PR-AT, PR.DS-3 - 8, PR.MA, PR.IP-4, PR.IP-5, PR.IP-7 - 12, PR.PT-2, PR.PT-5, DE.AE-4, DE.AE-5, DE.CM-2, DE.CM-6, DE.DP, RS.RP, RS.CO, RS.MI, RS.IM, RC

These Capital Group control statements are not applicable to this service: 7,8,9,10

## Preventative Controls
### 1. Permissions within the service are established in line with individual need and least-privilege is enforced
NIST CSF:
|NIST Subcategory Control|Description|
|-----------|------------------------|
|PR.AC-1|Identities and credentials are issued, managed, verified, revoked, and audited for authorized devices, users and processes|
|PR.AC-3|Remote Access is managed|
|PR.AC-4|Access permissions and authorizations are managed, incorporating the princciples of least privilege and separation of duties|
|PR.AC-6|Identities are proofed and bound to credentials and asserted in interactions|
|PR.PT-3|The principle of least frunctionality is incorporated by configuring systems to rpovide only essential capabilities|

Capital Group:
|Control Statement|Description|
|------|----------------------|
|8|Local IAM secrets are rotated every 90 days, including accounts IaaS resources.|

**Why?** Different users/services will require different levels of access within the service. Having clearly defined policies and limiters will ensure that a user is not able to manage build projects in ways that they shouldn't be able to.

**How?** Attach a permissions policy to a security group that is associated with the particular account that will access the node cluster.  

Use Redis AUTH to allow access to the ElastiCache cluster to execute commands.
There are constraints to be aware of when using AUTH.

* Tokens must be 16???128 printable characters.
* Nonalphanumeric characters are restricted to (!, &, #, $, ^, <, >, -).
* AUTH can only be enabled for encryption in-transit enabled ElastiCache for Redis clusters.
* When creating the token it is suggested to containt at least three offthe following 
    * Uppercase characters    
    * Lowercase characters    
    * Digits    
    * Nonalphanumeric characters (!, &, #, $, ^, <, >, -)
* Tokens must not contain a dictionary word or a slightly modified dictionary word.
* Tokens must not be the same as or similar to a recently used token.

You can apply the AUTH token when creating the cluster.
**Key Parameters**

* **--engine** ??? Must be redis.
* **--engine-version** ??? Must be 5.x or later.
* **--transit-encryption-enabled** ??? Required for authentication 
* **--auth-token** ??? This value must be the correct token for this token-protected Redis server.
* **--cache-subnet-group** ??? Required 


        aws elasticache create-replication-group \
            --replication-group-id authtestgroup \
            --replication-group-description authtest \
            --engine redis \
            --engine-version 4.0.10 \
            --cache-node-type cache.m4.large \
            --num-node-groups 1 \
            --replicas-per-node-group 2 \
            --cache-parameter-group default.redis3.2.cluster.on \
            --transit-encryption-enabled \
            --auth-token This-is-a-sample-token \
            --cache-subnet-group sng-test

Modifying the auth token supports two strategies: ROTATE and SET. The ROTATE strategy adds an additional AUTH token to the server while retaining the previous token. The SET strategy updates the server to support just a single AUTH token. Make these modification calls with the --apply-immediately parameter to apply changes immediately.

To update a Redis server with a new AUTH token, call the ```ModifyReplicationGroup``` API with the ```--auth-token``` parameter as the new auth token and the ```--auth-token-update-strategy``` with the value ROTATE. Once the modification is complete, the cluster will support the previous AUTH token in addition to the one specified in the ``auth-token`` parameter.

To Rotate an AUTH Toekn you will need to modifiy the config of a server that already supports **two AUTH** tokens, the oldest AUTH token will also be removed during this operation, allowing a server to support up to two most recent AUTH tokens at a given time.

At this point, you can proceed by updating the client to use the latest AUTH token. Once the clients are updated, you can use the SET strategy for AUTH token rotation (explained in the following section) to exclusively start using the new token.

The following AWS CLI operation modifies a replication group to rotate the AUTH token ``This-is-the-rotated-token``.

        aws elasticache modify-replication-group \ 
        --replication-group-id authtestgroup \ 
        --auth-token This-is-the-rotated-token \ 
        --auth-token-update-strategy ROTATE \ 
        --apply-immediately n


To SET the AUTH token on a Redis server with **two AUTH** tokens to support **a single AUTH token**, call the ``ModifyReplicationGroup`` API operation. Call ``ModifyReplicationGroup`` with the ``--auth-token`` parameter as the new AUTH token and the ``--auth-token-update-strategy`` parameter with the value SET. The auth-token parameter must be the same value as the last AUTH token rotated. After the modification is complete, the Redis server supports only the AUTH token specified in the auth-token parameter.

The following AWS CLI operation modifies a replication group to set the AUTH token to ``This-is-the-set-token``.

        aws elasticache modify-replication-group \ 
        --replication-group-id authtestgroup \ 
        --auth-token This-is-the-set-token\ 
        --auth-token-update-strategy SET \ 
        --apply-immediately 



### 2. Enable Security Groups to limit access to ElastiCache instances
NIST CSF:
|NIST Subcategory Control|Description|
|-----------|------------------------|
|PR.PT-4|Communications and control networks are protected|
|PR.AC-3|Remote access is managed|
|PR.AC-5|Network integrity is protected (e.g., network segregation, network segmentation)|

Capital Group:
|Control Statement|Description|
|------|----------------------|
|6|Any AWS service used by CG should not be directly available to the Internet and the default route is always the CG gateway.|

**Why?** Limiting access to resources ensures that data is protected and only visiable to identities that have permissions to view the data in the ElastiCache instances. 

**How?**  ElasticCache instaces work with EC2 and VPC. Create a Security Group in the EC2 instace that the Elasticcache instance will connect to. You will then need to create a **subnet group** that you cache clusters will operate within you VPC. ElastiCache uses that cache subnet group to assign IP addresses within that subnet to each cache node in the cluster. Depending on the size of the subnet will determine how many nodes you can add to the cluster.  



### 3. Data must be encrypted using CG BYOK for encyprtion
NIST CSF:
|NIST Subcategory Control|Description|
|-----------|------------------------|
|PR.DS-1|Data-at-rest is protected|
|PR.DS-2|Data-in-transit is protected|

Capital Group:
|Control Statement|Description|
|------|----------------------|
|1|All Data-at-rest must be enccrypted and use a Capital Group BYOK encryption key|
|2|All Data-in-transit must be encrypted using certificates using Capital Group Certificate Authority|
|3|Keys storied in a Key Management System (KMS) should be created by Capital Group's hardware security module (HSM) and are a minimum of AES-256|
|9|Encryption keys are rotated annually.|

**Why?** Data encryption helps prevent unauthorized users from reading data on a cluster and associated data storate systems. This includes data saved to persistent media and data that may be intercepted as it travels the network. Data encryption helps prevent unauthorized users from reading data on a cluster and associated data storage systems. This includes data saved to persistent media, known as data at rest, and data that may be intercepted as it travels the network, known as data in transit.

**How?** To help keep your data secure, Amazon ElastiCache and Amazon EC2 provide mechanisms to guard against unauthorized access of your data on the server. By providing in-transit encryption capability, ElastiCache gives you a tool you can use to help protect your data when it is moving from one location to another.


ElastiCache in-transit encryption allows you to increase the security of your data at its most vulnerable points???when it is in transit from one location to another. 
ElastiCache in-transit encryption implements the following features:

* **Encrypted connections**???both the server and client connections are Secure Socket Layer (SSL) encrypted.

* **Encrypted replication**???data moving between a primary node and replica nodes is encrypted.

* **Server authentication**???clients can authenticate that they are connecting to the right server.

* **Client authentication**???using the Redis AUTH feature, the server can authenticate the clients.

You can enable in-transit encryption when you create an ElastiCache for Redis replication group, use the parameter ```transit-encryption-enabled```. 

**Enabling In-Transit Encryption on Redis (Cluster Mode Disabled) Cluster (CLI)**
* **--engine**???Must be redis.
* **--engine-version**???Must be 3.2.6, 4.0.10 or later.
* **--transit-encryption-enabled???Required**. If you enable in-transit encryption you must also provide a value for the --cache-subnet-group parameter.
* **--num-cache-clusters**???Must be at least 1. The maximum value for this parameter is six.

**Enabling In-Transit Encryption on a Cluster for Redis (Cluster Mode Enabled) (CLI)**

* **--engine**???Must be redis.
*** --engine-version**???Must be 3.2.6, 4.0.10 or later.
* **--transit-encryption-enabled???Required**. If you enable in-transit encryption you must also provide a value for the --cache-subnet-group parameter.
* Use one of the following parameter sets to specify the configuration of the replication group's node groups:
    * **--num-node-groups**???Specifies the number of shards (node groups) in this replication group. The maximum value of this parameter is 90.
    * **--replicas-per-node-group**???Specifies the number of replica nodes in each node group. The value specified here is applied to all shards in this replication group. The maximum value of this parameter is 5.
    * **--node-group-configuration**???Specifies the configuration of each shard independently.
  
For More information see Endnote 1. 

ElastiCache for Redis at-rest encryption is a Capital Group required feature to increase data security by encrypting on-disk data. When enabled on a replication group, it encrypts the following aspects:

**Disk during sync, backup and swap operations
Backups stored in Amazon S3**

A CG issued encryption key is necessary for encryption at rest. To have one created you'll need to send a request to the Sec Eng - Cert/Key Services team

Include the following pieces of information 

* AWS Account Name
* Region
* AWS Service Name

        Example
        AWS Account Name: newnad-qa (348521753090)
        Region(s): USeast1
        AWS Service Name: Elasticache

**Enabling At-Rest Encryption on a Redis (Cluster Mode Disabled) Cluster (CLI)**
* **--engine???** Must be redis.
* **--engine-version???** Must be 3.2.6, 4.0.10 or later.
* **--at-rest-encryption-enabled???** Required to enable at-rest encryption.

**Example**

    aws elasticache create-replication-group \
        --replication-group-id my-classic-rg \
        --replication-group-description "3 node replication group" \
        --cache-node-type cache.m4.large \
        --engine redis \
        --engine-version 4.0.10 \
        --at-rest-encryption-enabled \  
        --num-cache-clusters 3 \
        --cache-parameter-group default.redis4.0
    
**Enabling At-Rest Encryption on a Cluster for Redis (Cluster Mode Enabled) (CLI)**

* **--engine???** Must be redis.
* **--engine-version???** Must be 3.2.6, 4.0.10 or later.
* **--at-rest-encryption-enabled???** Required to enable at-rest encryption.
*** --cache-parameter-group???** Must be default-redis4.0.cluster.on or one derived from it to make this a cluster mode enabled replication group.

**Example**

    aws elasticache create-replication-group \
       --replication-group-id my-clustered-rg \
       --replication-group-description "redis clustered cluster" \
       --cache-node-type cache.m3.large \
       --num-node-groups 3 \
       --replicas-per-node-group 2 \
       --engine redis \
       --engine-version 4.0.10 \
       --at-rest-encryption-enabled \
       --cache-parameter-group default.redis4.0.cluster.on



## Detective
### 1. Establish Config rules to monitor for deviations from normal configuration
NIST CSF:
|NIST Subcategory Control|Description|
|-----------|------------------------|
|DE.AE-1|A baseline of network operations and expected data flows for users and systems is established and managed|
|DE.AE-2|Detected events are analyzed to understand attack targets and methods|
|DE.AE-3|Event data are collected and correlated from multiple sources and sensors|
|DE.CM-1|The network is monitored to detect potential cybersecurity events|
|DE.CM-3|Personnel activity is monitored to detect potential cybersecurity events|
|DE.CM-7|Monitoring for unauthorized personnel, connections, devices, and software is performed|

**Why?** Having additional layers of protection helps keep the service within the organization's expected configuration. In the event that a cluster is incorrectly configured, and preventative controls somehow miss it, Config can be set up to monitor for this.

**How?** Amazon EMR automatically sends events to a CloudWatch event stream. You can create rules that match events according to a specified pattern, and route the events to targets to take action, such as sending an email notification. Patterns are matched against the event JSON object. For more information about Amazon EMR event details see Endnote 3 for Amazon EMR Events in the Amazon CloudWatch Events User Guide. 


## Respond/Recover
### 1. Utilize Amazon EventBridge for automated incident response
NIST CSF:
|NIST Subcategory Control|Description|
|-----------|------------------------|
|RS.AN-1|Notifications from detection systems are investigated|
|RS.AN-2|The impact of the incident is understood|
|RS.AN-3|Forensics are performed|
|RS.AN-4|Incidents are categorized consistent with response plans|
|RS.AN-5| Processes are established to receive, analyze and respond to vulnerabilities disclosed to the organization from internal and external sources (e.g. internal testing, security bulletins, or security researchers)|

**Why?** Using Amazon EventBridge, you can automatically report and respond to incidents, including actions taken on worker nodes, or EMR API Calls via CloudTrail.

**How?** For various incident types, an appropriate response rule should be determined and added to EventBridge. Depending on the organization's Incident Response Plan, there may be need for human interaction in some cases, or fully automated remediation in other cases.



## Endnotes
### Endnote 1 https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/in-transit-encryption.html#in-transit-encryption-overview

### Endnote 2 <!-- omit in toc -->

### Endnote 3 <!-- omit in toc -->

## Capital Group Control Statements
1. All Data-at-rest must be encrypted and use a CG BYOK encryption key.
2. All Data-in-transit must be encrypted using certificates using CG Certificate Authority.
3. Keys storied in a Key Management System (KMS) should be created by Capital Group's hardware security module (HSM) and are a minimum of AES-256.
4. AWS services should have logging enabled and those logs delivered to CloudTrail or Cloud Watch.
5. Local AWS IAM accounts are restricted to services and no user accounts are to be provisioned including IaaS resources.
6. Any AWS service used by CG should not be directly available to the Internet and the default route is always the CG gateway.
7. Use of AWS IAM accounts are restricted to CG networks.
8. Local IAM secrets are rotated every 90 days, including accounts IaaS resources.
9. Encryption keys are rotated annually.
10. Root accounts must have 2FA/MFA enabled.

## Capital Group Glossory 
**Data** - Digital pieces of information stored or transmitted for use with an information system from which understandable information is derived. Items that could be considered to be data are: Source code, meta-data, build artifacts, information input and output.  
 
**Information System** - An organized assembly of resources and procedures for the collection, processing, maintenance, use, sharing, dissemination, or disposition of information. All systems, platforms, compute instances including and not limited to physical and virtual client endpoints, physical and virtual servers, software containers, databases, Internet of Things (IoT) devices, network devices, applications (internal and external), Serverless computing instances (i.e. AWS Lambda), vendor provided appliances, and third-party platforms, connected to the Capital Group network or used by Capital Group users or customers.

**Log** - a record of the events occurring within information systems and networks. Logs are composed of log entries; each entry contains information related to a specific event that has occurred within a system or network.

**Information** - communication or representation of knowledge such as facts, data, or opinions in any medium or form, including textual, numerical, graphic, cartographic, narrative, or audiovisual. 

**Cloud computing** - A model for enabling ubiquitous, convenient, on-demand network access to a shared pool of configurable computing resources (e.g., networks, servers, storage, applications, and services) that can be rapidly provisioned and released with minimal management effort or service provider interaction.

**Vulnerability**  - Weakness in an information system, system security procedures, internal controls, or implementation that could be exploited or triggered by a threat source. Note: The term weakness is synonymous for deficiency. Weakness may result in security and/or privacy risks.
