---
title : "Kiểm Tra API Với Front-end"
date :  "`r Sys.Date()`" 
weight : 3
chapter : false
pre : " <b> 3. </b> "
---
Chúng ta sẽ thực hiện các chức năng tải tài liệu lên, tải tài liệu về và xoá tài liệu để kiểm tra hoạt động của các API.

1. Mở bảng điều khiển của stack **fcjdms**, ấn vào id của **DocAPI**

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-1.png?featherlight=false&width=90pc)

2. Chọn **Stages**, sau đó mở rộng stage **dev** và ghi lại URL của API

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-2.png?featherlight=false&width=90pc)

3. Mở tệp **constant.js** trong thư mục **src** của front-end project đã được tải về từ phần 2
4. Thay giá trị cho **APP_API_URL** bằng URL của bạn:

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-3.png?featherlight=false&width=90pc)

- Mở tệp **src/component/Home/Upload.js** trong thư mục source code của ứng dụng và bỏ comment đoạn code gọi API ghi dữ liệu vào DynamoDB.

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-3-1.png?featherlight=false&width=90pc)

5. Build và đẩy thư mục build lên s3 với câu lệnh sau:
```
yarn build
aws s3 cp build s3://BUCKET_NAME --recursive
```
Thay thế `BUCKET_NAME` bằng tên bucket bạn tạo để host website

6. Trở lại với trình duyệt web với ứng dụng của bạn và reload lại trang.
- Ấn **Upload**

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-4.png?featherlight=false&width=90pc)

7. Ấn **Add files** và chọn các tệp mà bạn muốn tải lên. Sau đó ấn **Upload**

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-5.png?featherlight=false&width=90pc)

- Chọn **My Profile** để xem thông tin các số tệp và dung lượng đã dùng

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-6.png?featherlight=false&width=90pc)

8. Chọn **My Document** ở menu phía bên trái để xem các tệp đã tải lên. Sau đó ấn **Select**

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-7.png?featherlight=false&width=90pc)

9. Chọn các tệp mà bạn muốn xoá và ấn **Delete**

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-8.png?featherlight=false&width=90pc)

- Ấn **OK** để xác nhận xoá

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-9.png?featherlight=false&width=90pc)

10. Bạn sẽ thấy các tệp đã được xoá và bạn có thể tải tệp về máy nếu ấn vào biểu tượng **Download**

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-10.png?featherlight=false&width=90pc)

11. Chọn **Home** ở menu phía bên trái, bạn sẽ thấy thông tin đã được cập nhật

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-11.png?featherlight=false&width=90pc)

Bạn có thể mở S3 bucket và DynamoDB table để kiểm tra các object và dữ liệu. Chúng ta đã hoàn thành việc triển khai ứng dụng serverless với SAM.