---
title : "Thiết Lập APIs Và Lambda Function"
date :  "`r Sys.Date()`" 
weight : 2
chapter : false
pre : " <b> 2. </b> "
---

1. Thêm đoạn code sau vào của tệp **template.yaml** trong sam project. Đoạn code mô tả một bảng DynamoDB với *Partition key* là id của user và *Sort key*là tên tệp.
```yaml
  # Create a table to storage document informations
  DocsTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: Documents
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: user_id
          AttributeType: S
        - AttributeName: file
          AttributeType: S
      KeySchema:
        - AttributeName: user_id
          KeyType: HASH
        - AttributeName: file
          KeyType: RANGE
      StreamSpecification:
        StreamViewType: NEW_IMAGE

  # Create a table to storage general informations
  GeneralTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: General
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S
      KeySchema:
        - AttributeName: id
          KeyType: HASH
```

![CreateLambda](/images/2-config-api-and-lambda-function/2-config-api-and-lambda-function-1.png?featherlight=false&width=90pc)

2. Sao chép đoạn code sau vào tệp **template.yaml** để thiết lập 5 Lambda function là: 
    - **upload_doc**
    - **list_docs**
    - **delete_doc**
    - **upload_general_infor**
    - **get_general_infor**
```yaml
  # Lambda function to scan all document by user id
  DocsList:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: lambda/list_docs
      Handler: list_docs.lambda_handler
      Runtime: python3.9
      FunctionName: list_docs
      Architectures:
        - x86_64
      Policies:
        - Statement:
            - Sid: ReadDynamoDB
              Effect: Allow
              Action:
                - dynamodb:Scan
                - dynamodb:Query
              Resource:
                - !GetAtt DocsTable.Arn
      Events:
        ListDocs:
          Type: Api
          Properties:
            Path: /docs/{id}
            Method: get
            RestApiId: !Ref DocApi
      Environment:
        Variables:
          TABLE_NAME: !Ref DocsTable

  # Lambda function to upload documents
  DocsUpload:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: lambda/upload_doc
      Handler: upload_doc.lambda_handler
      Runtime: python3.9
      FunctionName: upload_doc
      Architectures:
        - x86_64
      Policies:
        - Statement:
            - Sid: WriteToDynamoDB
              Effect: Allow
              Action:
                - dynamodb:PutItem
              Resource:
                - !GetAtt DocsTable.Arn
      Events:
        UpoadDocs:
          Type: Api
          Properties:
            Path: /docs
            Method: post
            RestApiId: !Ref DocApi
      Environment:
        Variables:
          TZ: Asia/Jakarta
          TABLE_NAME: !Ref DocsTable

  # Lambda function to delete documents by user id
  DocsDelete:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: lambda/delete_doc
      Handler: delete_doc.lambda_handler
      Runtime: python3.9
      FunctionName: delete_doc
      Architectures:
        - x86_64
      Policies:
        - Statement:
          - Sid: DeleteItems
            Effect: Allow
            Action:
              - dynamodb:DeleteItem
              - dynamodb:GetItem
              - dynamodb:Query
            Resource:
              - !GetAtt DocsTable.Arn
      Events:
        DeleteDoc:
          Type: Api
          Properties:
            Path: /docs/{id}
            Method: delete
            RestApiId: !Ref DocApi
      Environment:
        Variables:
          TABLE_NAME: !Ref DocsTable

  GeneralInforUpload:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: lambda/upload_general_infor
      Handler: upload_general_infor.lambda_handler
      Runtime: python3.9
      FunctionName: upload_general_infor
      Architectures:
        - x86_64
      Policies:
        - Statement:
          - Sid: DeleteItems
            Effect: Allow
            Action:
              - dynamodb:PutItem
            Resource:
              - !GetAtt GeneralTable.Arn
      Events:
        DeleteDoc:
          Type: Api
          Properties:
            Path: /docs/{id}/gen/
            Method: post
            RestApiId: !Ref DocApi
      Environment:
        Variables:
          TABLE_NAME: !Ref GeneralTable

  GeneralInforGet:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: lambda/get_general_infor
      Handler: get_general_infor.lambda_handler
      Runtime: python3.9
      FunctionName: get_general_infor
      Architectures:
        - x86_64
      Policies:
        - Statement:
          - Sid: DeleteItems
            Effect: Allow
            Action:
              - dynamodb:GetItem
              - dynamodb:Query
            Resource:
              - !GetAtt GeneralTable.Arn
      Events:
        DeleteDoc:
          Type: Api
          Properties:
            Path: /docs/{id}/gen/
            Method: get
            RestApiId: !Ref DocApi
      Environment:
        Variables:
          TABLE_NAME: !Ref GeneralTable
```

