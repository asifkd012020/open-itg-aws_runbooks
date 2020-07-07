<img src="https://a0.awsstatic.com/libra-css/images/logos/aws_logo_smile_1200x630.png" alt="AWS" width="250"/>

# EMR - Security Runbook <!-- omit in toc -->
## NIST Cybersecurity Framework Alignment <!-- omit in toc -->

**Generated By:**  
Josh Lutrell
Ken Jeanis

*AWS Professional Services*

## Disclaimer
> The following applies to this document and all other documents, information, data, and responses (written or verbal) provided by Amazon Web Services, Inc. or any of its affiliates (collectively, "**AWS**") in connection with responding to this request and other related requests (collectively, this "**Response**"): This Response is expressly (a) informational only and provided solely for discussion purposes, (b) non-binding and not an offer to contract that can be accepted by any party, (c) provided "as is" with no representations or warranties whatsoever, and (d) based on AWS's current knowledge and may change at any time due to a variety of factors such as changes to your requirements or changes to AWS's service offerings. All obligations must be set forth in a separate, definitive written agreement between the parties. Neither party will have any liability for any failure or refusal to enter into a definitive agreement. All use of AWS's service offerings will be governed by the AWS Customer Agreement available at [http://aws.amazon.com/agreement/](http://aws.amazon.com/agreement/) (or other definitive written agreement between the parties governing the use of AWS's service offerings) (as applicable, the "**Agreement**"). If the parties have an applicable Nondisclosure Agreement ("**NDA**"), then the NDA will apply to all Confidential Information (as defined in the NDA) disclosed in connection with this Response. AWS's pricing is publicly available and subject to change in accordance with the Agreement. Pricing information (if any) provided in this Response is only an estimate and is expressly not a binding quote. Fees and charges will be based on actual usage of AWS services, which may vary from the estimates provided. Nothing in this Response will modify or supplement the terms of the Agreement or the NDA. No part of this Response may be disclosed without AWS's prior written consent. 

