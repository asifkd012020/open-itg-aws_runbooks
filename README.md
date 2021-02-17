# aws_runbooks
AWS runbooks is aset of consumable and repeatable templates for teams to use when deploying various AWS resources aligned to the NIST Cybersecurity Framework (CSF) that will ensure services are secured from deployment and during life of the service.

# Playbooks
Playbooks are risk requirement documents that outline NIST (CSF) controls that are applicable to a particular AWS service and how an AWS service can meet those controls from the mapping provided in the document. Included in each playbook:

 - What the AWS service is capable of achieving
 - What actions should be done to align the service to the controls in the NIST CSF. 
 - highlight gaps for controls for the service it is written for. Details for address gaps are in the runbook. 
 
Control statements and 'why' statements for the controls are to be technology agnostic. The 'how' statements for control objectives is where examples of AWS technology that can meet the control objective can be mentioned. 

Additionally a playbook is not a technical document, implementation guide nor does it map to Capital Group internal controls.   

# Runbooks
Runbooks are design documents for implementation of an AWS service for how to meet the NIST CSF requirements that are outlined in the associated playbook. Runbooks should address gaps in controls that have been highlighted in the playbook by providing steps for addressing the gap, advise against feature sets of the service if the gap cannot be addressed as well as advising the Capital Group governing body for exception requests. 

## Contributing
While AWS, PDS, and Risk will continue to build out playbooks, in the spirit of OpenITG, individual contributions to updates or creation of playbooks is available to everyone. 

### Requesting a new Service Runbook
To request a review for the approval to use a new AWS service and start the process of the Runbook creation, one can open a request in [service now](https://now/) and assign to the following service group: `'Security Runbooks'`

The [Service Request documentation](/SERVICE_REQUEST_PROCESS.md) outlines the current process and timelines for a review.

### Creating a new Service Playbook
To create a playbook, create a branch for the given service from `master`. Once done, create a new folder for the service, using the most common name for the service. 

Pull the [template](/template/AWS_PS_RUNBOOK.md) into your new folder, and follow efforts to align to it.

When drafting a playbook, focus on the [capital group control](/template/AWS_PS_RUNBOOK.md#capital-group-control-statements). #1, #2, #4, #5, and #6 tend to be the controls that will have per service configuration, while the remaining control statements are generally managed by the overall cloud ecosystem. While many of the examples, such as [RDS](/rds/RUNBOOK.md) may include NIST control statement too, that is not required for a CG runbook.

After completion, submit a Pull Request to master. The major requirements for a PR will be listed when you do, but you can also [view them](/PULL_REQUEST_TEMPLATE.md) ahead of time. Approvers will automatically be calculated. 

### Updating an existing playbook
Create a branch for the changes you're proposing, and submit the changes. Think heavily about prevent & detect controls for major objectives such as encryption, authentication, no public access, etcetera. These are critical components for any service at CG being approved. 

Approvers will be automatically calculated, per the [CODEOWNERS](/CODEOWNERS) file

