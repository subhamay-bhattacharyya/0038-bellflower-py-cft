![](https://img.shields.io/github/commit-activity/t/subhamay-bhattacharyya/9998-cfn-template)&nbsp;![](https://img.shields.io/github/last-commit/subhamay-bhattacharyya/9998-cfn-template)&nbsp;![](https://img.shields.io/github/release-date/subhamay-bhattacharyya/9998-cfn-template)&nbsp;![](https://img.shields.io/github/repo-size/subhamay-bhattacharyya/9998-cfn-template)&nbsp;![](https://img.shields.io/github/directory-file-count/subhamay-bhattacharyya/9998-cfn-template
)&nbsp;![](https://img.shields.io/github/actions/workflow/status/subhamay-bhattacharyya/9998-cfn-template/deploy-stack.yaml)&nbsp;![](https://img.shields.io/bitbucket/issues/subhamay-bhattacharyya/9998-cfn-template)&nbsp;![](https://img.shields.io/github/languages/top/subhamay-bhattacharyya/9998-cfn-template)&nbsp;![](https://img.shields.io/github/commit-activity/m/subhamay-bhattacharyya/9998-cfn-template)&nbsp;![](https://img.shields.io/badge/project_status-red-red)



# Project Bellflower: Step Function with Lambda, SNS, SQS, S3 orchestration with callback functionality

A State Machine demonstrating Lambda orchestration with a third party application using S3 and SQS and Lambda Callback functionality and notifying the users using SNS notification.

## Description

This is demonstration of a State Machine with Lambda, SNS, SQS and S3. The processing Lambda generates a payload and pushes to a SQS Queue. An integration Lambda processes the payload and send a file with task token to S3 outbound folder. An Business Application processes the information and sends a success trigger to the S3 inbound folder. The integration Lambda reads the success trigger and sends a success signal to the State Machine and resumes processing. Once processing completes, the State Machine sends a Success notification to SNS Topic. If the processing Lambda fails after 3 retries the State Machine fails and a Failure notification is sent to the SNS Topic. The entire stack is created using CloudFormation.

```mermaid
flowchart TD
    A((Start)) --> B[Generate Data]
    B[Generate Data] --> C[Map State]
    C[Map State] --> D[Process Data]
    D[Process Data] --> C[Map State]
    C[Map State] -- After Processing all the records --> E[Purge Queue]
    E[Purge Queue] --> F{All records processed ?}
    F -- Yes --> G[Send Failed event to SQS]
    F -- No --> H[Success]
    G[Send Failed event to SQS] --> I[Fail]
    H --> J((End))
    I --> J((End))
```

![Project Carnation - Design Diagram](https://github.com/subhamay-bhattacharyya/0063-carnation-py-cft/blob/main/architecture-diagram/carnation-step-function.png)


### Services Used
```mermaid
mindmap
  root((1 -  AWS CloudFormation ))
    2
        AWS Step Function
    3
        AWS Lambda 
    4
        AWS IAM 
    5   
        AWS CloudWatch
    6   
        AWS Key Management Service
    7   
        AWS Simple Notification Service
    8   
        AWS Simple Queue Service
```

### Getting Started

* This repository is configured to deploy the stack in Development, Staging and Production AWS Accounts. To use the pipeline you need to have 
three AWS Accounts created in an AWS Org under a Management Account (which is the best practice). The Org structure will be as follows:

```
Root
├─ Management
├─ Development
├─ Test
└─ Production
```

* Create KMS Key in each of the AWS Accounts which will be used to encrypt the resources.

* Create an OpenID Connect Identity Provider

* Create an IAM Role for OIDC and use the sample Trust Policy in each of the three AWS accounts
```
{
    "Version": "2008-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::<Account Id>:oidc-provider/token.actions.githubusercontent.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringLike": {
                    "token.actions.githubusercontent.com:sub": [
                        "repo:<GitHub User>/<GitHub Repository>:*",
                    ]
                }
            }
        }
    ]
}
```

  * Create an IAM Policy to allow CloudFormation access and attach it to the OIDC Role, using the sample policy document:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "cloudformation:CreateStack",
                "cloudformation:DescribeStacks",
                "cloudformation:CreateChangeSet",
                "cloudformation:DescribeChangeSet",
                "cloudformation:DeleteChangeSet",
                "cloudformation:ExecuteChangeSet",
                "cloudformation:DeleteStack"
            ],
            "Resource": "*"
        }
    ]
}
```

  * Create an IAM Policy to allow creation and deletion of resources  and attach it to the OIDC Role, using the following sample policy document:
```
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Sid": "AllowS3Access",
			"Effect": "Allow",
			"Action": [
				"s3:PutObject",
				"s3:GetObject",
				"s3:CreateBucket",
				"s3:PutObjectTagging",
				"s3:PutBucketPublicAccessBlock",
				"s3:PutBucketAcl",
				"s3:DeleteObject",
				"s3:DeleteBucket",
				"s3:ListBucket",
				"s3:PutBucketTagging",
				"s3:GetBucketTagging",
				"s3:GetBucketPolicy",
				"s3:GetBucketAcl",
				"s3:GetObjectAcl",
				"s3:GetObjectVersionAcl",
				"s3:PutObjectAcl",
				"s3:PutObjectVersionAcl",
				"s3:GetBucketCORS",
				"s3:PutBucketCORS",
				"s3:GetBucketWebsite",
				"s3:GetBucketVersioning",
				"s3:GetAccelerateConfiguration",
				"s3:GetBucketRequestPayment",
				"s3:GetBucketLogging",
				"s3:GetLifecycleConfiguration",
				"s3:GetReplicationConfiguration",
				"s3:GetEncryptionConfiguration",
				"s3:GetBucketObjectLockConfiguration",
				"s3:GetBucketOwnershipControls",
				"s3:GetObjectTagging",
				"s3:GetBucketNotification",
				"s3:PutEncryptionConfiguration",
				"s3:PutBucketOwnershipControls",
				"s3:PutBucketNotification",
				"s3:ListBucketVersions",
				"s3:PutObjectTagging"
			],
			"Resource": "*"
		},
		{
			"Sid": "IAMAccess",
			"Effect": "Allow",
			"Action": [
				"iam:GetRole",
				"iam:CreateRole",
				"iam:DeleteRolePolicy",
				"iam:PutRolePolicy",
				"iam:DeleteRole",
				"iam:PassRole",
				"iam:TagRole",
				"iam:CreatePolicy",
				"iam:ListRolePolicies",
				"iam:GetPolicy",
				"iam:ListAttachedRolePolicies",
				"iam:GetPolicyVersion",
				"iam:ListInstanceProfilesForRole",
				"iam:AttachRolePolicy",
				"iam:DetachRolePolicy",
				"iam:ListPolicyVersions",
				"iam:DeletePolicy",
				"iam:UntagRole",
				"iam:getRolePolicy",
				"iam:CreateInstanceProfile",
				"iam:RemoveRoleFromInstanceProfile",
				"iam:AddRoleToInstanceProfile",
				"iam:DeleteInstanceProfile"
			],
			"Resource": "*"
		},
		{
			"Sid": "LambdaCreateAccess",
			"Effect": "Allow",
			"Action": [
				"lambda:GetFunction",
				"lambda:DeleteFunction",
				"lambda:CreateFunction",
				"lambda:TagResource",
				"lambda:InvokeFunction",
				"lambda:PutFunctionConcurrency",
				"lambda:CreateFunction",
				"lambda:ListVersionsByFunction",
				"lambda:GetFunctionCodeSigningConfig",
				"lambda:PutFunctionEventInvokeConfig",
				"lambda:AddPermission",
				"lambda:GetFunctionEventInvokeConfig",
				"lambda:GetPolicy",
				"lambda:DeleteFunctionEventInvokeConfig",
				"lambda:RemovePermission",
				"lambda:ListTags"
			],
			"Resource": "*"
		},
		{
			"Sid": "SQSCreateAccess",
			"Effect": "Allow",
			"Action": [
				"sqs:GetQueueAttributes",
				"sqs:ListQueueTags",
				"sqs:CreateQueue",
				"sqs:TagQueue",
				"sqs:SetQueueAttributes",
				"sqs:SendMessage",
				"sqs:DeleteQueue"
			],
			"Resource": "*"
		},
		{
			"Sid": "SNSCreateAccess",
			"Effect": "Allow",
			"Action": [
				"SNS:GetTopicAttributes",
				"SNS:ListTagsForResource",
				"SNS:GetSubscriptionAttributes",
				"SNS:CreateTopic",
				"SNS:TagResource",
				"SNS:SetTopicAttributes",
				"SNS:DeleteTopic",
				"SNS:Subscribe",
				"SNS:Unsubscribe"
			],
			"Resource": "*"
		},
		{
			"Sid": "DynamoDBAccess",
			"Effect": "Allow",
			"Action": [
				"dynamodb:DescribeTable",
				"dynamodb:DescribeContinuousBackups",
				"dynamodb:DescribeTimeToLive",
				"dynamodb:ListTagsOfResource",
				"dynamodb:CreateTable",
				"dynamodb:TagResource",
				"dynamodb:DeleteTable"
			],
			"Resource": "*"
		},
		{
			"Sid": "KMSKeyAccess",
			"Effect": "Allow",
			"Action": [
				"kms:Encrypt",
				"kms:Decrypt",
				"kms:DescribeKey"
			],
			"Resource": [
				"arn:aws:kms:<AWS Region>:<Account Id>:key/<KMS Key Id>"
			]
		},
		{
			"Sid": "CloudWatchAccess",
			"Effect": "Allow",
			"Action": [
				"cloudwatch:PutMetricAlarm",
				"cloudwatch:DescribeAlarms",
				"cloudwatch:ListTagsForResource",
				"cloudwatch:DeleteAlarms",
				"cloudwatch:TagResource"
			],
			"Resource": "*"
		},
		{
			"Sid": "StateMachineAccess",
			"Effect": "Allow",
			"Action": [
				"states:CreateStateMachine",
				"states:TagResource",
				"states:DescribeStateMachine",
				"states:DeleteStateMachine"
			],
			"Resource": "*"
		},
		{
			"Sid": "CWLogGroupAccess",
			"Effect": "Allow",
			"Action": [
				"logs:CreateLogGroup",
				"logs:TagResource",
				"logs:PutRetentionPolicy",
				"logs:DescribeLogGroups",
				"logs:DeleteLogGroup"
			],
			"Resource": "*"
		},
		{
			"Sid": "SSMAccess",
			"Effect": "Allow",
			"Action": [
				"ssm:GetParameters"
			],
			"Resource": "*"
		},
		{
			"Sid": "EC2Access",
			"Effect": "Allow",
			"Action": [
				"ec2:DescribeVpcs",
				"ec2:DescribeSubnets",
				"ec2:DescribeImages",
				"ec2:DescribeInstances",
				"ec2:DescribeSecurityGroups",
				"ec2:CreateSecurityGroup",
				"ec2:DeleteSecurityGroup",
				"ec2:AuthorizeSecurityGroupIngress",
				"ec2:CreateTags",
				"ec2:RunInstances",
				"ec2:TerminateInstances",
				"ec2:DescribeInstances"
			],
			"Resource": "*"
		}
	]
}
```

### Installing

* Clone the repository.
* Create a S3 bucket to used a code repository.
* Modify the params/cfn-parameter.json with you Key Id
```
{
    "stack-prefix": "<Stack Prefix>",
    "stack-suffix": "<Stack Suffix>",
    "template-path": "/cft/s3-custom-resource-lambda.yaml",
    "parameters": {
        "devl": [
            {
                "Environment": "devl",
                "KmsMasterKeyId": "<KMS Key Id in Dev Account>",
                "ProjectName": "<Your project name>"
            }
        ],
        "test": [
            {
                "Environment": "test",
                "KmsMasterKeyId": "<KMS Key Id in Test Account>",
                "ProjectName": "<Your project name>"
            }
        ],
        "prod": [
            {
                "Environment": "prod",
                "KmsMasterKeyId": "<KMS Key Id in Prod Account>",
                "ProjectName": "<Your project name>"
            }
        ]
    }
}
```

* Create three repository environments in GitHub (devl, test, prod)

* Create the following GitHub repository Secrets:


|Secret Name|Secret Value|
|-|-|
|AWS_REGION|```us-east-1```|
|DEVL_AWS_KMS_KEY_ARN|```arn:aws:kms:<AWS Region>:<Development Account Id>:key/<KMS Key Id in Development>```|
|TEST_AWS_KMS_KEY_ARN|```arn:aws:kms:<AWS Region>:<Test Account Id>:key/<KMS Key Id in Test>```|
|PROD_AWS_KMS_KEY_ARN|```arn:aws:kms:<AWS Region>:<Production Account Id>:key/<KMS Key Id in Production>```|
|DEVL_AWS_ROLE_ARN|```arn:aws:iam::<Development Account Id>:role/<OIDC IAM Role Name>```|
|TEST_AWS_ROLE_ARN|```arn:aws:iam::<Test Account Id>:role/<OIDC IAM Role Name>```|
|PROD_AWS_ROLE_ARN|```arn:aws:iam::<Production Account Id>:role/<OIDC IAM Role Name>```|
|DEVL_CODE_REPOSITORY_S3_BUCKET|```<Repository S3 Bucket in Development>```|
|TEST_CODE_REPOSITORY_S3_BUCKET|```<Repository S3 Bucket in Test>```|
|PROD_CODE_REPOSITORY_S3_BUCKET|```<Repository S3 Bucket in Production>```|

### Executing the CI/CD Pipeline

* Create Create a feature branch and push the code.
* The CI/CD pipeline will create a build and then will deploy the stack to devlopment.
* Once the Stage and Prod deployment are approved (If you have configured with protection rule ) the stack will be reployed in the respective environments

### Executing program

* To execute the state machine use the following input:
```
{
  "start_seq": 0,
  "end_seq":   <integer>
}
```

The above input will create a payload consisting of (end_seq - start_seq) elements.

In the subsequent run use the following input to just reprocess the events from the SQS queue:
```
{
  "start_seq": 0,
  "end_seq":  0
}
```

## Help

:email: Subhamay Bhattacharyya  - [subhamay.aws@gmail.com]


## Authors

Contributors names and contact info

Subhamay Bhattacharyya  - [subhamay.aws@gmail.com]

## Version History

* 0.1
    * Initial Release

## License

This project is licensed under Subhamay Bhattacharyya. All Rights Reserved.

## Acknowledgments

* AWS [Dynamically process data with a Map state] (https://docs.aws.amazon.com/step-functions/latest/dg/sample-map-state.html)
