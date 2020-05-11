# Amazon Elastic Container Service (ECS)
AWS provides a number of security features for Amazon Elastic Container Service (ECS) which help you comply with the NIST Cybersecurity Framework. The following playbook will outline what the AWS best practices are, how they align to NIST, and how to implement these best practices within your organization.
These NIST Controls and Subcategories are not applicable to this service:  PR.AT, PR.MA, PR.IP  (Unless stated), PR.AC-2, PR.AC-3, PR.DS-3, PR.DS-8, PR.PT-2, PR.PT-5, DE.DP1, DE.DP-2. DE.DP-3, DE.CM-3, DE.AE-5, RC, RS.MI.   

## DISCLAIMER
> The following applies to this document and all other documents, information, data, and responses (written or verbal) provided by Amazon Web Services, Inc. or any of its affiliates (collectively, "AWS") in connection with responding to this request and other related requests (collectively, this "Response"): This Response is expressly (a) informational only and provided solely for discussion purposes, (b) non-binding and not an offer to contract that can be accepted by any party, (c) provided "as is" with no representations or warranties whatsoever, and (d) based on AWS's current knowledge and may change at any time due to a variety of factors such as changes to your requirements or changes to AWS's service offerings. All obligations must be set forth in a separate, definitive written agreement between the parties. Neither party will have any liability for any failure or refusal to enter into a definitive agreement. All use of AWS's service offerings will be governed by the AWS Customer Agreement available at http://aws.amazon.com/agreement/ (or other definitive written agreement between the parties governing the use of AWS's service offerings) (as applicable, the "Agreement"). If the parties have an applicable Nondisclosure Agreement ("NDA"), then the NDA will apply to all Confidential Information (as defined in the NDA) disclosed in connection with this Response. AWS's pricing is publicly available and subject to change in accordance with the Agreement. Pricing information (if any) provided in this Response is only an estimate and is expressly not a binding quote. Fees and charges will be based on actual usage of AWS services, which may vary from the estimates provided. Nothing in this Response will modify or supplement the terms of the Agreement or the NDA. No part of this Response may be disclosed without AWS's prior written consent.  

## PREVENTATIVE CONTROLS  
### IMPLEMENT LEAST PRIVILEGE IAM ROLES FOR TASKS

#### NIST CSF 
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

## DETECTIVE CONTROLS
#### NIST CSF
| NIST Subcategory Control | Description              |
| ------------------------ | ------------------------ |
|DE.AE-1|A baseline of network operations and expected data flows for users and systems is established and managed|
|DE.CM-1|The network is monitored to detect potential cybersecurity events|
|DE.AE-3|Event data are aggregated and correlated from multiple sources and sensors|
|DE.AE-4|Impact of events is determined|

#### Why?
You can configure your container instances to send log information to CloudWatch Logs. This enables your security team the ability to monitor, store and access different logs from your various container instances in one convenient location. You can use ECS logs as a security tool to monitor the traffic that is reaching your containers. Once configured the logs can be utilized to establish network baselines, help determine event impacts and detect anomalous activity.  

#### How?
By enabling the awslogs Log Driver and specifying it in your containers task definitions. [link stuff here](link stuff here)

## RESPOND/RECOVER
### UTILIZE AMAZON ECS EVENTS AND EVENTBRIDGE 
#### NIST CSF 
| NIST Subcategory Control | Description              |
| ------------------------ | ------------------------ |
|RS.CO-3|Information is shared consistent with response plans|
|RS.AN-2|The impact of the incident is understood|

#### Why?
Amazon EventBridge enables you to automate your AWS services and respond automatically to system events such as application availability issues or resource changes. Events from AWS services are delivered to EventBridge in near real time. You can write simple rules to indicate which events are of interest to you and what automated actions to take when an event matches a rule.

#### How?
You can use Amazon ECS events for EventBridge to receive near real-time notifications regarding the current state of your Amazon ECS clusters. If your tasks are using the Fargate launch type, you can see the state of your tasks. If your tasks are using the EC2 launch type, you can see the state of both the container instances and the current state of all tasks running on those container instances. For services, you can see events related to the health of your service. Using EventBridge, you can build custom schedulers on top of Amazon ECS that are responsible for orchestrating tasks across clusters and monitoring the state of clusters in near real time.