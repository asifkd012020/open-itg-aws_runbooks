# AWS Data Pipeline - Security Playbook <!-- omit in toc -->
## NIST Cybersecurity Framework Alignment <!-- omit in toc -->

## Table of Contents <!-- omit in toc -->
- [Overview](#overview)
- [Preventative Controls](#preventative-controls)
  - [1. Use VPC Endpoints to restrict traffic to the CG Network](#1-use-vpc-endpoints-to-restrict-traffic-to-the-cg-network)
  - [2. Restrict Access to Deny Regions Outside US](#2-Restrict-Access-to-Deny-Regions-Outside-US)
- [Respond/Recover](#respondrecover)
- [Endnotes](#endnotes)
- [Capital Group Glossory](#capital-group-glossory)
- [Capital Group Control Statements](#capital-group-control-statements)

## Overview
AWS Auto Scaling monitors your applications and automatically adjusts capacity to maintain steady, predictable performance at the lowest possible cost.

AWS Auto Scaling is best viewed as an orchestrator, and has minimal inherent capabilities to be managed. 

<br>
## Preventative Controls
<img src="/docs/img/Prevent.png" width="50">

### 1. Use VPC Endpoints to restrict traffic to the CG Network
AWS DataPipeline as with many AWS PaaS services by nature do not usually allow for the service to be built within a Private VPC, and therefore Identity and Access Management (IAM) controls may be the only true option for securing access to the service and the data stored within.

### 1. Restrict Access to Deny Regions Outside US region
 *Policy resticts the access to Deny any resources outside of US Region*
 ```
 {
            "Sid": "DenyAllOutsideUS",
            "Effect": "Deny",
            "NotAction": [
                "support:*",
                "sts:*"
            ],
            "Resource": "*",
            "Condition": {
                "StringNotEquals": {
                    "aws:RequestedRegion": [
                        "us-west-1",
                        "us-west-2",
                        "us-east-1",
                    ]
                },
                "StringNotLike": {
                    "aws:PrincipalArn": [
                        "arn:aws:iam::*:role/OrganizationAccountAccessRole"
                    ]
                }
            }
        },
 ```
<br>

**Capital Group:** <br>

|Control Statement|Description|
|------|----------------------|
|7|Use of AWS IAM accounts are restricted to CG networks.|
<br>

**Why?**<br>
AutoScaling should be accessed directly from Capital Group VPC's to minimize public network traffic and public egress, reducing the risk of connectivity loss or latency

<br>

**How?**<br>
You can create a VPC endpoint for the AWS Auto Scaling service using either the Amazon VPC console or the AWS Command Line Interface (AWS CLI). Create an endpoint for AWS Auto Scaling using the following service name:

`com.amazonaws.{region}.autoscaling-plans` — Creates an endpoint for the AWS Auto Scaling API operations.

For more information, see Creating an interface endpoint in the Amazon VPC User Guide.

Enable private DNS for the endpoint to make API requests to the supported service using its default DNS hostname (for example, autoscaling-plans.us-east-1.amazonaws.com). When creating an endpoint for AWS services, this setting is enabled by default. For more information, see Accessing a service through an interface endpoint in the Amazon VPC User Guide.

You do not need to change any AWS Auto Scaling settings. AWS Auto Scaling calls other AWS services using either service endpoints or private interface VPC endpoints, whichever are in use.

## Respond/Recover
N/A - Not a resource unto itself

## Endnotes
**Resources**<br>
1. https://docs.aws.amazon.com/datapipeline/latest/DeveloperGuide/dp-example-tag-policies.html
2. https://aws.amazon.com/blogs/aws/category/aws-data-pipeline/

<br>

## Capital Group Glossory 
**Data** - Digital pieces of information stored or transmitted for use with an information system from which understandable information is derived. Items that could be considered to be data are: Source code, meta-data, build artifacts, information input and output.  
 
**Information System** - An organized assembly of resources and procedures for the collection, processing, maintenance, use, sharing, dissemination, or disposition of information. All systems, platforms, compute instances including and not limited to physical and virtual client endpoints, physical and virtual servers, software containers, databases, Internet of Things (IoT) devices, network devices, applications (internal and external), Serverless computing instances (i.e. AWS Lambda), vendor provided appliances, and third-party platforms, connected to the Capital Group network or used by Capital Group users or customers.

**Log** - a record of the events occurring within information systems and networks. Logs are composed of log entries; each entry contains information related to a specific event that has occurred within a system or network.

**Information** - communication or representation of knowledge such as facts, data, or opinions in any medium or form, including textual, numerical, graphic, cartographic, narrative, or audiovisual. 

**Cloud computing** - A model for enabling ubiquitous, convenient, on-demand network access to a shared pool of configurable computing resources (e.g., networks, servers, storage, applications, and services) that can be rapidly provisioned and released with minimal management effort or service provider interaction.

**Vulnerability**  - Weakness in an information system, system security procedures, internal controls, or implementation that could be exploited or triggered by a threat source. Note: The term weakness is synonymous for deficiency. Weakness may result in security and/or privacy risks.


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