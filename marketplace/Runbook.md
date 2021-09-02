<img src="https://a0.awsstatic.com/libra-css/images/logos/aws_logo_smile_1200x630.png" alt="AWS" width="250"/>

# AWS Private Marketplace - Security Playbook <!-- omit in toc -->
## Capgroup Cybersecurity Control Alignment <!-- omit in toc -->

**Generated By:**  
[Rob Goss (RMG)](https://cgweb3/profile/RMG)
<br>
Security Engineering

**Last Update:** *07/28/2021*

## Table of Contents <!-- omit in toc -->
- [Overview](#overview)
- [Cloud Security Requirements](#Cloud-Security-Requirements)
  - [1. Private Marketplace administrative rights only assigned to Platform Design Team](#1-Private-Marketplace-administrative-rights-only-assigned-to-Platform-Design-Team)
  - [2. IAM Policy Enforces Least Priviledge for Private Marketplace Resources](#2-IAM-Policy-Enforces-Least-Priviledge-for-Private-Marketplace-Resources)
  - [3. CloudTrail logging enabled for Private Marketplace](#3-CloudTrail-logging-enabled-for-Private-Marketplace)
- [Other Operational Expectations](#Other-Operational-Expectations)
  - [1. Private Marketplace Resources are tagged according to CG standards](#1-Private-Marketplace-Resources-are-tagged-according-to-CG-standards)
- [Endnotes](#Endnotes)
- [Capital Group Glossory](#Capital-Group-Glossory) 


## Overview
Private marketplace controls which products users in your AWS account are able to access. As such business users and engineering teams might be able to procure only certain items from AWS Marketplace based on their role. It is built on top of AWS Marketplace, and enables your administrators to create and customize curated digital catalogs of approved independent software vendors (ISVs) and products that conform to their in-house policies. Users in your AWS account can find, buy, and deploy approved products from your private marketplace, and ensure that all available products comply with your organization’s policies and standards.

Your private marketplace is shared across your organization. With AWS Organizations, you can create a series of accounts linked for permissions and payments. You can create one or more private marketplace experiences that are associated with one or more accounts in your organization, each with its own set of approved products. Your administrators can also apply company branding to each private marketplace experience with your company or team’s logo, messaging, and color scheme.

A private marketplace provides you with a broad catalog of products available in AWS Marketplace to choose from, in addition to fine-grained control of those products.

<img src="/docs/img/marketplace/example.png" width="800"><br>
<br><br>

## Cloud Security Requirements
<img src="/docs/img/Prevent.png" width="50">

### 1. Private Marketplace administrative rights only assigned to Platform Design Team

**Capital Group:** <br>

|Control Statement|Description|
|------|----------------------|
|[CS0012299](https://capitalgroup.service-now.com/cg_grc?id=cg_grc_action_item_details&sys_id=44df48c01bac20506a50beef034bcb34&table=sn_compliance_policy_statement&view=sp)|Access to change cloud resource-based access policies is restricted to authorized personnel. |
<br>

**Why?** <br>
As AWS Private Marketplace provides access for users to provision many types of resources, which may be either costly or possibly insecure, the permissions to add new catalogue items for teams to provision will be limited to only the Platform Design Team at this time.
<br><br>

**How?** <br>
To create a role for the PDS Team, simply attach the `AWSMarketplaceProcurementSystemAdminFullAccess` policy to the appropriate IAM identities.

This policy grants admin permissions that allow managing all aspects of an AWS Marketplace eProcurement integration, including listing the accounts in your organization. For more information about eProcurement integrations, see Integrating AWS Marketplace with procurement systems .

#### Permissions Details:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "aws-marketplace:PutProcurementSystemConfiguration",
                "aws-marketplace:DescribeProcurementSystemConfiguration",
                "organizations:Describe*",
                "organizations:List*"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```
<br>

### 2. IAM Policy Enforces Least Priviledge for Private Marketplace Resources

**Capital Group:** <br>

|Control Statement|Description|
|------|----------------------|
|Control Definition Needed|Control Definition Description Needed|
<br>

**Why?** <br>
A core tenet of CG's cloud migration, and security strategy is the principal of least priviledge. This means providing users and groups the least ammount of access needed to perform the tasks that they need to be successful. In this case, with private marketplace teams will only be presented an `experience` with items that they have been pre-approved to install or provision.
<br><br>

**How?** <br>
The implementation and maintenance of permissions on a per account basis will be delivered via the CG `Broad-Access-Role` in each account. Below is the default set of permissions that will be added as a baseline.

#### Permissions Detail:
1. **PDS Team Admin Role**<br>
   This policy will allow user to create request for any unapproved product.
   ```
   {
      "Version": "2012-10-17",
      "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "aws-marketplace:CreatePrivateMarketplaceRequests",
                "aws-marketplace:ListPrivateMarketplaceRequests",
                "aws-marketplace:DescribePrivateMarketplaceRequests"
            ],
            "Resource": "*"
        }
     ]
   }
   ```

2. **Broad-Access-Role**<br>
This policy Provides the ability to subscribe and unsubscribe from approved AWS Marketplace software.
   ```
   {
	"Version": "2012-10-17",
	"Statement": [
			{
					"Action": [
							"aws-marketplace:ViewSubscriptions",
							"aws-marketplace:Subscribe",
							"aws-marketplace:Unsubscribe",
							"aws-marketplace:CreatePrivateMarketplaceRequests",
							"aws-marketplace:ListPrivateMarketplaceRequests",
							"aws-marketplace:DescribePrivateMarketplaceRequests",
							"aws-marketplace:StartBuild",
							"aws-marketplace:ListBuilds"
					],
					"Effect": "Allow",
					"Resource": "*"
			}
	   ]
   }
   ```

<br>

### 3. CloudTrail logging enabled for Private Marketplace

**Capital Group:** <br>

|Control Statement|Description|
|------|----------------------|
|Control Definition Needed|Control Definition Description Needed|

**What, Why & How?**

The Marketplace direct APIs service is integrated with AWS CloudTrail. CloudTrail is a service that provides a record of actions taken by a user, role, or an AWS service in the Marketplace direct APIs. CloudTrail captures multiple items including StartChangeSet, DescribeChangeSet and ListChange Set calls for the Marketplace APIs as events. You can use the information collected by CloudTrail to determine the request that was made to the Marketplace APIs, the IP address from which the request was made, who made the request, when it was made, and additional details.

- A `default trail` should have been enabled through automation to allow for the continuous delivery of CloudTrail events to an Amazon Simple Storage Service (Amazon S3) bucket, including events for the Marketplace APIs. This will enable the forwarding of logs into Splunk for long term archival and reporting.
- Creation of non-default Cloud Trails should be avoided where possible as all EBS data should be logged and monitored though the aforementioned default trail.
<br><br>

## Other Operational Expectations
<img src="/docs/img/Operations.png" width="50">

### 1. Private Marketplace Resources are tagged according to CG standards

**Capital Group:** <br>

|Control Statement|Description|
|------|----------------------|
|N/A| No security control currently defined.|

**What, Why & How?**

Tagging resources in the cloud is an easy way for teams to provide information related to who owns the resource, what the resource is used for, as well as other important information related to the deployment lifecycle of the resource. CG has mandated that all cloud resources are to be tagged with certain important for cross-team use. Although most of the mandatory tags will be added through automation, one should still check to make sure that all newly deployed recources have the appropriate tags attached. please see the documentation below for the latest tagging standards.

[CG Cloud Tagging Strategy](https://confluence.capgroup.com/display/HCEA/Resource+Tagging+standards)
<br><br>

## Endnotes
**Resources**<br>
1. https://docs.aws.amazon.com/marketplace/latest/buyerguide/private-marketplace.html
2. https://docs.aws.amazon.com/marketplace-catalog/latest/api-reference/logging-api-calls-with-cloudtrail.html
3. [PDS Team - AWS Marketplace Documentation](https://confluence.capgroup.com/pages/viewpage.action?spaceKey=HCEA&title=AWS+Private+Marketplace)
<br><br>

## Capital Group Glossory 
**Data** - Digital pieces of information stored or transmitted for use with an information system from which understandable information is derived. Items that could be considered to be data are: Source code, meta-data, build artifacts, information input and output.  
 
**Information System** - An organized assembly of resources and procedures for the collection, processing, maintenance, use, sharing, dissemination, or disposition of information. All systems, platforms, compute instances including and not limited to physical and virtual client endpoints, physical and virtual servers, software containers, databases, Internet of Things (IoT) devices, network devices, applications (internal and external), Serverless computing instances (i.e. AWS Lambda), vendor provided appliances, and third-party platforms, connected to the Capital Group network or used by Capital Group users or customers.

**Log** - a record of the events occurring within information systems and networks. Logs are composed of log entries; each entry contains information related to a specific event that has occurred within a system or network.

**Information** - communication or representation of knowledge such as facts, data, or opinions in any medium or form, including textual, numerical, graphic, cartographic, narrative, or audiovisual. 

**Cloud computing** - A model for enabling ubiquitous, convenient, on-demand network access to a shared pool of configurable computing resources (e.g., networks, servers, storage, applications, and services) that can be rapidly provisioned and released with minimal management effort or service provider interaction.

**Vulnerability**  - Weakness in an information system, system security procedures, internal controls, or implementation that could be exploited or triggered by a threat source. Note: The term weakness is synonymous for deficiency. Weakness may result in security and/or privacy risks.