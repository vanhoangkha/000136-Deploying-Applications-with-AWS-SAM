---
title : "Deploy Cognito And S3 Bucket"
date :  "`r Sys.Date()`" 
weight : 1
chapter : false
pre : " <b> 1.1 </b> "
---
In this section, we will create a Cognito User Pool, Identity Pool, and S3 bucket that stores file uploads with SAM:

{{% notice note %}}
You need install [SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html) and configure credentials before doing this part.
{{% /notice %}}

1. Run the following command to initialize a SAM project:
```
#Step 1 - Download a sample application
sam init
```

- Select the following information:
```shell
  Which template source would you like to use?
	  1 - AWS Quick Start Templates
	  2 - Custom Template Location
  Choice: 1

  Choose an AWS Quick Start application template
	  1 - Hello World Example
	  2 - Multi-step workflow
	  3 - Serverless API
	  4 - Scheduled task
	  5 - Standalone function
	  6 - Data processing
	  7 - Infrastructure event management
	  8 - Machine Learning
  Template: 1

  Use the most popular runtime and package type? (Python and zip) [y/N]: y
  Would you like to enable X-Ray tracing on the function(s) in your application?  [y/N]: n
  Project name [sam-app] : fcjdmssam
```

![SAMInit](/images/1-authenication-and-storage/1-authenication-and-storage-1.png?featherlight=false&width=90pc)

2. Open SAM project by IDE. Open **template.yaml** file.
- Delete unnecessary part

![SAMInit](/images/1-authenication-and-storage/1-authenication-and-storage-2.png?featherlight=false&width=90pc)

3. Copy the following code into the **Resources** section:
```yml
  # Creates a user pool in cognito for your app to auth against
  FcjDMSUserPool:
    Type: AWS::Cognito::UserPool
    Properties:
      UserPoolName: "cognito-fcj-dms"
      MfaConfiguration: "OFF"
      AliasAttributes:
        - preferred_username
        - email
      AutoVerifiedAttributes: 
        - email
      EmailConfiguration:
        EmailSendingAccount: COGNITO_DEFAULT
      Policies:
        PasswordPolicy:
          MinimumLength: 8
          RequireLowercase: true
          RequireNumbers: true
          RequireSymbols: false
          RequireUppercase: true
          TemporaryPasswordValidityDays: 30
      UserAttributeUpdateSettings:
        AttributesRequireVerificationBeforeUpdate:
          - email
          
  # Creates a User Pool Client to be used by the identity pool
  FcjDMSUserPoolClient:
    Type: AWS::Cognito::UserPoolClient
    Properties:
      ClientName: "fcj-dms"
      UserPoolId: !Ref FcjDMSUserPool
      GenerateSecret: false
      ExplicitAuthFlows:
        - ALLOW_USER_PASSWORD_AUTH
        - ALLOW_CUSTOM_AUTH
        - ALLOW_USER_SRP_AUTH
        - ALLOW_REFRESH_TOKEN_AUTH
  
  # Creates a federeated Identity pool
  FcjDMSUserPoolIdentityPool:
    Type: "AWS::Cognito::IdentityPool"  
    Properties:
      IdentityPoolName: "fcj-dms-identity"
      AllowUnauthenticatedIdentities: true
      CognitoIdentityProviders: 
        - ClientId: !Ref FcjDMSUserPoolClient
          ProviderName: !GetAtt FcjDMSUserPool.ProviderName
```
This code block used to initialize resources:
- A User Pool: Allow users sign in by username, authenticate account by email and config password policies.
- A User Pool Client: Allow users sign in with a combination password and user name and integrate with User Pool.
- A Identity Pool: Supports unauthenticated credentials and user pool and client ID settings.

![CreateBucket](/images/1-authenication-and-storage/1-authenication-and-storage-3.png?featherlight=false&width=90pc)


4. Run below command to build project:
```
sam build
```
![CreateUserPool](/images/1-authenication-and-storage/1-authenication-and-storage-4.png?featherlight=false&width=90pc)

