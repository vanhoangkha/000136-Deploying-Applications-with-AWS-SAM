---
title : "Dọn dẹp tài nguyên"
date :  "`r Sys.Date()`" 
weight : 4
chapter : false
pre : " <b> 4. </b> "
---
1. Làm rỗng S3 bucket
- Chạy câu lệnh sau: `aws s3 rm s3://BUCKET_NAME --recursive`
- Thay thế `BUCKET_NAME` bằng tên bucket cần làm rỗng: `fcjdmswebstore`, `fcjdmsstore`
2. Chạy các câu lệnh sau tại thư mục gốc của front-end project:
```
amplify remove storage
amplify remove auth
amplify push
amplify delete
```
3. Xoá stack của CloudFormation
- Chạy câu lệnh dưới đây để xoá ứng dụng AWS SAM: `sam delete --stack-name fcjdmsapp`
- Chạy câu lệnh sau để làm rỗng S3 bucket: `aws s3 rm s3://BUCKET_NAME --recursive`
- Thay thế `BUCKET_NAME` bằng tên bucket bắt đầu bằng: **aws-sam-cli-managed-default-samclisourcebucket-**
- Mở bảng điều khiển của [Amazon S3](https://ap-southeast-1.console.aws.amazon.com/s3/buckets?region=ap-southeast-1)
- Chọn bucket bắt đầu bằng **aws-sam-cli-managed-default-samclisourcebucket-**
- Ấn **Show versions**
- Chọn thư mục **fcjdmsapp**
- Chọn tất cả các object
- Ấn **Delete**
- Nhập `permanently delete`
- Ấn **Delete**
- Chạy câu lệnh sau để xoá stack:
`sam delete --stack-name aws-sam-cli-managed-default`