---
title : "Serverless - Deploying Applications with AWS SAM"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
---
# Serverless - Deploying Applications with AWS SAM

#### Overview

In the first two articles of this series, we know how to build a simple serverless web application using the AWS console. To build the serverless application faster, AWS provides users with the AWS Serverless Application Model (SAM) service, which is an open-source framework for building a serverless applications. It provides shorthand syntax to express functions, APIs, databases, and event source mappings. You define the application you want with just a few lines per resource and model it using YAML. During deployment, SAM transforms and expands the SAM syntax into AWS CloudFormation syntax. Then, CloudFormation provides the resources to the user.

In this workshop, we will rebuild the previous web application with AWS SAM.

The architecture of the application we will build:

![WebArchitect](/images/serverless-architect-diagram.png?featherlight=false&width=50pc)

#### Content

1. [Authentication And Storage](1-authenication-and-storage/)
2. [Config APIs And Lambda Function](2-config-api-and-lambda-function/)
3. [Test API With Front-end](3-test-api-with-front-end/)
4. [Cleanup](4-cleanup)