- Run the below command to check if the SAM template is valid or not:
```
sam validate
```

![CreateUserPool](/images/1-authenication-and-storage/1-authenication-and-storage-5.png?featherlight=false&width=90pc)

- Run the below command to deploy SAM:
```
sam deploy --guided
```

- Enter stack name: `fcjdmsapp`
- Enter region you want deploy, such as: `ap-southeast-1`
- hen, enter other information as the below figure:

![CreateUserPool](/images/1-authenication-and-storage/1-authenication-and-storage-6.png?featherlight=false&width=90pc)

- Wait a while to create CloudFormation stack changeset
- Enter `y` when is ask **Deploy this changeset?**

![CreateUserPool](/images/1-authenication-and-storage/1-authenication-and-storage-7.png?featherlight=false&width=90pc)

- The result after CloudFormation is completed:

![CreateUserPool](/images/1-authenication-and-storage/1-authenication-and-storage-8.png?featherlight=false&width=90pc)

4. Open [CloudFormation console](https://ap-southeast-1.console.aws.amazon.com/cloudformation/home?region=ap-southeast-1#/). 
- Select **Stack details** on the left menu, then select **fcjdmsapp** stack.
- Select **Resources** tab. You will see the resources we described in the template.
- Click **FcjDmsUserPool** to navigate to this User Pool console.

![CheckUserPool](/images/1-authenication-and-storage/1-authenication-and-storage-9.png?featherlight=false&width=90pc)

5. Next, we will create a S3 bucket to store files that the user uploads
- Open **template** file and add the following code to declare the parameters:: 
```yml
Parameters:
  DocumentStoreBucketName:
    Type: String
    Default: fcjdmsstore
```
{{% notice note %}}
Change the **Default** value to change bucket name
{{% /notice %}}

![CreateBucket](/images/1-authenication-and-storage/1-authenication-and-storage-10.png?featherlight=false&width=90pc)

- Add the following code to file:
```yml
  FcjDMSStore:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Ref DocumentStoreBucketName
      CorsConfiguration:
        CorsRules:
          - AllowedHeaders:
              - '*'
            AllowedMethods:
              - GET
              - HEAD
              - PUT
              - POST
              - DELETE
            AllowedOrigins:
               - '*'
            ExposedHeaders:
              - x-amz-server-side-encryption
              - x-amz-request-id
              - x-amz-id-2
              - ETag
            MaxAge: 1800
```
With the above code, the S3 bucket is configured CorsRules which allows our web application access to it.

![CreateBucket](/images/1-authenication-and-storage/1-authenication-and-storage-11.png?featherlight=false&width=90pc)

6. Next, we will create the roles for unauthorized and authorized access to S3. Add the following code to the bottom file:
```yml
  # Create a role for unauthorized access to AWS resources. Very limited access. Only allows users in the previously created
  CognitoUnAuthorizedRole:
    Type: "AWS::IAM::Role"
    Properties:
      AssumeRolePolicyDocument: 
        Version: "2012-10-17"
        Statement:
          - Effect: "Allow"
            Principal: 
              Federated: "cognito-identity.amazonaws.com"
            Action: 
              - "sts:AssumeRoleWithWebIdentity"
            Condition:
              StringEquals: 
                "cognito-identity.amazonaws.com:aud": !Ref FcjDMSUserPoolIdentityPool
              "ForAnyValue:StringLike":
                "cognito-identity.amazonaws.com:amr": unauthenticated
      Policies:
        - PolicyName: "CognitoUnauthorizedPolicy"
          PolicyDocument: 
            Version: "2012-10-17"
            Statement: 
              - Effect: "Allow"
                Action:
                  - "mobileanalytics:PutEvents"
                  - "cognito-sync:*"
                Resource: "*"
              - Effect: "Allow"
                Action:
                  - "s3:GetObject"
                Resource: !Join
                  - ""
                  - - "arn:aws:s3:::"
                    - !Ref DocumentStoreBucketName
                    - /protected/*
  
  # Create a role for authorized access to AWS resources. Control what your user can access. This example only allows Lambda invokation
  # Only allows users in the previously created Identity Pool
  CognitoAuthorizedRole:
    Type: "AWS::IAM::Role"
    Properties:
      AssumeRolePolicyDocument: 
        Version: "2012-10-17"
        Statement:
          - Effect: "Allow"
            Principal: 
              Federated: "cognito-identity.amazonaws.com"
            Action: 
              - "sts:AssumeRoleWithWebIdentity"
            Condition:
              StringEquals: 
                "cognito-identity.amazonaws.com:aud": !Ref FcjDMSUserPoolIdentityPool
              "ForAnyValue:StringLike":
                "cognito-identity.amazonaws.com:amr": authenticated
      Policies:
        - PolicyName: "CognitoAuthorizedPolicy"
          PolicyDocument: 
            Version: "2012-10-17"
            Statement: 
              - Effect: "Allow"
                Action:
                  - "mobileanalytics:PutEvents"
                  - "cognito-sync:*"
                  - "cognito-identity:*"
                Resource: "*"
              - Effect: "Allow"
                Action:
                  - "lambda:InvokeFunction"
                  - "s3:GetObject"
                  - "s3:PutObject"
                  - "s3:DeleteObject"
                Resource: '*'
              - Effect: "Allow"
                Action:
                  - "s3:GetObject"
                Resource: !Join
                  - ""
                  - - "arn:aws:s3:::"
                    - !Ref DocumentStoreBucketName
                    - /protected/*
  
  # Assigns the roles to the Identity Pool
  IdentityPoolRoleMapping:
    Type: "AWS::Cognito::IdentityPoolRoleAttachment"
    Properties:
      IdentityPoolId: !Ref FcjDMSUserPoolIdentityPool
      Roles:
        authenticated: !GetAtt CognitoAuthorizedRole.Arn
        unauthenticated: !GetAtt CognitoUnAuthorizedRole.Arn
```

![CreateBucket](/images/1-authenication-and-storage/1-authenication-and-storage-12.png?featherlight=false&width=90pc)

7. Run the following commands:
```
sam build
sam deploy
``` 

![CreateBucket](/images/1-authenication-and-storage/1-authenication-and-storage-13.png?featherlight=false&width=90pc)

8. Next, we will use amplify to import authentication and storage into our web application. Run the following command at the root of the front-end project.
```
amplify init
```
- Entering follow the below information:

    ? Enter a name for the project `fcjdms`\
    The following configuration will be applied:

    Project information\
    | Name: fcjdms\
    | Environment: dev\
    | Default editor: Visual Studio Code\
    | App type: javascript\
    | Javascript framework: react\
    | Source Directory Path: src\
    | Distribution Directory Path: build\
    | Build Command: npm run-script build\
    | Start Command: npm run-script start

    ? Initialize the project with the above configuration? `Yes`\
    Using default provider  awscloudformation\
    ? Select the authentication method you want to use: *AWS profile*

    For more information on AWS Profiles, see:\
    https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html

    ? Please choose the profile you want to use *default*\
    ? Help improve Amplify CLI by sharing non sensitive configurations on failures (y/N) › `No`

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-14.png?featherlight=false&width=90pc)

{{% notice note %}}
If you have not downloaded the front-end project, run the following commands:\
`git clone https://github.com/AWS-First-Cloud-Journey/FCJ-Serverless-DMS`\
`cd FCJ-Serverless-DMS`\
`npm install`
{{% /notice %}}

9. Run the following command to import authentication into project:
```
amplify import auth
```
- Select **Cognito User Pool and Identity Pool** for *What type of auth resource do you want to import?*

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-15.png?featherlight=false&width=90pc)

10. Run the following command to import storage into project:
```
amplify import storage
```
- Select **S3 bucket - Content (Images, audio, video, etc.)** for *Select from one of the below mentioned services*
- Select the bucket you created from the above steps

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-16.png?featherlight=false&width=90pc)

10. Run the command: `amplify push` to update cloud resources:

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-17.png?featherlight=false&width=90pc)

