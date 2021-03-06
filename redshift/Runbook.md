<img src="https://a0.awsstatic.com/libra-css/images/logos/aws_logo_smile_1200x630.png" alt="AWS" width="250"/>

# AWS Redshift - Security Playbook <!-- omit in toc -->

## NIST Cybersecurity Framework Alignment <!-- omit in toc -->

**Generated By:**  
[John Soto (JHVS)](https://cgweb3/profile/JHVS)<br>
[Rob Goss (RMG)](https://cgweb3/profile/RMG)
<br>
Security Engineering

**Last Update:** *05/03/2021*

## Table of Contents <!-- omit in toc -->
- [Overview](#overview)
- [Preventative](#preventative-controls)
    - [1. Redshift resources deployed in Private VPC to prevent Public Access](#1-Redshift-resources-deployed-in-Private-VPC-to-prevent-Public-Access)
    - [2. Redshift resources are Encrypted using CG Managed KMS Keys](#2-Redshift-resources-are-Encrypted-using-CG-Managed-KMS-Keys)
    - [3. Redshift connections are Encrypted in transitusing TLS 1.2](#3-Redshift-connections-are-Encrypted-in-transitusing-TLS-1-2)
    - [4. Redshift implements access controls to enforce least privilege](#4-Redshift-implement-access-controls-to-enforce-least-privilege)
- [Detective](#Detective)
    - [1. Amazon Redshift should have automatic upgrades to major versions enabled](#1-amazon-redshift-should-have-automatic-upgrades-to-major-versions-enabled)
    - [2. Redshift Resources are tagged according to CG standards](#2-Redshift-Resources-are-tagged-according-to-CG-standards)
    - [3. CloudTrail logging enabled and sent to Splunk](#3-CloudTrail-logging-enabled-and-sent-to-Splunk)
    - [4. CloudWatch logging enabled and sent to Splunk](#4-CloudWatch-logging-enabled-and-sent-to-Splunk)
- [Respond & Recover](#Respond/Recover)
- [Endnotes](#Endnotes)
- [Capital Group Glossory ](#Capital-Group-Glossory)
<br><br>

## Overview
Amazon Redshift is a fully managed, cloud-based, petabyte-scale data warehouse service by Amazon Web Services (AWS). It is an efficient solution to collect and store all your data and enables you to analyze it using various business intelligence tools to acquire new insights for your business and customers.

<img src="/docs/img/redshift/redshift.png" width="800"><br>

### **Core Features of Redshift**
 - Redshift is very fast and performant when it comes to loading data and querying it for analytical and reporting purposes
 - Horizontally scalable
 - Massive Storage Capacity
 - VPC network isolation
 - No servers to manage
 - Various data encryption options are available in multiple places in Redshift
<br><br>

## Preventative Controls
<img src="/docs/img/Prevent.png" width="50">

### 1. Redshift resources deployed in Private VPC to prevent Public Access
<br>

**NIST CSF:** <br>

|NIST Subcategory Control|Description|
|-----------|------------------------|
|PR.PT-4|Communications and control networks are protected|
|PR.PT-5|Mechanisms (e.g., failsafe, load balancing, hot swap) are implemented to achieve resilience requirements in normal and adverse situations|
|PR.AC-3|Remote access is managed|
|PR.AC-5|Network integrity is protected (e.g., network segregation, network segmentation)|
<br>

**Capital Group:** <br>

|Control Statement|Description|
|------|----------------------|
|6|Any AWS service used by CG should not be directly available to the Internet and the default route is always the CG gateway.|
|7|Use of AWS IAM accounts are restricted to CG networks.|
<br>

**Why?** <br>
Multiple layers of security are needed to help ensure that resources are safe from unwanted access. In addition to IAM policies, having strong network controls in place isolates your instances from outside threats. If traffic is not able to reach an instance, then remote threats can be mitigated.  Amazon Redshift cluster is locked down by default so nobody has access to it. To grant other users inbound access to an Amazon Redshift cluster, you associate the cluster with a security group. If you are on the EC2-VPC platform, you can either use an existing Amazon VPC security group or define a new one. You then associate it with a cluster as described following. If you are on the EC2-Classic platform, you define a cluster security group and associate it with a cluster. For more information on using cluster security groups on the EC2-Classic platform, see `Endnote 1`.  This section details the process of deploying a Redshift Cluster in a private VPC on a CG internal subnet. 

**How?**<br> 
1. Set up a VPC:
    
    - When you launch a Redshift Cluster, you can utilize the VPC that has been generated for your AWS account upon creation of the account.

    - To Determine your VPC identifier: 
        ``` 
        aws ec2 describe-vpcs 
        ```

2. Create / Connect an Amazon Redshift Cluster subnet group to your Amazon Redshift Cluster

    - Your VPC that was created at the creation of your account will have subnets, you can either use the already created subnets or create your own. 
    
    - Why to create your own?
        - A cluster subnet group allows you to specify a set of subnets in your VPC for only your Redshift cluster. When provisioning a cluster you provide the subnet group and Amazon Redshift creates the cluster on one of the subnets in the group.
    
    - To create a subnet:
        
        - [AWS Console](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-getting-started.html)

        - AWS CLI
            ```
            aws ec2 create-subnet --vpc-id vpc-07e8ffd50fEXAMPLE --cidr-block 10.211.169.x/26
            ``` 
    
    - To get the subnets in VPC:
        
        - [AWS Console](https://console.aws.amazon.com/vpc/home?region=REGION-HERE#subnets)

        - AWS CLI
            ```
            aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-3EXAMPLE"
            ``` 

3. Create / Connect a Security Group to the Redshift Cluster
    - ***NOTE:*** You can create up to 100 VPC security groups for a VPC and associate a VPC security group with many clusters. However, you can only associate up to five VPC security groups with a given cluster.

    - ***Recommended:*** Create your own Security Group for your Redshift Cluster 
        
        - To create your own Security Group:
        
            - [AWS Console](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html)
            
            - AWS CLI
                ```
                aws ec2 create-security-group --group-name MySecurityGroup --description "My security group" --vpc-id vpc-1a2b3c4d
                ```

    - To get the ID of a pre-existing Security Group:

        -  [AWS Console](https://console.aws.amazon.com/vpc/home?region=REGION-HERE#securityGroups)

        - AWS CLI
            ```
            aws ec2 describe-security-groups --filters "Name=vpc-id,Values=vpc-3EXAMPLE
            ```

4. Create the Redshift Cluster, make the following modifications:

    -  Via AWS Console:
        - Follow the steps in [Getting started with AWS Redshift](https://docs.aws.amazon.com/redshift/latest/gsg/getting-started.html)
        - To display the Additional configurations section, switch off Use defaults.
        - In the Network and security section, specify the Virtual private cloud (VPC), Cluster subnet group, and VPC security group that you set up.
    - Via AWS CLI:
        ```
        aws redshift create-cluster --node-type NODE-TYPE-HERE --number-of-nodes x --master-username example-name --master-user-password SuperSecretPasswordHere --cluster-security-groups securityGroup-ID --cluster-subnet-group-name SubnetGroupNames --cluster-identifier mycluster
        ```
<br>

### 2. Redshift-resources-are-Encrypted-using-CG-Managed-KMS-Keys 
<br>

NIST CSF:
|NIST Subcategory Control|Description|
|-----------|------------------------|
|PR.DS-1|Data-at-rest is protected|

<br></br>
Capital Group:
|Control Statement|Description|
|------|----------------------|
|1|All Data-at-rest must be encrypted and use a CG BYOK encryption key.|
|2|All Data-in-transit must be encrypted using certificates using CG Certificate Authority.|
|3|Keys storied in a Key Management System (KMS) should be created by Capital Group's hardware security module (HSM) and are a minimum of AES-256.|

<br>

**Why?** 

AWS Redshift encrypted DB instances provide an additional layer of data protection by securing your data from unauthorized access to the underlying storage. You can use AWS Redshift encryption to increase data protection of your applications deployed in the cloud, and to fulfill compliance requirements for data-at-rest encryption.  By default, Amazon Redshift selects your default key as the master key, however please use CG encryption keys when encrypting your Redshift instance that is package with your AWS account.


**How?** 
#### Enable encryption of data-at-rest <!-- omit in toc -->
For Amazon Redshift, you can enable database encryption for your clusters to help protect data at rest. When you enable encryption for a cluster, the data blocks and system metadata are encrypted for the cluster and its snapshots.

**In the Console:**  
To enable encryption for a new Redshift Cluster, please follow the steps outlined [here](https://docs.aws.amazon.com/redshift/latest/mgmt/configuring-db-encryption-console.html) by AWS.  Also, please **utilize CG CMKs to encrypt the Cluster** in order to stay compliant with CG standards.

**In the AWS CLI:**  
If you use the `create-cluster` AWS CLI command to create an encrypted DB instance, set the `--encrypted` parameter to `true`. Set the `--kms-key-id` parameter to the Amazon Resource Name (ARN) for the AWS KMS encryption key for the Redshift Cluster instance. 

- Example:
    ```
    aws redshift create-cluster --node-type NODE-TYPE-HERE --number-of-nodes x --master-username example-name /
    --master-user-password SuperSecretPasswordHere --cluster-security-groups securityGroup-ID /
    --cluster-subnet-group-name SubnetGroupNames --cluster-identifier mycluster /
    --encrypted true --kms-key-id arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab
    ```

<br>

### 3. Redshift connections are Encrypted in transitusing TLS 1.2 
<br>

**NIST CSF:**
|NIST Subcategory Control|Description|
|-----------|------------------------|
|PR.DS-2|Data-in-transit is protected|

<br>

**Capital Group:**
|Control Statement|Description|
|------|----------------------|
|1|All Data-in-transit must be encrypted using certificates using CG Certificate Authority.|

<br>

**Why?** 

AWS Redshift as with all cloud resources deployed for CG requires the encryption of data in transit through AWS's network. Redshift supports TLS 1.2, allowing the service to meet CG's stringent cloud security standards in this regard.

**How?** 
#### Enable encryption of data-in-transit <!-- omit in toc -->
You can configure your environment to protect the confidentiality and integrity data in transit.  One of the perks to AWS Redshift is that the service protects your data in transit within the AWS Cloud, Amazon Redshift uses hardware accelerated SSL to communicate with Amazon S3 or Amazon DynamoDB for COPY, UNLOAD, backup, and restore operations.

- [Encryption of data in transit between an Amazon Redshift cluster and SQL clients over JDBC/ODBC](#using-tls-with-mariadb)  
- [Encryption of data in transit between an Amazon Redshift cluster and Amazon S3 or DynamoDB](#using-tls-with-mysql)  
- [Encryption and signing of data in transit between AWS CLI, SDK, or API clients and Amazon Redshift endpoints](#using-tls-with-oracle)  
- [Encryption of data in transit between Amazon Redshift clusters and AQUA](#using-tls-with-postgresql)

<br>

##### Encryption of data in transit between an Amazon Redshift cluster and SQL clients over JDBC/ODBC <!-- omit in toc -->
- You can connect to Amazon Redshift clusters from SQL client tools over Java Database Connectivity (JDBC) and Open Database Connectivity (ODBC) connections.  This connection will be secured upon connection.

- Redshift also supports Secure Sockets Layer (SSL) connections to encrypt data and server certificates to validate the server certificate that the client connects to at the time of connection. The client connects to the leader node of an Amazon Redshift cluster to connect to all instances of the database. The following links below detail how to setup SSL and certificates for Redshift:
    - [Configuring SSL](https://docs.aws.amazon.com/redshift/latest/mgmt/connecting-ssl-support.html)

    - To support SSL connections, Amazon Redshift creates and installs AWS Certificate Manager (ACM) issued certificates on each cluster. To set this up please refere to
    `Endnote 2`

    
##### Encryption of data in transit between an Amazon Redshift cluster and Amazon S3 or DynamoDB <!-- omit in toc -->
- Amazon Redshift uses hardware accelerated SSL to communicate with Amazon S3 or DynamoDB for COPY, UNLOAD, backup, and restore operations.

    - Redshift Spectrum supports the Amazon S3 server-side encryption (SSE) using your account's encryption key managed by the AWS Key Management Service (KMS).

    - Encrypt Amazon Redshift loads with Amazon S3 and AWS KMS, for more info please refere to [AWS Documentation](https://aws.amazon.com/blogs/big-data/encrypt-your-amazon-redshift-loads-with-amazon-s3-and-aws-kms/)


##### Encryption and signing of data in transit between AWS CLI, SDK, or API clients and Amazon Redshift endpoints <!-- omit in toc -->
- Amazon Redshift provides HTTPS endpoints for encrypting data in transit.

-   To protect the integrity of API requests to Amazon Redshift, API calls must be signed by the caller. Calls are signed by an X.509 certificate or the customer's AWS secret access key according to the Signature Version 4 Signing Process (Sigv4). 
    - Documentation: [Signature Version 4 Signing Process](https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html) in the AWS General Reference.

- Use the AWS CLI or one of the AWS SDKs to make requests to AWS. These tools automatically sign the requests for you with the access key that you specify when you configure the tools.

##### Encryption of data in transit between Amazon Redshift clusters and AQUA <!-- omit in toc -->
- Data is transmitted between AQUA and Amazon Redshift clusters over a TLS-encrypted channel. This channel is signed according to the Signature Version 4 Signing Process (Sigv4).
    - For what AWS AQUA is, please refere to [Advance Query Accelerator Documentation](https://aws.amazon.com/blogs/aws/new-aqua-advanced-query-accelerator-for-amazon-redshift/)

<br>

### 4. Redshift implements access controls to enforce least privilege

NIST CSF:
|NIST Subcategory Control|Description|
|-----------|------------------------|
|PR.AC-1|Identities and credentials are issued, managed, verified, revoked, and audited for authorized devices, users and processes|
|PR.AC-3|Remote access is managed|
|PR.AC-4|Access permissions and authorizations are managed, incorporating the principles of least privilege and separation of duties|
|PR.AC-6|Identities are proofed and bound to credentials and asserted in interactions|
|PR.AC-7|Users, devices, and other assets are authenticated (e.g., single-factor, multi-factor) commensurate with the risk of the transaction (e.g., individuals??? security and privacy risks and other organizational risks)|

Capital Group:
|Control Statement|Description|
|------|----------------------|
|5|AWS IAM User accounts are only to be created for use by services or products that do not support IAM Roles. Services are not allowed to create local accounts for human use within the service. All human user authentication will take place within CG???s Identity Provider.|
|8|AWS IAM User secrets, including passwords and secret access keys, are to be rotated every 90 days. Accounts created locally within any service must also have their secrets rotated every 90 days.|
|10|Administrative access to AWS resources will have MFA enabled|

<br>

**Why?** 

When you create custom policies, grant only the permissions required to perform a task. Start with a minimum set of permissions and grant additional permissions as necessary. Doing so is more secure than starting with permissions that are too lenient and then trying to tighten them later.  
To the extent that it's practical, define the conditions under which your identity-based policies allow access to a resource. For example, you can write conditions to specify a range of allowable IP addresses that a request must come from. You can also write conditions to allow requests only within a specified date or time range, or to require the use of SSL or MFA.


**How?** 

Redshift is fully integrated with AWS IAM for authentication and access control. Use IAM to create IAM users, groups and roles with appropriate permissions, and then add users to appropriate groups and roles. Use Multifactor Authentication for extra security (especially for users with administrative privileges).  
Use AWS Identity and Access Management (IAM) policies to assign permissions that determine who is allowed to manage Amazon Redshift resources. For example, you can use IAM to determine who is allowed to create, describe, modify, and delete DB instances, tag resources, or modify security groups. List 

The following shows an example of a permissions policy. The policy allows a user to create, delete, modify, and reboot all clusters, and then denies permission to delete or modify any clusters where the cluster identifier starts with production.
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid":"AllowClusterManagement",
      "Action": [
        "redshift:CreateCluster",
        "redshift:DeleteCluster",
        "redshift:ModifyCluster",
        "redshift:RebootCluster"
      ],
      "Resource": [
        "*"
      ],
      "Effect": "Allow"
    },
    {
      "Sid":"DenyDeleteModifyProtected",
      "Action": [
        "redshift:DeleteCluster",
        "redshift:ModifyCluster"
      ],
      "Resource": [
        "arn:aws:redshift:us-west-2:123456789012:cluster:production*"
      ],
      "Effect": "Deny"
    }
  ]
}
```


#### IAM Database Authentication <!-- omit in toc -->
Amazon Redshift provides the [GetClusterCredentials API](https://docs.aws.amazon.com/redshift/latest/APIReference/API_GetClusterCredentials.html) operation to generate temporary database user credentials. You can configure your SQL client with Amazon Redshift JDBC or ODBC drivers that manage the process of calling the GetClusterCredentials operation. They do so by retrieving the database user credentials, and establishing a connection between your SQL client and your Amazon Redshift database. You can also use your database application to programmatically call the GetClusterCredentials operation, retrieve database user credentials, and connect to the database.

With the current Okta setup as CG's identity provider (IdP), we can manage access to Amazon Redshift resources via an IAM role.  With that IAM role, you can generate temporary database credentials and log in to Amazon Redshift databases.

For more information on setting up temporary credentials to access your Redshift Cluster, please refer to `Endnote 3` 

##### Identity provider federation - 
- You can use Okta as an identity provider (IdP) to access your Amazon Redshift cluster.  The following is an example of how to setup MFA for Redshift via Okta [AWS Redshift Okta MFA Setup](https://aws.amazon.com/blogs/big-data/federate-amazon-redshift-access-with-okta-as-an-identity-provider/)


## Detective Controls
<img src="/docs/img/Detect.png" width="50">
<br>

### 1. Amazon Redshift should have automatic upgrades to major versions enabled

**Why?**<br> 
This control checks whether automatic major version upgrades are enabled for the Amazon Redshift cluster\.

Enabling automatic major version upgrades ensures that the latest major version updates to Amazon Redshift clusters are installed during the maintenance window\. These updates might include security patches and bug fixes\. Keeping up-to-date with patch installation is an important step in securing systems\.

**How?**<br>
Enable Redshift allow version upgrades\.

 - To enable this option from the AWS CLI, use the Amazon Redshift `modify-cluster` command to set the `--allow-version-upgrade attribute`\.
 - AWS Redshift `modify-cluster --cluster-identifier clustername --allow-version-upgrade`
 - Where clustername is the name of your Amazon Redshift cluster.
<br>

### 2. Redshift Resources are tagged according to CG standards

`This Section will be updated soon.`

### 3. CloudWatch logging enabled and sent to Splunk

`This Section will be updated soon.`

### 4. CloudTrail logging enabled and sent to Splunk
`This Section will be updated soon.`
<br><br>

## Respond/Recover
<img src="/docs/img/Monitor.png" width="50">

`This Section will be updated soon.`
<br><br>

## Endnotes
1. https://docs.aws.amazon.com/redshift/latest/mgmt/working-with-security-groups.html
2. https://docs.aws.amazon.com/redshift/latest/mgmt/connecting-transitioning-to-acm-certs.html
3. https://docs.aws.amazon.com/redshift/latest/mgmt/generating-iam-credentials-steps.html
<br><br>

## Capital Group Glossory 
**Data** - Digital pieces of information stored or transmitted for use with an information system from which understandable information is derived. Items that could be considered to be data are: Source code, meta-data, build artifacts, information input and output.  
 
**Information System** - An organized assembly of resources and procedures for the collection, processing, maintenance, use, sharing, dissemination, or disposition of information. All systems, platforms, compute instances including and not limited to physical and virtual client endpoints, physical and virtual servers, software containers, databases, Internet of Things (IoT) devices, network devices, applications (internal and external), Serverless computing instances (i.e. AWS Lambda), vendor provided appliances, and third-party platforms, connected to the Capital Group network or used by Capital Group users or customers.

**Log** - a record of the events occurring within information systems and networks. Logs are composed of log entries; each entry contains information related to a specific event that has occurred within a system or network.

**Information** - communication or representation of knowledge such as facts, data, or opinions in any medium or form, including textual, numerical, graphic, cartographic, narrative, or audiovisual. 

**Cloud computing** - A model for enabling ubiquitous, convenient, on-demand network access to a shared pool of configurable computing resources (e.g., networks, servers, storage, applications, and services) that can be rapidly provisioned and released with minimal management effort or service provider interaction.

**Vulnerability**  - Weakness in an information system, system security procedures, internal controls, or implementation that could be exploited or triggered by a threat source. Note: The term weakness is synonymous for deficiency. Weakness may result in security and/or privacy risks.