## Table of Contents <!-- omit in toc -->
- [Disclaimer](#disclaimer)
- [Overview](#overview)
- [Preventative Controls](#preventative-controls)
  - [1. Permissions within the service are established in line with individual need and least-privilege is enforced](#1-preventative-item-1)
  - [2. Data must be encrypted at all times using a CG BYOK for encryption where possible ](#2-preventative-item-2)
  - [3Infrastructure must be secured for monitoring, audit, availability, and access control](#3-preventative-item-3)
- [Detective](#detective)
  - [1. Establish Config rules to monitor for deviations from normal configuration](#1-detective-item-1)
- [Respond/Recover](#respondrecover)
  - [1. Utilize Amazon EventBridge for automated incident response](#1-respondrecover-item-1)
- [Endnotes](#endnotes)

## Overview
AWS provides a number of security features for EMR which help you comply with the NIST Cybersecurity Framework. The following runbook will outline what the AWS best practices are, how they align to NIST, and how to implement these best practices within your organization.

These NIST Controls and Subcategories are not applicable to this service: ID, PR-AC2, PR-AT, PR.DS-3 - 8, PR.MA, PR.PT-2, PR.PT-5, DE.AE-4, DE.AE-5, DE.CM-2, DE.CM-6, DE.DP, RS.RP, RS.CO, RS.MI, RS.IM, RC

These Capital Group control statements are not applicable to this service: 5,7,8,9,10

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


**Why?** Different users/services will require different levels of access within the service. Having clearly defined policies and limiters will ensure that a user is not able to manage build projects in ways that they shouldn't be able to.
* Amazon EMR and applications such as Hadoop and Spark need permissions to access other AWS resources and perform actions when they run. Each cluster in Amazon EMR must have a service role and a role for the Amazon EC2 instance profile. 

**How?** Specify custom IAM Roles when creating the cluster. A User account is not authorized to call EC2 instances from EMR a service role is needed to acces the EC2 instance. When creating the cluster specifiy a custom service role for Amazon EMR along with a custom role for EC2 instances when creating a cluster. 

Potential Required Rules
* Custom autoscaling role required if cluster in EMR uses automatic scaling `--auto-scaling-role`
* Custom service role for EMR Notebooks required if using EMR Notebooks

 


when the --use-default-roles option is specified, the cluster is created using the MyCustomServiceRoleForEMR and MyCustomServiceRoleForClusterEC2Instances. By default, the config file specifies the default service_role as AmazonElasticMapReduceRole and the default instance_profile as EMR_EC2_DefaultRole.
* https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-iam-roles-custom.html

```json
[default]
output = json
region = us-west-1
aws_access_key_id = myAccessKeyID
aws_secret_access_key = mySecretAccessKey
emr =
     service_role = MyCustomServiceRoleForEMR
     instance_profile = MyCustomServiceRoleForClusterEC2Instances
```                


**Configure IAM Roles for EMRFS Request to Amazon S3 buckets**
When an application running on an Amazon EMR cluster references data using the s3://mydata format, Amazon EMR uses EMRFS to make the request. To interact with Amazon S3, EMRFS assumes the permissions policies attached to the Service Role for Cluster EC2 Instances (EC2 Instance Profile) specified when the cluster was created. The same service role for cluster EC2 instances is used regardless of the user or group using the application or the location of the data in Amazon S3. If you have clusters with multiple users who need different levels of access to data in Amazon S3 through EMRFS, you can set up a security configuration with IAM roles for EMRFS. EMRFS can assume a different service role for cluster EC2 instances based on the user or group making the request, or based on the location of data in Amazon S3. Each IAM role for EMRFS can have different permissions for data access in Amazon S3.
NOTE: IAM roles for EMRFS are available with Amazon EMR release version 5.10.0 and later. 

Use the following guidelines for the structure of the MyEmrFsSecConfig.json file. You can specify this structure along with structures for other security configuration options. For more information, see Create a Security Configuration.

The following is an example JSON snippet for specifying custom IAM roles for EMRFS within a security configuration. It demonstrates role mappings for the three different identifier types, followed by a parameter reference.
```json
{
  "AuthorizationConfiguration": {
    "EmrFsConfiguration": {
      "RoleMappings": [{
        "Role": "arn:aws:iam::123456789101:role/allow_EMRFS_access_for_user1",
        "IdentifierType": "User",
        "Identifiers": [ "user1" ]
      },{
        "Role": "arn:aws:iam::123456789101:role/allow_EMRFS_access_to_MyBuckets",
        "IdentifierType": "Prefix",
        "Identifiers": [ "s3://MyBucket/","s3://MyOtherBucket/" ]
      },{
        "Role": "arn:aws:iam::123456789101:role/allow_EMRFS_access_for_AdminGroup",
        "IdentifierType": "Group",
        "Identifiers": [ "AdminGroup" ]
      }]
    }
  }
}
```


|Parameter|Description |
|------|----------------------|
|`"AuthorizationConfiguration":`|Required|
|`"EmrFsConfiguration":`|Required. Contains role mappings.|
|`"RoleMappings":`|Required. Contains one or more role mapping definitions. Role mappings are evaluated in the top-down order that they appear. If a role mapping evaluates as true for an EMRFS call for data in Amazon S3, no further role mappings are evaluated and EMRFS uses the specified IAM role for the request. Role mappings consist of the following required parameters:|
|`"Role":`| Specifies the ARN identifier of an IAM role in the format arn:aws:iam::account-id:role/role-name. This is the IAM role that Amazon EMR assumes if the EMRFS request to Amazon S3 matches any of the Identifiers specified.|
|`"IdentifierType":`|one of the following: <br/> * `"User"` identifiers are one or more Hadoop users <br/> *  `"Prefix"` specifies that the identifier is an Amazon S3 location. e.g. `s3://mybucket/mydir` and `s3://mybucket/yetanotherdir.`   |
|`"Identifiers":`|Specifies one or more identifiers of the appropriate identifier type. Separate multiple identifiers by commas with no spaces.|


**Resource-Based Policies for Amazon EMR Access to AWS Glue Data Catalog**
AWS Glue supports resource-based policies to control access to Data Catalog resources. These resources include databases, tables, connections, and user-defined functions. For more information

When using resource-based policies to limit access to AWS Glue from within Amazon EMR, the principal that you specify in the permissions policy must be the role ARN associated with the EC2 instance profile that is specified when a cluster is created. For example, for a resource-based policy attached to a catalog, you can specify the role ARN for the default service role for cluster EC2 instances, `EMR_EC2_DefaultRole` as the Principal, using the format shown in the following example:

```json arn:aws:iam::acct-id:role/EMR_EC2_DefaultRole```

### 2. Data must be encrypted at all times using a CG BYOK for encryption where possible 

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

**Why?** Data encryption helps prevent unauthorized users from reading data on a cluster and associated data storate systems. This includes data saved to persistent media and data that may be intercepted as it travels the network. Data encryption helps prevent unauthorized users from reading data on a cluster and associated data storage systems. This includes data saved to persistent media, known as data at rest, and data that may be intercepted as it travels the network, known as data in transit.

**How?** AWS EC2 instances deployed into CG private subnets and allow for BYOK. 

**Use Security Configurations to Set Up Cluster Security**
 
Beginning with Amazon EMR version 4.8.0, you can use Amazon EMR security configurations to configure data encryption settings for clusters more easily. Security configurations offer settings to enable security for data in-transit and data at-rest in Amazon Elastic Block Store (Amazon EBS) volumes and EMRFS on Amazon S3. Each security configuration that you create is stored in Amazon EMR rather than in the cluster configuration, so you can easily reuse a configuration to specify data encryption settings whenever you create a cluster.
  
Syntax
To declare this entity in your AWS CloudFormation template, use the following syntax:

JSON
```json
"Type" : "AWS::EMR::SecurityConfiguration",
      "Properties" : {
          "Name" : String,
          "SecurityConfiguration" : Json
        }
    }
```
    
    
YAML
```ymal
Type: AWS::EMR::SecurityConfiguration
Properties: 
  Name: String
  SecurityConfiguration: Json
```
The example below illustrates the following scenario:
* In-transit data encryption is enabled with a custom key provider.
* CSE-Custom is used for Amazon S3 data.
* Local disk encryption uses a custom key provider.

```
aws emr create-security-configuration --name "MySecConfig" --security-configuration '{
    "EncryptionConfiguration": {
        "EnableInTransitEncryption" : "true",
        "EnableAtRestEncryption" : "true",
        "InTransitEncryptionConfiguration" : {
            "TLSCertificateConfiguration" : {
                "CertificateProviderType" : "Custom",
                "S3Object" : "s3://MyConfig/artifacts/MyCerts.jar", 
                "CertificateProviderClass" : "com.mycompany.MyCertProvider"
            }
        },
        "AtRestEncryptionConfiguration" : {
            "S3EncryptionConfiguration" : {
                "EncryptionMode" : "CSE-Custom",
                "S3Object" : "s3://MyConfig/artifacts/MyCerts.jar", 
                "EncryptionKeyProviderClass" : "com.mycompany.MyKeyProvider"
                },
            "LocalDiskEncryptionConfiguration" : {
                "EncryptionKeyProviderType" : "Custom",
                "S3Object" : "s3://MyConfig/artifacts/MyCerts.jar",
                "EncryptionKeyProviderClass" : "com.mycompany.MyKeyProvider"
            }
        }
    }
}'    
```
**JSON Reference for Encryption Settings**
The following table lists the JSON parameters for encryption settings and provides a description of acceptable values for each parameter.
|Parameter|Description |
|------|----------------------|
|"EnableInTransitEncryption" : true / false|Specify true to enable in-transit encryption and false to disable it. If omitted, false is assumed, and in-transit encryption is disabled.|
|"EnableAtRestEncryption" : true / false|Specify true to enable at-rest encryption and false to disable it. If omitted, false is assumed and at-rest encryption is disabled.|
|In-transit encryption parameters|
|"InTransitEncryptionConfiguration" :|Specifies a collection of values used to configure in-transit encryption when EnableInTransitEncryption is true.|
|"CertificateProviderType" : "PEM" | "Custom"|Specifies whether to use PEM certificates referenced with a zipped file, or a Custom certificate provider. If PEM is specified, S3Object must be a reference to the location in Amazon S3 of a zip file containing the certificates. If Custom is specified, S3Object must be a reference to the location in Amazon S3 of a JAR file, followed by a CertificateProviderClass entry.|
|"S3Object" : "ZipLocation" | "JarLocation"|Provides the location in Amazon S3 to a zip file when PEM is specified, or to a JAR file when Custom is specified. The format can be a path (for example, s3://MyConfig/artifacts/CertFiles.zip) or an ARN (for example, arn:aws:s3:::Code/MyCertProvider.jar). If a zip file is specified, it must contain files named exactly privateKey.pem and certificateChain.pem. A file named trustedCertificates.pem is optional.|
|"CertificateProviderClass" : "MyClassID"|Required only if Custom is specified for CertificateProviderType. `MyClassID` specifies a full class name declared in the JAR file, which implements the TLSArtifactsProvider interface. For example, com.mycompany.MyCertProvider.|
|**At-rest encryption parameters**|
|"AtRestEncryptionConfiguration" :|Specifies a collection of values for at-rest encryption when EnableAtRestEncryption is true, including Amazon S3 encryption and local disk encryption.|
|"EncryptionMode" : "SSE-S3" | "SSE-KMS" | "CSE-KMS" | "CSE-Custom"| Specifies the type of Amazon S3 encryption to use. If SSE-S3 is specified, no further Amazon S3 encryption values are required. If either SSE-KMS or CSE-KMS is specified, an AWS KMS customer master key (CMK) ARN must be specified as the AwsKmsKey value. If CSE-Custom is specified, S3Object and EncryptionKeyProviderClass values must be specified. |
|"AwsKmsKey" : `"MyKeyARN"`| Required only when either SSE-KMS or CSE-KMS is specified for EncryptionMode. `MyKeyARN` must be a fully specified ARN to a key (for example, arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012).|
|"S3Object" : `"JarLocation"`|Required only when CSE-Custom is specified for CertificateProviderType. `JarLocation` provides the location in Amazon S3 to a JAR file. The format can be a path (for example, s3://MyConfig/artifacts/MyKeyProvider.jar) or an ARN (for example, arn:aws:s3:::Code/MyKeyProvider.jar).|
|"EncryptionKeyProviderClass" : `"MyS3KeyClassID"`|Required only when CSE-Custom is specified for EncryptionMode. MyS3KeyClassID specifies a full class name of a class declared in the application that implements the EncryptionMaterialsProvider interface; for example, `com.mycompany.MyS3KeyProvider`.|
|Local disk encryption parameters|
|**Local disk encryption parameters**|
|"LocalDiskEncryptionConfiguration"|Specifies the key provider and corresponding values to be used for local disk encryption.|
|"EncryptionKeyProviderType" : "AwsKms" `|` "Custom"| Specifies the key provider. If AwsKms is specified, an AWS KMS CMK ARN must be specified as the AwsKmsKey value. If Custom is specified, S3Object and EncryptionKeyProviderClass values must be specified. |
|"AwsKmsKey" : `"MyKeyARN"`|Required only when AwsKms is specified for Type. _`MyKeyARN`_ must be a fully specified ARN to a key (for example, arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-456789012123). |
|"S3Object" : _`"JarLocation"`_|Required only when CSE-Custom is specified for CertificateProviderType. JarLocation provides the location in Amazon S3 to a JAR file. The format can be a path (for example, s3://MyConfig/artifacts/MyKeyProvider.jar) or an ARN (for example, arn:aws:s3:::Code/MyKeyProvider.jar). |
|"EncryptionKeyProviderClass" : _`"MyLocalDiskKeyClassID"`_|Required only when Custom is specified for Type. _`MyLocalDiskKeyClassID`_ specifies a full class name of a class declared in the application that implements the EncryptionMaterialsProvider interface; for example, _`com.mycompany.MyLocalDiskKeyProvider`_.|


**Encryption at Rest**
* EMR EBS volumes are encrypted using CG BYOK. 
* S3 Utility and Data buckets are encrypted using CG BYOK.
* EMR logs S3 is encrypted, but using AWS managed key. 

    **Amazon S3 Server-Side Encryption**

     Capital Group utilizes Amazon's S3 Server-Side Encryption. When you set up Amazon S3 server-side encryption, Amazon S3 encrypts data at the object level as it writes the data to disk and decrypts the data when it is accessed.

    You can choose between two different key management systems when you specify SSE in Amazon EMR:
    
    * SSE-S3 – Amazon S3 manages keys for you.
    * SSE-KMS – You use an AWS KMS customer master key (CMK) set up with policies suitable for Amazon EMR. 
    * SSE with customer-provided keys (SSE-C) is not available for use with Amazon EMR.
    
    **EBS Volume Encryption**
    If you create a cluster in a region where Amazon EC2 encryption of EBS volumes is enabled by default for your account, EBS volumes are encrypted even if local disk encryption is not enabled.
    
    The following options are available to encrypt EBS volumes using a security configuration:
    
    **EBS encryption – Beginning with Amazon EMR version 5.24.0, you can choose to enable EBS encryption. The EBS encryption option encrypts the EBS root device volume and attached storage volumes. The EBS encryption option is available only when you specify AWS Key Management Service as your key provider. We recommend using EBS encryption.
    
    LUKS encryption – If you choose to use LUKS encryption for Amazon EBS volumes, the LUKS encryption applies only to attached storage volumes, not to the root device volume. For more information about LUKS encryption, see the LUKS on-disk specification.**
    
    For your key provider, you can set up an AWS KMS customer master key (CMK) with policies suitable for Amazon EMR, or a custom Java class that provides the encryption artifacts. When you use AWS KMS, charges apply for the storage and use of encryption keys.




**Encryption in Transit**

All communication between EMR cluster nodes is encrypted via use of TLS custom certificate. rsa:4096  

**Use a Customer Key Provider for Amazon S3 Encryption wiht EMRFS**
When Amazon EMR fetches the encryption materials from the EncryptionMaterialsProvider class to perform encryption, EMRFS optionally populates the materialsDescription argument with two fields: the Amazon S3 URI for the object and the JobFlowId of the cluster, which can be used by the EncryptionMaterialsProvider class to return encryption materials selectively.


```java  
import com.amazonaws.services.s3.model.EncryptionMaterials;
import com.amazonaws.services.s3.model.EncryptionMaterialsProvider;
import com.amazonaws.services.s3.model.KMSEncryptionMaterials;
import org.apache.hadoop.conf.Configurable;
import org.apache.hadoop.conf.Configuration;

import java.util.Map;

//Provides KMSEncryptionMaterials according to Configuration

public class MyEncryptionMaterialsProviders implements EncryptionMaterialsProvider, Configurable{
  private Configuration conf;
  private String kmsKeyId;
  private EncryptionMaterials encryptionMaterials;

  private void init() {
    this.kmsKeyId = conf.get("my.kms.key.id");
    this.encryptionMaterials = new KMSEncryptionMaterials(kmsKeyId);
  }

  @Override
  public void setConf(Configuration conf) {
    this.conf = conf;
    init();
  }

  @Override
  public Configuration getConf() {
    return this.conf;
  }

  @Override
  public void refresh() {

  }

  @Override
  public EncryptionMaterials getEncryptionMaterials(Map<String, String> materialsDescription) {
    return this.encryptionMaterials;
  }

  @Override
  public EncryptionMaterials getEncryptionMaterials() {
    return this.encryptionMaterials;
  }
}
```

### 3. Infrastructure must be secured for monitoring, audit, availability, and access control
NIST CSF:
|NIST Subcategory Control|Description|
|-----------|------------------------|
|PR.PT-1|Audit/log records are determined, documented, implemented, and reviewed in accordance with policy|
|PR.PT-4|Communications and control networks are protected|
|PR.PT-5|Mechanisms (e.g., failsafe, load balancing, hot swap) are implemented to achieve resilience requirements in normal and adverse situations|
|PR.AC-3|Remote access is managed|
|PR.AC-5|Network integrity is protected (e.g., network segregation, network segmentation)|
|PR.DS-4|Adequate capacity to ensure availability is maintained|

Capital Group:
|Control Statement|Description|
|------|----------------------|
|6|Any AWS service used by CG should not be directly available to the Internet and the default route is always the CG gateway.|
#### 3.1. Configuring VPC Endpoints for EMR
**Why?** You can improve the security posture of your VPC by configuring Amazon EMR to use an interface VPC endpoint. Interface endpoints are powered by AWS PrivateLink. PrivateLink restricts all network traffic between your VPC and Amazon EMR to the Amazon network. You don't need an internet gateway, a NAT device, or a virtual private gateway. The instances in your VPC don't need public IP addresses to communicate with the Amazon EMR API. Capital Group VPCs communication occurs through the tranist gateway. 

**How?** To use Amazon EMR through your VPC, you must connect from an instance that is inside the VPC or connect your private network to your VPC by using an Amazon Virtual Private Network (VPN) or AWS Direct Connect. 

After you create an interface VPC endpoint, if you enable private DNS hostnames for the endpoint, the default Amazon EMR endpoint resolves to your VPC endpoint. The default service name endpoint for Amazon EMR is in the following format.

      elasticmapreduce.Region.amazonaws.com
If you do not enable private DNS hostnames, Amazon VPC provides a DNS endpoint name that you can use in the following format.

    VPC_Endpoint_ID.elasticmapreduce.Region.vpce.amazonaws.com

Additionally Amazon EMR supports making calls to all of its API Actions inside your VPC.

**Create a VPC Endpoint Policy for Amazon EMR**
You can create a policy for Amazon VPC endpoints for Amazon EMR to specify the following:
  * The principal that can or cannot perform actions
  * The actions that can be performed
  * The resources on which actions can be performed 
<br>https://docs.aws.amazon.com/emr/latest/ManagementGuide/interface-vpc-endpoint.html 

Example – VPC Endpoint Policy to Deny All Access From a Specified AWS Account. e.g. '123456789012`
```json
{
    "Statement": [
        {
            "Action": "*",
            "Effect": "Allow",
            "Resource": "*",
            "Principal": "*"
        },
        {
            "Action": "*",
            "Effect": "Deny",
            "Resource": "*",
            "Principal": {
                "AWS": [
                    "123456789012"
                ]
            }
        }
    ]
}
```

Example – VPC Endpoint Policy to Allow VPC Access Only to a Specified IAM Principal (User), eg.g IAM user 'lijuan' for AWS account '123456789012`

```json
{
    "Statement": [
        {
            "Action": "*",
            "Effect": "Allow",
            "Resource": "*",
            "Principal": {
                "AWS": [
                    "arn:aws:iam::123456789012:user/lijuan"
                ]
            }
        }]
}
```
Example – VPC Endpoint Policy to Allow Read-Only EMR Operations
The actions specified provide the equivalent of read-only access for Amazon EMR. All other actions on the VPC are denied for the specified account.
```json
{
    "Statement": [
        {
            "Action": [
                "elasticmapreduce:DescribeSecurityConfiguration",
                "elasticmapreduce:GetBlockPublicAccessConfiguration",
                "elasticmapreduce:ListBootstrapActions",
                "elasticmapreduce:ViewEventsFromAllClustersInConsole",
                "elasticmapreduce:ListSteps",
                "elasticmapreduce:ListInstanceFleets",
                "elasticmapreduce:DescribeCluster",
                "elasticmapreduce:ListInstanceGroups",
                "elasticmapreduce:DescribeStep",
                "elasticmapreduce:ListInstances",
                "elasticmapreduce:ListSecurityConfigurations",
                "elasticmapreduce:DescribeEditor",
                "elasticmapreduce:ListClusters",
                "elasticmapreduce:ListEditors"            
            ],
            "Effect": "Allow",
            "Resource": "*",
            "Principal": {
                "AWS": [
                    "123456789012"
                ]
            }
        }
    ]
}
```

Example – VPC Endpoint Policy Denying Access to a Specified Cluster
The following VPC endpoint policy allows full access for all accounts and principals, but denies any access for AWS account 123456789012 to actions performed on the Amazon EMR cluster with cluster ID j-A1B2CD34EF5G. Other Amazon EMR actions that don't support resource-level permissions for clusters are still allowed. For a list of Amazon EMR actions and their corresponding resource type, see Actions, Resources, and Condition Keys for Amazon EMR. See Endnote #4

```json
{
    "Statement": [
        {
            "Action": "*",
            "Effect": "Allow",
            "Resource": "*",
            "Principal": "*"
        },
        {
            "Action": "*",
            "Effect": "Deny",
            "Resource": "arn:aws:elasticmapreduce:us-west-2:123456789012:cluster/j-A1B2CD34EF5G",
            "Principal": {
                "AWS": [
                    "123456789012"
                ]
            }
        }
    ]
}
```



#### 3.2. Enable SSH Access to EMR Cluster Instances 
**Why?** 
Amazon EMR cluster nodes run on Amazon EC2 instances. You can connect to cluster nodes in the same way that you can connect to Amazon EC2 instances. When you create a cluster, you can specify the Amazon EC2 key pair that will be used for SSH connections to all cluster instances. 

* You can also create a cluster without a key pair. This is usually done with <b><i>transient clusters</i></b> that start, run steps, and then terminate automatically. https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-plan-access-ssh.html 

**How?** 
EMR cluster consists of EC2 instances deployed in CG Private subsets in CDD AWS accounts. Every cluster has its SSH key, and this key is secured using AWS secrets manager. This key is introduced into AWS Secrets manager via Automation

* <b>Capital Group Control</b>: Interactive access to the cluster is via SSH key vaulted in CyberArk or Secrets manager. User access to these keys is limited based on user role managed in the CG User domain (AD).  

#### 3.2. Enable Logging to Cloudtrails & Cloudtrail to Splunk 
* Side Note -- Loggers 
  1. Cloudtrail - EMR API Calls 
  2. VPC - VPC Flow Logs
  3. EMRFS/S3 - S3 Access Logs

**Why?** AWS CloudTrail, a service that provides a record of actions taken by a user, role, or an AWS service, is integrated with Amazon EMR. CloudTrail captures all API calls for Amazon EMR as events. The calls captured include calls from the Amazon EMR console and code calls to the Amazon EMR API operations. If you create a trail, you can enable continuous delivery of CloudTrail events to an Amazon S3 bucket, including events for Amazon EMR.

* You can also audit the S3 objects that EMR accesses by using S3 access logs. AWS CloudTrail provides logs only for AWS API calls. Thus, if a user runs a job that reads and writes data to S3, the S3 data that was accessed by EMR doesn't show up in CloudTrail. By using S3 access logs, you can comprehensively monitor and audit access against your data in S3 from anywhere, including EMR.

**How?**
Every event or log entry contains information about who generated the request. The identity information helps you determine the following:

* Whether the request was made with root or AWS Identity and Access Management (IAM) user credentials.
* Whether the request was made with temporary security credentials for a role or federated user.
* Whether the request was made by another AWS service.

A trail is a configuration that enables delivery of events as log files to an Amazon S3 bucket that you specify. CloudTrail log files contain one or more log entries. An event represents a single request from any source and includes information about the requested action, the date and time of the action, request parameters, and so on. CloudTrail log files aren't an ordered stack trace of the public API calls, so they don't appear in any specific order.

The following example shows a CloudTrail log entry that demonstrates the RunJobFlow action.

```json
{
    "Records": [
    {
         "eventVersion":"1.01",
         "userIdentity":{
            "type":"IAMUser",
            "principalId":"EX_PRINCIPAL_ID",
            "arn":"arn:aws:iam::123456789012:user/temporary-user-xx-7M",
            "accountId":"123456789012",
            "userName":"temporary-user-xx-7M"
         },
         "eventTime":"2018-03-31T17:59:21Z",
         "eventSource":"elasticmapreduce.amazonaws.com",
         "eventName":"RunJobFlow",
         "awsRegion":"us-west-2",
         "sourceIPAddress":"192.0.2.1",
         "userAgent":"aws-sdk-java/unknown-version Linux/xx Java_HotSpot(TM)_64-bit_Server_VM/xx",
         "requestParameters":{
            "tags":[
               {
                  "value":"prod",
                  "key":"domain"
               },
               {
                  "value":"us-west-2",
                  "key":"realm"
               },
               {
                  "value":"VERIFICATION",
                  "key":"executionType"
               }
            ],
            "instances":{
               "slaveInstanceType":"m5.xlarge",
               "ec2KeyName":"emr-integtest",
               "instanceCount":1,
               "masterInstanceType":"m5.xlarge",
               "keepJobFlowAliveWhenNoSteps":true,
               "terminationProtected":false
            },
            "visibleToAllUsers":false,
            "name":"MyCluster",
            "ReleaseLabel":"emr-5.16.0"
         },
         "responseElements":{
            "jobFlowId":"j-2WDJCGEG4E6AJ"
         },
         "requestID":"2f482daf-b8fe-11e3-89e7-75a3d0e071c5",
         "eventID":"b348a38d-f744-4097-8b2a-e68c9b424698"
      },
    ...additional entries
  ]
}
```

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
|RS.AN-5|Processes are established to receive, analyze and respond to vulnerabilities disclosed to the organization from internal and external sources (e.g. internal testing, security bulletins, or security researchers)|  
**Why?** Using Amazon EventBridge, you can automatically report and respond to incidents, including actions taken on worker nodes, or EMR API Calls via CloudTrail.

**How?** For various incident types, an appropriate response rule should be determined and added to EventBridge. Depending on the organization's Incident Response Plan, there may be need for human interaction in some cases, or fully automated remediation in other cases.


## Endnotes
### Endnote 1 <!-- omit in toc -->
https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-iam-roles.html
### Endnote 2 <!-- omit in toc -->
https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-specify-security-configuration.html
### Endnote 3 <!-- omit in toc -->
https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html
### Endnote 4 <!-- omit in toc -->
https://docs.aws.amazon.com/IAM/latest/UserGuide/list_amazonelasticmapreduce.html

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