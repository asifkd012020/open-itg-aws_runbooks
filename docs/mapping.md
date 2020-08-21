# GRC Mapping Files
Information related to the creation, modification and use of the GRC mapping files.

## What is GRC?
Our new GRC (Global Risk and Compliance) module in ServiceNow has allowed for a better place for storing the Cloud security controls that all deployments need to adhere to. Previously all the control statements were stored in Confluence and were both hard to find and not specifically talored to include Cloud based deployments.  The new GRC platform allows for easier lookup of controls and where they should be applied.

## What are GRC Mapping Files?
We now know that our GRC tool allows for a better place to store and lookup our controls, but how do we ensure we adhere to them correctly across all the Cloud resources we have out there?  This is where the GRC mapping files come into play. Each service, in each cloud provider should have a runbook associated with it. The runbooks give exaples of how CG expects a specific service to be built, including which detective and preventative controls should be present. 

The GRC mapping files, as with the runbooks should be present for each service that a cloud provider offers and allows a platform that teams can utilize to show how they are meeting the control objectives set out in the runbook for the specific service as seen in the diagram below.

## Structure of a Mapping File
Now that we have a good understanding of the what and why around GRC Mapping files, the next step is to delve into the structure of the files as below.

### Service requirement name
This section of code will list the service that this mapping file was created for, in the example below AWS's ECS (Elastic Container Service).

```json
    "servicerequirement_name": [
          "AWS::ECS::*"
    ],
```

### Service display name
This field provides a friendly name for the service, in the example ECS (Elastic Container Service).

```json
   "servicedisplay_name": "ECS (Elastic Container Service)",
```

### Phase
This field provides the current CG lifecycle phase for the service e.g. Discovery, Prefered etc.

```json
   "phase": "discovery",
```

### Mapping
This field provides the actual mapping of each runbok requirement and all the associated GRC Controls and control implementations.

```json
    "mapping": [
        {
```

### Requirement Mapping
Under the mapping there can be multiple requirements mapping for the service selected, each requirement mapping will contain the following fields:
1. requirement_name: This field will be the actual requirement name.
2. policy: This field contains a link to the runbook policy for the requirement.
3. tags[]: This field is a placeholder that will be used in future.
4. severity: This field and subfields allow us to set the severity of a violation in each environment.
5. control_mapping[]: This field will contain all the GRC control ID's associated with the specific requirement.
6. implementations[]: This field and subfields contain the implementation provider, links tothe implementation code, and the type of control e.g. detective or preventative.

```json
            "requirement_requirement_name": "ECS - Disable Public IP Access",
            "policy": "https://confluence.capgroup.com/display/HCEA/AWS+Services+Security+Checklist+for+CG+Deployments#AWSServicesSecurityChecklistforCGDeployments-ECS_DisablePubAPIAccess",
            "tags":[],
            "severity": [
                {
                    "Prod": "critical",
                    "SNP": "critical",
                    "NP": "medium"
                }
            ],
            "control_mapping": [],
            "implementations": [
                {
                    "provider": "dome9",
                    "implementation": [
                        "https://bitbucket.capgroup.com/projects/CLSEC/repos/dome9/browse/AWS_Rulesets/CG-AWS-SO-NetworkControl.json#63",
                        "https://bitbucket.capgroup.com/projects/CLSEC/repos/dome9/browse/AWS_Rulesets/CG-AWS-SO-NetworkControl.json#73"
                    ],
                    "type": "detect"
                },
                {
                    "provider": "aws config",
                    "implementation": [
                        "https://confluence.capgroup.com/display/HCEA/Operational+Best+Practices+-+Detect+Publicly+Accessible+Resources"
                    ],
                    "type": "detect"
                },
                {
                    "provider": "panoptes",
                    "implementation": [
                        "https://github.com/open-itg/panoptes/blob/master/panoptes-rules/ecs.py#L5"
                    ],
                    "type": "prevent"
                }
            ]
        },
```

### Full Example Mapping File
Below is a full example of what a mapping file might look like.

```json
{
    "servicerequirement_name": [
        "AWS::ECS::*"
    ],
    "servicedisplay_name": "ECS (Elastic Container Service)",
    "phase": "discovery",
    "mapping": [
        {
            "requirement_requirement_name": "ECS - Disable Public IP Access",
            "policy": "https://confluence.capgroup.com/display/HCEA/AWS+Services+Security+Checklist+for+CG+Deployments#AWSServicesSecurityChecklistforCGDeployments-ECS_DisablePubAPIAccess",
            "tags":[],
            "severity": [
                {
                    "Prod": "critical",
                    "SNP": "critical",
                    "NP": "medium"
                }
            ],
            "control_mapping": [],
            "implementations": [
                {
                    "provider": "dome9",
                    "implementation": [
                        "https://bitbucket.capgroup.com/projects/CLSEC/repos/dome9/browse/AWS_Rulesets/CG-AWS-SO-NetworkControl.json#63",
                        "https://bitbucket.capgroup.com/projects/CLSEC/repos/dome9/browse/AWS_Rulesets/CG-AWS-SO-NetworkControl.json#73"
                    ],
                    "type": "detect"
                },
                {
                    "provider": "aws config",
                    "implementation": [
                        "https://confluence.capgroup.com/display/HCEA/Operational+Best+Practices+-+Detect+Publicly+Accessible+Resources"
                    ],
                    "type": "detect"
                },
                {
                    "provider": "panoptes",
                    "implementation": [
                        "https://github.com/open-itg/panoptes/blob/master/panoptes-rules/ecs.py#L5"
                    ],
                    "type": "prevent"
                }
            ]
        },
        {
            "requirement_name": "ECS - Resource Tagging",
            "policy": "https://confluence.capgroup.com/display/HCEA/AWS+Services+Security+Checklist+for+CG+Deployments#AWSServicesSecurityChecklistforCGDeployments-ECS_Tagging",
            "tags":[],
            "severity": [
                {
                    "Prod": "high",
                    "SNP": "high",
                    "NP": "medium"
                }
            ],
            "control_mapping": [],
            "implementations": []
        }
    ]
}
```

