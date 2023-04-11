---
title : "Cleanup"
date :  "`r Sys.Date()`" 
weight : 4
chapter : false
pre : " <b> 4. </b> "
---
1. Empty S3 bucket
- Open [AWS S3 console](https://s3.console.aws.amazon.com/s3/buckets?region=ap-southeast-1)
- Select **fcjdmswebstore**
- Click **Empty**
- Enter **permanently delete**
- Click **Empty**
- Do the same for buckets: **aws-sam-cli-managed-default-** and **fcjdmsstore**
2. Run the following commands at the root of the front-end project:
```
amplify remove storage
amplify remove auth
amplify push
amplify delete
```
3. Delete CloudFormation stacks
- Execute the below command to delete the AWS SAM application
```
sam delete --stack-name fcjdmsapp
sam delete --stack-name aws-sam-cli-managed-default
```