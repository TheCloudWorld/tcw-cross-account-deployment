# Please follow the below steps to deploy cross-account.

A brief description of what this project does and who it's for

# Step - 1 (In Developer Account)::

Please create Same account CodePipeline using GitHub, CodeBuild and CodeDeploy.
refer: CodePipeline Lecture:  https://youtu.be/-OsKxWO4-Fk

# Step - 2 (In Developer Account):

Create the CMK key in Developer Account and Share this with Production Account.


## Steps to create CMK key:
```bash
  a. Go to the AWS KMS console.
  b. Go to Customer Managed Key.
  c. Leave the symmetric key as default.
  d. Enter the Alias for this key. Ex- myKey
  e. In Define Key Administrative Permissions, choose your IAM user and any other users or groups you want to act as administrators for this key, and then choose Next.
  f. In Define Key Usage Permissions, under This Account, select the name of the service role for the pipeline.
  g, Under Other AWS accounts, choose Add another AWS account. Enter the account ID for Production Account.
  h. Keep the ARN of the key handy.
```

# Now we need to setup the Account policies and Roles:

# Step - 3 (In Developer Account):

Create a policy for S3 bucket which grants access to Production Account.

```bash
  a. Open the bucket which you have created in step-1 to store the artifacts.
  b. Go to properties of the bucket.
  c. Expand permissions and choose "Add bucket policy"
  d. Add the below policy to the bucket and save.
```
## S3 bucket policy:

-> Copy and Paste the below policy, then Replace "codepipeline-us-east-1-667481606687" with your bucket-name & Replace "112027425997" with your Production Account ID.

```javascript
{
    "Version": "2012-10-17",
    "Id": "SSEAndSSLPolicy",
    "Statement": [
        {
            "Sid": "DenyUnEncryptedObjectUploads",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::codepipeline-us-east-1-667481606687/*",
            "Condition": {
                "StringNotEquals": {
                    "s3:x-amz-server-side-encryption": "aws:kms"
                }
            }
        },
        {
            "Sid": "DenyInsecureConnections",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::codepipeline-us-east-1-667481606687/*",
            "Condition": {
                "Bool": {
                    "aws:SecureTransport": "false"
                }
            }
        },
        {
            "Sid": "",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::112027425997:root"
            },
            "Action": [
                "s3:Get*",
                "s3:Put*"
            ],
            "Resource": "arn:aws:s3:::codepipeline-us-east-1-667481606687/*"
        },
        {
            "Sid": "",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::112027425997:root"
            },
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::codepipeline-us-east-1-667481606687"
        }
    ]
}
```



# Step - 4 (In Developer Account):

Create a policy for the Codepipeline Service Role.

```bash
  a. Go to IAM roles and open your Codepipeline Service role.
  b. On the Permissions tab, choose Add inline policy.
  c. Choose JSON tab and copy and paste the below policy. It allows production account to assume this role.
  d. Replace "112027425997" with your Production Account ID. 
```


```javascript
{
    "Version": "2012-10-17",
    "Statement": {
        "Effect": "Allow",
        "Action": "sts:AssumeRole",
        "Resource": [
            "arn:aws:iam::112027425997:role/*"
        ]
    }
}
```

