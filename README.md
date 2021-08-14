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

-> Replace "codepipeline-us-east-1-667481606687" with your bucket-name
-> Replace "112027425997" with your Production Account ID.

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



# Step - 3 (In Developer Account):

Create a policy for S3 bucket which grants access to Production Account.


# Step - 3 (In Developer Account):

Create a policy for S3 bucket which grants access to Production Account.



# Step - 3 (In Developer Account):

Create a policy for S3 bucket which grants access to Production Account.


# Step - 3 (In Developer Account):

Create a policy for S3 bucket which grants access to Production Account.


# Step - 3 (In Developer Account):

Create a policy for S3 bucket which grants access to Production Account.














## Demo

Insert gif or link to demo

  
## Deployment

To deploy this project run

```bash
  npm run deploy
```

  
## Lessons Learned

What did you learn while building this project? What challenges did you face and how did you overcome them?

  
## Screenshots

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

  
## Usage/Examples

```javascript
import Component from 'my-project'

function App() {
  return <Component />
}
```

  
tcw-cross-account-deployment
