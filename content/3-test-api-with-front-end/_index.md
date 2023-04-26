---
title : "Test API With Front-end"
date :  "`r Sys.Date()`" 
weight : 3
chapter : false
pre : " <b> 3. </b> "
---
We will perform the document upload, document download, and delete document functions to test the working of the APIs.

1. Open the console of **fcjdms** stack, click to id of **DocAPI**

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-1.png?featherlight=false&width=90pc)

2. Select **Stages**, then expand **dev** stage and record the API's URL

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-2.png?featherlight=false&width=90pc)

3. Open **constant.js** file in **src** folder of front-end project.
4. Relpace value of **APP_API_URL** with your URL:

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-3.png?featherlight=false&width=90pc)

- Open the **src/component/Home/Upload.js** file in the application's source code directory and uncomment the code that calls the API to write data to DynamoDB.

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-3-1.png?featherlight=false&width=90pc)

5. Build and upload build folder to s3 with the following commmand:
```
yarn build
aws s3 cp build s3://BUCKET_NAME --recursive
```
Replace `BUCKET_NAME` with the name of the bucket you created to host the website

6. Back to web application and reload web page.
- Click **Upload**

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-4.png?featherlight=false&width=90pc)

7. Click **Add files** and select the files you want to upload. Then click **Upload**

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-5.png?featherlight=false&width=90pc)

- Select **My Profile** to view information about file numbers and used storage

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-6.png?featherlight=false&width=90pc)

8. Select **My Document** on the left menu to view all uploaded files. Then click **Select**

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-7.png?featherlight=false&width=90pc)

9. Check to files that you want to delete and click **Delete**

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-8.png?featherlight=false&width=90pc)

- Click **OK** to confirm deletion

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-10.png?featherlight=false&width=90pc)

10. You will see the deleted files and you can download them if you click the **Download** icon

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-10.png?featherlight=false&width=90pc)

11. Select **Home** on the left menu, you will see the information has been updated

![TestAPI](/images/3-test-api-with-front-end/3-test-api-with-front-end-11.png?featherlight=false&width=90pc)

You can open the S3 bucket and DynamoDB table to examine objects and data. We have completed the deployment of a serverless application with SAM.