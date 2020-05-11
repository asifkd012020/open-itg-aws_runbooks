# aws_runbooks
AWS runbooks deliver a set of consumable and repeatable templates for teams to use when deploying various AWS resources that will ensure services are secured from deployment and during life of the service.

The runbooks will be built off of playbooks that align with NIST Cyber Security Frame work and establish guardrails for teams to develop the runbooks. 

## Contributing
While AWS, PDS, and Risk will continue to build out playbooks, in the spirit of OpenITG, individual contributions to updates or creation of playbooks is available to everyone. 

### Creating a new playbook
To create a playbook, create a branch for the given service from `master`. Once done, create a new folder for the service, using the most common name for the service. 

Pull the [template](/template/PLAYBOOK.md) into your new folder, and follow efforts to align to it.

(Supplementary reading todo: NIST framework)

When drafting a playbook, 

After completion, submit a Pull Request to master. The major requirements for a PR will be listed when you do, but you can also [view them](/PULL_REQUEST_TEMPLATE.md) ahead of time. Approvers will automatically be calculated. 

### Updating an existing playbook
Create a branch for the changes you're proposing, and submit the changes. Think heavily about prevent & detect controls for major objectives such as encryption, authentication, no public access, etcetera. These are critical components for any service at CG being approved. 

Approvers will be automatically calculated, per the [CODEOWNERS](/CODEOWNERS) file

