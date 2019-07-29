# CloudFormation
## Security Groups
### Circular Dependencies
With Security Groups you can open ports for IP-Adresses, Subnets or other security groups. If your architecture has two security groups (think of APP and WEB-Tier), which are referencing each others, the CloudFormation Stack will fail.

*This won't work, because none of the Security Groups can be created without the other one:*
```
AppTier:
  Type: AWS::EC2::SecurityGroup
  Properties: 
    GroupDescription: My App Tier
    GroupName: app-sg
    SecurityGroupIngress: 
      -   Description: Allow HTTP from Web Tier
          FromPort: 80
          ToPort: 80
          IpProtocol: tcp
          SourceSecurityGroupId: !Ref WebTier
          
WebTier:
  Type: AWS::EC2::SecurityGroup
  Properties: 
    GroupDescription: My Web Tier
    GroupName: web-sg
    SecurityGroupIngress: 
      -   Description: Allow SSH from App Tier
          FromPort: 22
          ToPort: 22
          IpProtocol: tcp
          SourceSecurityGroupId: !Ref AppTier

```

To fix this issue, CloudFormation offers an externalization of the SecurityGroupIngress as own resource. The fix would be the following:

```
AppTier:
  Type: AWS::EC2::SecurityGroup
  Properties: 
    GroupDescription: My App Tier
    GroupName: app-sg
    
AppTierWebIngress:
  Type: AWS::EC2::SecurityGroupIngress
  Properties: 
    Description: Allow HTTP from Web Tier
    FromPort: 80
    ToPort: 80
    IpProtocol: tcp
    SourceSecurityGroupId: !Ref WebTier
    GroupId: !Ref AppTier

WebTier:
  Type: AWS::EC2::SecurityGroup
  Properties: 
    GroupDescription: My Web Tier
    GroupName: web-sg

WebTierAppIngress:
  Type: AWS::EC2::SecurityGroupIngress
  Properties:
    Description: Allow SSH from App Tier
    FromPort: 22
    ToPort: 22
    IpProtocol: tcp
    SourceSecurityGroupId: !Ref AppTier
    GroupId: !Ref WebTier
```

In this example a single SecurityGroupIngress Resource either WebTierAppIngress or AppTierWebIngress would be sufficient. However I got into the way of creating an Ingress-Resource for each reference, to avoid circular dependencies.
The creation order would be something like the following: AppTier -> WebTier -> AppTierWebIngress -> WebTierAppIngress
