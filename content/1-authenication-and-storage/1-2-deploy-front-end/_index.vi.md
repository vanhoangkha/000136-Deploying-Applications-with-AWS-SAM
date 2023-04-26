---
title : "Triển Khai Front-end"
date :  "`r Sys.Date()`" 
weight : 2
chapter : false
pre : " <b> 1.2 </b> "
---
1. Thêm đoạn code sau tại phần **Prarameters** của tệp **template.yaml** trong sam project:
```yaml
  WebStoreBucketName:
    Type: String
    Default: fcjdmswebstore
```
{{% notice note %}}
Thay đổi giá trị **Default** để thay đổi tên cho bucket
{{% /notice %}}

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-18.png?featherlight=false&width=90pc)


2. Thêm đoạn code sau bên dưới mô tả S3 bucket ở phần 1.1. Đoạn code này mô tả bucket cho phép truy cập public và enable hosting website:
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

3. Chạy câu lệnh sau để build và deploy lại sam project:
```
sam build
sam deploy
```

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-20.png?featherlight=false&width=90pc)

4. Quay lại với thư mục gốc của Front-end project, chạy câu lệnh sau:
```
yarn build
aws s3 cp build s3://BUCKET_NAME --recursive
```
Thay thế `BUCKET_NAME` bằng tên bucket bạn vừa tạo để host website, ví dụ: `aws s3 cp build s3://fcjdmswebstore --recursive`

5. Mở bảng điều khiển của [Amazon S3](https://s3.console.aws.amazon.com/s3/buckets?region=ap-southeast-1)
- Chọn bucket vừa tạo

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-21.png?featherlight=false&width=90pc)

6. Chọn tab **Properties** và kéo xuống cuối cùng. Ấn vào URL của website

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-22.png?featherlight=false&width=90pc)

7. Ấn **Sign up**

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-23.png?featherlight=false&width=90pc)

8. Nhập tên người dùng, email và mật khẩu. Sau đó ấn **Sign up**

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-24.png?featherlight=false&width=90pc)

9. Mở email để lấy verify code và nhập vào trang web. Sau đó ấn **Submit**

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-25.png?featherlight=false&width=90pc)

10. Quay lại với bảng điều khiển của Cognito User Pool, ấn sang tab **User** bạn sẽ thấy tài khoản mình vừa đăng ký.

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-26.png?featherlight=false&width=90pc)

11. Đăng nhập vào website bằng tài khoản vừa đăng ký. 

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-27.png?featherlight=false&width=90pc)

12. Đăng nhập thành công

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-28.png?featherlight=false&width=90pc)

13. Chọn **My Profile** ở menu phía bên trái, tiếp theo ấn **Update profile**.

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-29.png?featherlight=false&width=90pc)

14. Nhập mật khẩu cũ và mới, sau đó ấn **Upadate**

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-30.png?featherlight=false&width=90pc)

15. Ấn **OK**

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-31.png?featherlight=false&width=90pc)

16. Ấn **Sign out** ở menu phía bên trái

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-32.png?featherlight=false&width=90pc)

17. Ấn **Sign in**

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-33.png?featherlight=false&width=90pc)

18. Đăng nhập lại với tài khoản mà bạn vừa cập nhật.

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-34.png?featherlight=false&width=90pc)

19. Bạn đã đăng nhập thành công với thông tin tài khoản mới.

![AddAmplify](/images/1-authenication-and-storage/1-authenication-and-storage-35.png?featherlight=false&width=90pc)

Bạn đã hoàn thành việc host một website tĩnh trên S3, xác thực người dùng với Cognito, lưu trữ tài liệu trên S3. Tiếp theo chúng tã sẽ thiết lập REST API cho ứng dụng.