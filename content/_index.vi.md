---
title : "Serverless - Triển khai ứng dụng với SAM"
date :  "`r Sys.Date()`" 
weight : 1 
chapter : false
---
# Serverless - Triển khai ứng dụng với SAM

### Tổng quan

Trong các bài trước của series này, chúng ta đã tìm hiểu cách xây dựng một ứng dụng web đơn giản theo mô hình serverless trên cách sử dụng bảng điều khiển của AWS. Để xây dựng ứng dụng serverless nhanh hơn, AWS cung cấp cho người dùng dịch vụ AWS Serverless Application Model (SAM) - là một open-source framework để xây dựng một ứng dụng serverless. SAM cung cấp cú pháp để diễn đạt các hàm, API, cơ sở dữ liệu và event source mappings. Chỉ với vài dòng cho mỗi tài nguyên, bạn có thể xác định ứng dụng bạn muốn và lập mô hình ứng dụng đó bằng YAML. Trong quá trình triển khai, SAM chuyển đổi và mở rộng cú pháp SAM thành cú pháp của AWS CloudFormation. Sau đó, CloudFormation cung cấp các tài nguyên cho người dùng.  

Trong bài này, chúng ta sẽ cùng nhau xây dựng lại ứng dụng web ở bài trước với AWS SAM.

Kiến trúc của ứng dụng chúng ta sẽ xây dựng:

![WebArchitect](/images/serverless-architect-diagram.png?featherlight=false&width=50pc)

### Nội dung
1. [Xác thực và lưu trữ](1-authenication-and-storage/)
2. [Thiết lập APIs và Lambda function](2-config-api-and-lambda-function/)
3. [Kiểm tra API với Front-end](3-test-api-with-front-end/)
4. [Dọn dẹp tài nguyên](4-cleanup)