# AWS Elastic Compute Cloud (EC2)
- [AWS Simple Storage Service (S3)](#aws-simple-storage-service-s3)
    - [Move a bucket to another account](#move-a-bucket-to-another-account)
        - [Source Bucket Policy](#source-bucket-policy)
        - [IAM Policy for user in the target account](#iam-policy-for-user-in-the-target-account)
        - [Copy Content from Source to Destination](#copy-content-from-source-to-destination)
    - [Allow access to bucket from accounts in the same organization](#allow-access-to-bucket-from-accounts-in-the-same-organization)

## Tagging EBS Volumes in an Auto Scaling Group
Per Default all tags defined in an Auto Scaling Group are only added to the EC2 Instances, not to the root volumes of those.
To workaround this issue you need to add the following role and attach it to the instance profile which is used by your Auto Scaling Launch Configuration
```yaml
# role with permission to create tags and describe all volumes
InstanceRole:
    Type: AWS::IAM::Role
    Properties:
      Policies:
      - PolicyDocument:
          Version: '2012-10-17'
          Statement:
          - Action: 'ec2:CreateTags'
            Resource:
              Fn::Sub: arn:aws:ec2:${AWS::Region}:${AWS::AccountId}:volume/*
            Effect: Allow
          - Action: 'ec2:DescribeVolumes'
            Resource : '*'
            Effect: Allow
        PolicyName: auto-tag-volumes-policy
      AssumeRolePolicyDocument:
        Statement:
        - Action:
          - sts:AssumeRole
          Principal:
            Service:
            - ec2.amazonaws.com
          Effect: Allow
        Version: '2012-10-17'
# instance profile	with the role attached
BastionHostProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      Roles:
      - Ref: BastionHostRole
      Path: "/"
```

The following user data need to be added to the launch configuration:

```yaml
LaunchConfiguration:
	Type: AWS::AutoScaling::LaunchConfiguration
    Properties:
		[...]
		UserData:
			Fn::Base64:
			  Fn::Join:
			  - ""
			  - - "#!/bin/bash"
				- "\n"
				- 'set -x'
				- "\n"
				- INSTANCE_ID=`curl http://169.254.169.254/latest/meta-data/instance-id`
				- "\n"
				- APPNAME=test-dev-1
				- "\n"
				- REGION=
				- Fn::ImportValue:
					Fn::Sub: "${AWS::Region}"
				- "\n"
				- ROOT_DISK_ID=$( aws ec2 describe-volumes  --filters 'Name=attachment.device,Values=/dev/xvda' Name=attachment.instance-id,Values=$INSTANCE_ID --query 'Volumes[*].{ID:VolumeId}' --region $REGION --output text )
				- "\n"
				- aws ec2 create-tags --resource $ROOT_DISK_ID --tags Key=Name,Value=$APPNAME-xvda --region $REGION

```