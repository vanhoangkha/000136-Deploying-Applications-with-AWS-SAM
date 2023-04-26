---
title : "Cleanup"
date :  "`r Sys.Date()`" 
weight : 4
chapter : false
pre : " <b> 4. </b> "
---
1. Empty S3 bucket
- Run the command: `aws s3 rm s3://BUCKET_NAME --recursive`
- Replace `BUCKET_NAME` with bucket name to empty: `fcjdmswebstore`, `fcjdmsstore`
2. Run the following commands at the root of the front-end project:
```
amplify remove storage
amplify remove auth
amplify push
amplify delete
```
3. Delete CloudFormation stacks
- Execute the command to delete the AWS SAM application: `sam delete --stack-name fcjdmsapp`
- Execute the command to empty S3 bucket: `aws s3 rm s3://BUCKET_NAME --recursive`
- Replace `BUCKET_NAME` with bucket names starting with: **aws-sam-cli-managed-default-samclisourcebucket-**
- Open [Amazon S3 console](https://ap-southeast-1.console.aws.amazon.com/s3/buckets?region=ap-southeast-1)
- Selecting the bucket name start with **aws-sam-cli-managed-default-samclisourcebucket-**
- Click **Show versions**
- Select the **fcjdmsapp** folder
- Select all objects
- Click **Delete**
- Enter `permanently delete`
- Click **Delete**
- Run the command to stack:
`sam delete --stack-name aws-sam-cli-managed-default`