![CreateLambda](/images/2-config-api-and-lambda-function/2-config-api-and-lambda-function-2.png?featherlight=false&width=90pc)

3. Tải tệp sau về và thêm vào thư mục gốc của sam project. Tệp này mô tả các API mà chúng ta cần.

{{%attachments title="Swagger" pattern=".*\.(yaml)"/%}}

![CreateLambda](/images/2-config-api-and-lambda-function/2-config-api-and-lambda-function-3.png?featherlight=false&width=90pc)

4. Thêm đoạn code sau vào cuối tệp **template.yaml** để import tệp **swagger.yaml**.
```yaml
  # Create REST Api
  DocApi:
    Type: AWS::Serverless::Api
    Properties:
      StageName: dev
      DefinitionBody:
        'Fn::Transform':
          Name: 'AWS::Include'
          Parameters:
            Location: "./swagger.yaml"
```

![CreateLambda](/images/2-config-api-and-lambda-function/2-config-api-and-lambda-function-4.png?featherlight=false&width=90pc)

5. Tạo thư mục và tệp: **lambda/list_docs/list_docs.py**
6. Thêm code sau vào tệp **list_docs.py**
```python
import json
import boto3
import os
from decimal import *
from boto3.dynamodb.types import TypeDeserializer

dynamodb = boto3.client('dynamodb') 
serializer = TypeDeserializer()

class DecimalEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, Decimal):
            return str(obj)
        return json.JSONEncoder.default(self, obj)

def deserialize(data):
    if isinstance(data, list):
        return [deserialize(v) for v in data]

    if isinstance(data, dict):
        try:
            return serializer.deserialize(data)
        except TypeError:
            return {k: deserialize(v) for k, v in data.items()}
    else:
        return data
        
def lambda_handler(event, context):
    table_name = os.environ['TABLE_NAME']
    user_id = event['pathParameters']['id']
    print(user_id)
    docs = dynamodb.query(
        TableName=table_name,
        KeyConditionExpression="user_id = :id",
        ExpressionAttributeValues={ ":id": { 'S': user_id } }
    )
    
    format_data_docs = deserialize(docs["Items"])
    # TODO implement
    return {
        "statusCode": 200,
        "headers": {
            "Content-Type": "application/json",
            "Access-Control-Allow-Origin": "*",
            "Access-Control-Allow-Methods": "GET,PUT,POST,DELETE, OPTIONS",
            "Access-Control-Allow-Headers": "Access-Control-Allow-Headers, Origin,Accept, X-Requested-With, Content-Type, Access-Control-Request-Method,X-Access-Token,XKey,Authorization"
        },
        "body": json.dumps(format_data_docs, cls=DecimalEncoder)
    }

```

![CreateLambda](/images/2-config-api-and-lambda-function/2-config-api-and-lambda-function-5.png?featherlight=false&width=90pc)

7. Tương tự tạo thư mục và tệp **upload_doc/upload_doc.py** trong thư mục **lambda**. Tiếp theo sao chép code dưới đây cho tệp đã tạo.
```python
import json
import boto3
import os
from datetime import datetime, timezone

dynamodb = boto3.resource('dynamodb')
client_cloudwatch = boto3.client('cloudwatch')

def lambda_handler(event, context):
    table_name = os.environ['TABLE_NAME']
    now = datetime.now(tz=timezone.utc)
    dt_string = now.strftime("%d/%m/%Y %H:%M:%S")
    doc_data = json.loads(event["body"])

    path = "protected/{}/{}".format(doc_data['identityId'], doc_data['file'])
    doc_data.update({"path": path, "modified": dt_string})
    table = dynamodb.Table(table_name)
    table.put_item(Item = doc_data)
        
    # TODO implement
    return {
        'statusCode': 200,
        'body': 'successfully upload!',
        'headers': {
            'Content-Type': 'application/json',
            "Access-Control-Allow-Headers": "Access-Control-Allow-Headers, Origin, Accept, X-Requested-With, Content-Type, Access-Control-Request-Method,X-Access-Token, XKey, Authorization",
            "Access-Control-Allow-Origin": "*",
            "Access-Control-Allow-Methods": "GET,PUT,POST,DELETE,OPTIONS"
        }
    }
```