![image](https://user-images.githubusercontent.com/74950445/129444217-9a03d5dc-df8c-4403-bd1f-59739ff245cb.png)



# Step - 5 (In Production Account):

Create a policy for EC2 Instance Role.

```bash
  a. Go to IAM roles and open your EC2 Service role.
  b. On the Permissions tab, choose Add inline policy.
  c. Choose the JSON tab, and enter the following policy to grant access to the Amazon S3 bucket used by Developer Account to store artifacts for pipelines.
  d. Replace "codepipeline-us-east-1-667481606687" with your S3 bucket.
```
```javascript
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:Get*"
            ],
            "Resource": [
                "arn:aws:s3:::codepipeline-us-east-1-667481606687/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::codepipeline-us-east-1-667481606687"
            ]
        }
    ]
}
```
![image](https://user-images.githubusercontent.com/74950445/129444631-191c87ce-a7df-4401-b6d9-b150585c0b19.png)


```bash
  e. Create the second policy for KMS.
  f. Copy and Paster the below policy.
  g. Replace "arn:aws:kms:us-east-1:667481606687:key/d3cb5a16-2cd0-43c5-94c9-6b98af73a1eb" with your KMS ARN from Developer Account.
```

```javascript
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "kms:DescribeKey",
                "kms:GenerateDataKey*",
                "kms:Encrypt",
                "kms:ReEncrypt*",
                "kms:Decrypt"
            ],
            "Resource": [
                "arn:aws:kms:us-east-1:667481606687:key/d3cb5a16-2cd0-43c5-94c9-6b98af73a1eb"
            ]
        }
    ]
}
```
![image](https://user-images.githubusercontent.com/74950445/129444745-7467dc90-9d61-4cfa-82d6-f3251ca85877.png)

# Step - 6 (In Production Account):

Configure the Cross-Account Role.

```bash
  a. Go to IAM console.
  b. Go to Roles and create new role.
  c. Under Select type of trusted entity, choose Another AWS account. Under Specify accounts that can use this role, in Account ID, enter the AWS account ID of Developer Account        and then choose Next: Permissions.
  d. UnderAttach permissions policies, choose AmazonS3ReadOnlyAccess, and then choose Next: Tags. (AmazonS3ReadOnlyAccess policy needs to be deleted in further steps)
  e. Create the role.
  f. Now, Open the same role you just created.
  g. On the Permissions tab, choose Add inline policy.
  h. Choose the JSON tab, and enter the following policy to allow access to CodeDeploy resources:
```
```javascript
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "codedeploy:CreateDeployment",
                "codedeploy:GetDeployment",
                "codedeploy:GetDeploymentConfig",
                "codedeploy:GetApplicationRevision",
                "codedeploy:RegisterApplicationRevision"
            ],
            "Resource": "*"
        }
    ]
}
```

![image](https://user-images.githubusercontent.com/74950445/129444924-6d79a72f-ae9d-4c0f-ae11-4ad0dc3cc301.png)

```bash
  i. Again, Choose Add Inline Policy.
  j. Choose the JSON tab, and enter the following policy to allow this role to retrieve input artifacts from, and put output artifacts into, the Amazon S3 bucket in Developer          Account.
  k. Replace "codepipeline-us-east-1-667481606687" with your S3 bucket name.
```
```javascript
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject*",
                "s3:PutObject",
                "s3:PutObjectAcl",
                "codecommit:ListBranches",
                "codecommit:ListRepositories"
            ],
            "Resource": [
                "arn:aws:s3:::codepipeline-us-east-1-667481606687/*"
            ]
        }
    ]
}
```
![image](https://user-images.githubusercontent.com/74950445/129445015-e31b0c86-4fdf-4aec-adb7-3e5153c617e8.png)


```bash
  l. On the Permissions tab, find AmazonS3ReadOnlyAccess in the list of policies under Policy Name, and choose the delete icon (X) next to the policy. When prompted, choose Detach
  m. Your permission tab will look like below.
```
![image](https://user-images.githubusercontent.com/74950445/129445027-c8d009d2-4f53-4f32-b1e5-057bd804eff4.png)

```bash
  n. Go to the Trust Relationship Tab, and make sure you see the Account ID of Developer Account.
```
![image](https://user-images.githubusercontent.com/74950445/129445054-206f0246-97f6-48d9-bd8a-7a4ff7a97e78.png)


# Step - 7 (In Developer Account):

Edit the pipeline created in Developer Account.

## Note

The below steps cannot be performed using Console. You need to use the AWS CLI to perform the below steps:

```bash
  a. At a terminal (Linux, macOS, or Unix) or command prompt (Windows), run the get-pipeline command on the pipeline to which you want to add resources. Copy the command output to      a JSON file. For example, for a pipeline named MyFirstPipeline, you would type something similar to the following:
  b. Replace "MyFirstPipeline" with your pipeline name. The Output will be stored in a file named "pipeline.json".
```
```javascript
aws codepipeline get-pipeline --name MyFirstPipeline >pipeline.json
```
```bash
  c. Open the JSON file in any plain-text editor, like notepad ++ or VS Code. After "type": "S3" under artifact store (Line no - 289), Replace "codepipeline-us-east-1-                  667481606687" with your Amazon S3 bucket name used to store artifacts for the pipeline & "arn:aws:kms:us-east-1:667481606687:key/d3cb5a16-2cd0-43c5-94c9-6b98af73a1eb" with        your KMS id.
```
```javascript

      "artifactStore": {
            "type": "S3",
            "location": "codepipeline-us-east-1-667481606687",
            "encryptionKey": {
                "id": "arn:aws:kms:us-east-1:667481606687:key/d3cb5a16-2cd0-43c5-94c9-6b98af73a1eb",
                "type": "KMS"
            }
        },
```
```bash
  c. Add a deploy action in a stage to use the CodeDeploy resources associated with Production Account, including the roleArn values for the cross-account role you created              (CrossAccount_Role).
  d. Replace "codedeploy-production" with your CodeDeploy Application name.
  e. Replace "codedeploy-group" with your COdeDeploy Group name.
  f. Replace "arn:aws:iam::112027425997:role/cross-account-s3-read-only" with your cross account role ARN.
```
```javascript

            {
                "name": "Staging",
                "actions": [
                    {
                        "inputArtifacts": [
                            {
                                "name": "SourceArtifact"
                            }
                        ],
                        "name": "ExternalDeploy",
                        "actionTypeId": {
                            "category": "Deploy",
                            "owner": "AWS",
                            "version": "1",
                            "provider": "CodeDeploy"
                        },
                        "outputArtifacts": [],
                        "configuration": {
                            "ApplicationName": "codedeploy-production",
                            "DeploymentGroupName": "codedeploy-group"
                        },
                        "runOrder": 1,
                        "roleArn": "arn:aws:iam::112027425997:role/cross-account-s3-read-only"
                    }
                ]
            }
```
```bash
  f. You must remove the metadata lines from the file so the update-pipeline command can use it. Remove the section from the pipeline structure in the JSON file (the "metadata": {      } lines and the "created", "pipelineARN", and "updated" fields). Example:
```
```javascript

"metadata": {  
  "pipelineArn": "arn:aws:codepipeline:region:account-ID:pipeline-name",
  "created": "date",
  "updated": "date"
  }
```
```bash
  g. Save the file.
  h. Now we need to update this "pipeline.json" file in codepipeline to make the entire changes.
  i. Run the below command to update the CodePipeline using CLI.
  j. Replace "pipeline.json" with your file name.
```
```javascript
aws codepipeline update-pipeline --cli-input-json file://pipeline.json
```

## Note

For undertanding purpose, i have added my pipeline.json file. Kindly go through it.

## May be you will get the below error, while uploading the JSON file using above command:

![image](https://user-images.githubusercontent.com/74950445/129445936-4f76c323-908c-475a-ba51-0a9beeef5cd6.png)

In this situation, please follow the below steps:

```bash
  i. Convert your JSON file in YAML file.
  ii. Now use the below command to update the CodePipeline. (Using google). I have also added Pipeline.YML file for your reference.
```
```javascript
aws codepipeline update-pipeline --cli-input-json file://pipeline.yml
```

## Once you finish with all above step, your pipeline should look like below:


![image](https://user-images.githubusercontent.com/74950445/129446111-05a247d1-3c8f-44df-ab58-8f4ca4dbe52a.png)

Where, the first two stages is in Developer Account and the 3rd stage is in Production Account.








## Reference:

[Documentation](https://docs.aws.amazon.com/codepipeline/latest/userguide/pipelines-create-cross-account.html#pipelines-create-cross-account-setup-accounta)
