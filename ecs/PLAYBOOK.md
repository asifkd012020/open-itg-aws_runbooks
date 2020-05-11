# Amazon Elastic Container Service (ECS)
AWS provides a number of security features for Amazon Elastic Container Service (ECS) which help you comply with the NIST Cybersecurity Framework. The following playbook will outline what the AWS best practices are, how they align to NIST, and how to implement these best practices within your organization.
These NIST Controls and Subcategories are not applicable to this service:  PR.AT, PR.MA, PR.IP  (Unless stated), PR.AC-2, PR.AC-3, PR.DS-3, PR.DS-8, PR.PT-2, PR.PT-5, DE.DP1, DE.DP-2. DE.DP-3, DE.CM-3, DE.AE-5, RC, RS.MI.   

## DISCLAIMER
> The following applies to this document and all other documents, information, data, and responses (written or verbal) provided by Amazon Web Services, Inc. or any of its affiliates (collectively, "AWS") in connection with responding to this request and other related requests (collectively, this "Response"): This Response is expressly (a) informational only and provided solely for discussion purposes, (b) non-binding and not an offer to contract that can be accepted by any party, (c) provided "as is" with no representations or warranties whatsoever, and (d) based on AWS's current knowledge and may change at any time due to a variety of factors such as changes to your requirements or changes to AWS's service offerings. All obligations must be set forth in a separate, definitive written agreement between the parties. Neither party will have any liability for any failure or refusal to enter into a definitive agreement. All use of AWS's service offerings will be governed by the AWS Customer Agreement available at http://aws.amazon.com/agreement/ (or other definitive written agreement between the parties governing the use of AWS's service offerings) (as applicable, the "Agreement"). If the parties have an applicable Nondisclosure Agreement ("NDA"), then the NDA will apply to all Confidential Information (as defined in the NDA) disclosed in connection with this Response. AWS's pricing is publicly available and subject to change in accordance with the Agreement. Pricing information (if any) provided in this Response is only an estimate and is expressly not a binding quote. Fees and charges will be based on actual usage of AWS services, which may vary from the estimates provided. Nothing in this Response will modify or supplement the terms of the Agreement or the NDA. No part of this Response may be disclosed without AWS's prior written consent.  

## PREVENTATIVE CONTROLS  
### IMPLEMENT LEAST PRIVILEGE IAM ROLES FOR TASKS
| NIST Subcategory Control | Description              |
| ------------------------ | ------------------------ |
|PR.AC-1|Identities and credentials are issued, managed, verified, revoked, and audited for authorized devices, users and processes|
|PR.AC-4|Access permissions and authorizations are managed, incorporating the principles of least privilege and separation of duties|
|PR.AC-7|Users, devices, and other assets are authenticated (e.g., single-factor, multi-factor) commensurate with the risk of the transaction (e.g., individuals’ security and privacy risks and other organizational risks)|
|PR.PT-3|The principle of least functionality is incorporated by configuring systems to provide only essential capabilities|

#### Why?
With IAM roles for Amazon ECS tasks, you can specify an IAM role that can be used by the containers in a task. Having overly permissive IAM Roles for Tasks can lead to unauthorized access to resources not in scope of your application. 

#### How 	
Evaluate task to determine required access. Create a role with an attached policy that grants only that level of access. Review these roles and policies on a regular basis (at least annually).

## TODO: AWS TEAM TO COMPLETE
