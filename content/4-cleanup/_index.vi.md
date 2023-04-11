---
title : "Dọn dẹp tài nguyên"
date :  "`r Sys.Date()`" 
weight : 4
chapter : false
pre : " <b> 4. </b> "
---
1. Làm rỗng S3 bucket
- Mở bảng điều khiển của [AWS S3](https://s3.console.aws.amazon.com/s3/buckets?region=ap-southeast-1)
- Chọn **fcjdmswebstore**
- Ấn **Empty**
- Nhập **permanently delete**
- Ấn **Empty**
- Làm tương tự với bucket: **aws-sam-cli-managed-default-** và **fcjdmsstore**
2. Chạy các câu lệnh sau tại thư mục gốc của front-end project:
```
amplify remove storage
amplify remove auth
amplify push
amplify delete
```
3. Xoá stack của CloudFormation
- Chạy câu lệnh dưới đây để xoá ứng dụng AWS SAM
```
sam delete --stack-name fcjdmsapp
sam delete --stack-name aws-sam-cli-managed-default
```