![CreateLambda](/images/2-config-api-and-lambda-function/2-config-api-and-lambda-function-6.png?featherlight=false&width=90pc)

- Tương tự với tệp **lambda/delete_doc/delete_doc.py**
```python
import json
import boto3
import os

client = boto3.resource('dynamodb')

def lambda_handler(event, context):
    # TODO implement
    table_name = os.environ['TABLE_NAME']
    error = None
    doc_pk = event['pathParameters']['id']
    print("doc_pk ", doc_pk)
    doc_sk = event['queryStringParameters']['file']
    print("doc_sk ", doc_sk)
    table = client.Table(table_name)
    key = {
        'user_id':doc_pk,
        'file':  doc_sk
    }
    
    try:
        table.delete_item(Key = key)
    except Exception as e:
        error = e
        
        
    except Exception as e:
        error = e
        
    if error is None:
        message = 'successfully document item!'
    else:
        print(error)
        message = 'delete document fail'
    
    return {
            'statusCode': 200,
            'body': message,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*'
            },
        }

```

![CreateLambda](/images/2-config-api-and-lambda-function/2-config-api-and-lambda-function-7.png?featherlight=false&width=90pc)

- Tương tự với tệp **lambda/upload_general_infor/upload_general_infor.py**

```
import json
import boto3
import os

dynamodb = boto3.resource('dynamodb')

def lambda_handler(event, context):
    table_name = os.environ['TABLE_NAME']
    data = json.loads(event["body"])
    table = dynamodb.Table(table_name)
    data.update({"id": event['pathParameters']['id']})
    table.put_item(Item = data)

    # TODO implement
    return {
        'statusCode': 200,
        'body': 'successfully upload!',
        'headers': {
            'Content-Type': 'application/json',
            "Access-Control-Allow-Headers": "Access-Control-Allow-Headers, Origin, Accept, X-Requested-With, Content-Type, Access-Control-Request-Method,X-Access-Token, XKey, Authorization",
            "Access-Control-Allow-Origin": "*",
            "Access-Control-Allow-Methods": "GET,PUT,POST,DELETE,OPTIONS"
        }
    }

```

![CreateLambda](/images/2-config-api-and-lambda-function/2-config-api-and-lambda-function-8.png?featherlight=false&width=90pc)

- Tương tự với tệp **lambda/get_general_infor/get_general_infor.py**
```
import json
import boto3
import os
from decimal import *
from boto3.dynamodb.types import TypeDeserializer

dynamodb = boto3.client('dynamodb') 
serializer = TypeDeserializer()
    
class DecimalEncoder(json.JSONEncoder):
    def default(self, obj):
        if isinstance(obj, Decimal):
            return str(obj)
        return json.JSONEncoder.default(self, obj)
        
def deserialize(data):
    if isinstance(data, list):
        return [deserialize(v) for v in data]

    if isinstance(data, dict):
        try:
            return serializer.deserialize(data)
        except TypeError:
            return {k: deserialize(v) for k, v in data.items()}
    else:
        return data
        
def lambda_handler(event, context):
    # TODO implement
    table_name = os.environ['TABLE_NAME']
    user_id = event['pathParameters']['id']
    print(user_id)
    data = dynamodb.query(
        TableName=table_name,
        KeyConditionExpression="id = :id",
        ExpressionAttributeValues={ ":id": { 'S': user_id } }
    )
    
    format_data = deserialize(data["Items"])

    return {
        "statusCode": 200,
        "headers": {
            "Content-Type": "application/json",
            "Access-Control-Allow-Origin": "*",
            "Access-Control-Allow-Methods": "GET,PUT,POST,DELETE, OPTIONS",
            "Access-Control-Allow-Headers": "Access-Control-Allow-Headers, Origin,Accept, X-Requested-With, Content-Type, Access-Control-Request-Method,X-Access-Token,XKey,Authorization"
        },
        "body": json.dumps(format_data, cls=DecimalEncoder)
        #"body": format_data
    }

```

![CreateLambda](/images/2-config-api-and-lambda-function/2-config-api-and-lambda-function-9.png?featherlight=false&width=90pc)

8. Chạy câu lệnh dưới đây để build và deploy sam project sau khi cập nhật template:
```
sam build
sam deploy
```

Trong lúc đợi CloudFormation hoàn thành, bạn có thể tìm hiểu về tệp **swagger.yaml**. Phần tiếp theo chúng ta sẽ thực hiện các thao tác trên Front-end để kiểm tra hoạt động của các API.