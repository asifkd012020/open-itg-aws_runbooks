# Review Notes
This is simply a record of what Security Reviews have looked at historically; it will be superceded in the future, but provides an initial notes on the kind of things we look at.

This may help calibrate playbook & runbook for this service.

## Encryption
For EKS, EncryptionConfig should be set to use a CG KMS key to encrypt secrets. This allows EKS secrets to meet encryption controls (though we favor using AWS IAM secrets, it's not a requirement from a control perspective - it'll be encrypted and access is managed by kubctl)

### Using FarGate
Make /tmp read only, using the correct dockerfile directives. /tmp is currently unencrypted, and until our posture changes, we should prevent use of that store (do mem backed storage instead)

## Network
Communication in the ResourceVpcConfig should be setup to only allow node to node communication

### Cluster intercommunication
(Something like this)

```
  IngressDefaultClusterToNodeSG:
    Type: 'AWS::EC2::SecurityGroupIngress'
    Properties:
      Description: >-
        Allow managed and unmanaged nodes to communicate with each other (all
        ports)
      FromPort: 0
      GroupId: !Ref ClusterSharedNodeSecurityGroup
      IpProtocol: '-1'
      SourceSecurityGroupId: !GetAtt ControlPlane.ClusterSecurityGroupId
      ToPort: 65535
  IngressInterNodeGroupSG:
    Type: 'AWS::EC2::SecurityGroupIngress'
    Properties:
      Description: Allow nodes to communicate with each other (all ports)
      FromPort: 0
      GroupId: !Ref ClusterSharedNodeSecurityGroup
      IpProtocol: '-1'
      SourceSecurityGroupId: !Ref ClusterSharedNodeSecurityGroup
      ToPort: 65535
  IngressNodeToDefaultClusterSG:
    Type: 'AWS::EC2::SecurityGroupIngress'
    Properties:
      Description: Allow unmanaged nodes to communicate with control plane (all ports)
      FromPort: 0
      GroupId: !GetAtt ControlPlane.ClusterSecurityGroupId
      IpProtocol: '-1'
      SourceSecurityGroupId: !Ref ClusterSharedNodeSecurityGroup
      ToPort: 65535
```

### ALB
We should have an ALB playbook, but generally verify it is set up to require HTTPS

## Access Management
The ingress controller can have IAM integration, though keeping the role sufficiently narrow is important.

The baseline one with a narrow assume role policy, and only the `ALBIngressControllerIAMPolicy` tends to check those boxes. It's close to least privilege. 

Setting up the OIDC federation for this most likely requires SecEng - OKTA Services at this point; we don't let app teams create their own ferdation today. 

## Repo Source
a CG ECR or Nexus; use the ECR playbook to validate for appropriateness when we get there. 