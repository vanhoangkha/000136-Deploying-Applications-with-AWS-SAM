---
title : "Deploy Front-end"
date :  "`r Sys.Date()`" 
weight : 2
chapter : false
pre : " <b> 1.2 </b> "
---
1. Add the following code block at **Prarameters** section of **template.yaml** file in sam project:
```yaml
  WebStoreBucketName:
    Type: String
    Default: fcjdmswebstore
```
{{% notice note %}}
Change the **Default** value to change bucket name
{{% /notice %}}

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-18.png?featherlight=false&width=90pc)


2. Add the following code below the S3 bucket description in section 1.1. This code block describes the bucket that allows public access and enables website hosting:
```yaml
  FcjDMSWebStore:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Ref WebStoreBucketName
      PublicAccessBlockConfiguration:
        BlockPublicAcls: "false"
        BlockPublicPolicy: "false"
      WebsiteConfiguration:
        IndexDocument: 'index.html'

  FcjDMSWebStorePolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      Bucket: !Ref FcjDMSWebStore
      PolicyDocument:
        Version: 2012-10-17
        Statement:
          - Action:
              - "s3:GetObject"
            Effect: Allow
            Principal: "*"
            Resource: !Join
              - ""
              - - "arn:aws:s3:::"
                - !Ref FcjDMSWebStore
                - /*
```

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-19.png?featherlight=false&width=90pc)

3. Run the following commands to build and deploy sam project again:
```
sam build
sam deploy
```

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-20.png?featherlight=false&width=90pc)

4. Back to the root of the Front-end project, run the following command:
```
yarn build
aws s3 cp build s3://BUCKET_NAME --recursive
```
Replace `BUCKET_NAME` with the name of the bucket you just created to host the static website, such as: `aws s3 cp build s3://fcjdmswebstore --recursive`

5. Open [Amazon S3 console](https://s3.console.aws.amazon.com/s3/buckets?region=ap-southeast-1)
- Select just created bucket

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-21.png?featherlight=false&width=90pc)

6. Select **Properties** tab and scroll down to bottom. Click website's URL

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-22.png?featherlight=false&width=90pc)

7. Click **Sign up**

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-23.png?featherlight=false&width=90pc)

8. Enter user name, email và password. Then click **Sign up**

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-24.png?featherlight=false&width=90pc)

9. Open your email to get verify code and enter into the web. Then click **Submit**

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-25.png?featherlight=false&width=90pc)

10. Back to Cognito User Pool console, click **User** tab so you will see the account you just registered.

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-26.png?featherlight=false&width=90pc)

11. Sign in to website with the account you just registered.

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-27.png?featherlight=false&width=90pc)

12. Logged in successfully

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-28.png?featherlight=false&width=90pc)

13. Select **My Profile** on the left menu, then click **Update profile**.

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-29.png?featherlight=false&width=90pc)

14. Enter old and new password, then click **Upadate**

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-30.png?featherlight=false&width=90pc)

15. Click **OK**

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-31.png?featherlight=false&width=90pc)

16. Click **Logout** on the left menu

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-32.png?featherlight=false&width=90pc)

17. Click **Sign in**

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-33.png?featherlight=false&width=90pc)

18. Sign in again with the account you just updated.

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-34.png?featherlight=false&width=90pc)

19. You sign in successfully with the updated account.

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-35.png?featherlight=false&width=90pc)

You have finished hosting a static website on S3, authentication with Cognito, storage with S3. Next, we will set up the REST API for the application.