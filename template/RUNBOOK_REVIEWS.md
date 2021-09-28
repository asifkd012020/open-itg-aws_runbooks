# Runbook Reviews
Runbooks are the 
<br><br>

## **1. CG Core Cloud Security Tenets**
Since CG's first cloud deployment, we have always been striving to enforce certain security requirements as they are intrinsic to security of the cloud at its core. These core requirements or tenets are as follows: Prevention of Public Access, Encryption of data at rest and in-transit and finally Identity and Access Management. Adherence to these tenets will in most cases provide a good baseline level of protections against most common cloud security problems. 

When reviewing a new or existing runbook and the associated `mapping.json` file, the review should initialy focus on the requirements and instructions therin as they relate to the core tenets. If these are not fully met, any new runbook should not be marked as `Service Production Ready` until there are requirements pertaining to all three tenets. 

Each of the three core tenets are further outlined below:

## 1. Public Access Prevention

Public Access prevention is arguably the most crucial item to get right first time, and at CG is  defined as:

No direct Public Access outside of Digital, Where Public Access is defined as direct communications inbound to AWS from outside CG's network which do not first traverse CG's public internet edge security complex in CXDCs. No direct Public Access outside of specifically approved instances (eg. Digital). This shall be done in one of three ways (`in order of preference`).

 1. *The resource is `VPC attached` with `no public ip address` assigned*
 2. *Resource policies limit the resource to only being accessible through a `VPC Endpoint`*
 3. *Resource policies limit source addresses to only `CG approved internal addresses`*
 4. *For any other scenarios, runbooks must define alternate mitigations or approaches to limit the exposure*

## 2. Encryption
Encryption at rest and in-transit of all cloud data is fundamental to securing cloud computing environmets, and is defined at CG as:

 1. *All data stored `at rest` in the cloud is encrypted using a CG managed `Customer Master Key (CMK)`*
 2. *All data in transit is encrypted with `TLS 1.2` or newer*

## 3. Identity & Access Management
Identity & Access Management is the third pillar of cloud security at CG, and is usually one of the most consistant controls that can be leveraged in the security of cloud environments and at CG is defined as:

 1. *Identity & Access Management (IAM) follows a least privilege model, meaning all cloud service to service roles only include the `permissions necessary` for the role to function. Permissions wildcards are never acceptable in any cloud environment*
 2. *All cloud secrets, api keys and other sensitive credential information will be stored in a CG approved `secrets management tool`, with secrets `rotated` on a periodic basis and credentials `deleted` when no longer needed*
<br><br>

## **2. Phases of Service Approval**
Runbooks are living documents and as such may be in varieng states of completion over time, this leads to the possibility of confusion around when a service can be considered ready for use in each environment. To help quell any confusion, the following **`Service Approval Phases`** will be attached to each runbook, and the associted mapping files.

- ### Service Denied
	- Service not yet reviewed
	- Service fails to meet requirements 

- ### Service Discovery
	- Initial review of service in progress
	- initial review of public access requirements

- ### Service Development Ready
	- Public Access requirements defined and documented
	- Public Access rules are defined & implemented

- ### Service Production Ready
	- Public Access rules defined and documented
	- Public Access rules are defined & implemented
	- Encryption requirements defined and documented
	- IAM requirements defined and documented
	- Encryption & IAM rules not yet defined

- ### Service Fully Approved
	- All Public Access, Encryption & IAM requirements defined and documented
    - All Public Access, Encryption & IAM rules defined & implemented
<br><br>

## **3. Runbook Review Procedure for Pull Requests (PR)**