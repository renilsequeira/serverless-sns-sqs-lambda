# Serverless AWS Architecture with SNS, SQS, and Lambda

## Project Overview

This project implements a serverless architecture using AWS services, leveraging Simple Notification Service (SNS), Simple Queue Service (SQS), and AWS Lambda. The architecture is designed to facilitate asynchronous communication and efficient event processing in a scalable and cost-effective manner.

## Image Processing Workflow Overview

![Overview](https://github.com/renilsequeira/serverless-sns-sqs-lambda/assets/77531126/d19feaba-87f9-4b29-a806-e95431bef513)

### User Uploads Image to S3 Bucket:

A user initiates the workflow by uploading an image file to an Amazon S3 bucket. This action triggers an event notification within the S3 bucket.

### S3 Event Notification to SNS Topic:

Upon uploading the image file, an S3 event notification is automatically sent to an Amazon SNS (Simple Notification Service) topic. This topic acts as a central hub for handling notifications.

### SNS Distributes Notifications to SQS Queues:

The SNS topic is configured to distribute notifications to three separate Amazon SQS (Simple Queue Service) queues. Each SQS queue is subscribed to the SNS topic, allowing it to receive relevant notifications.

### Lambda Functions Triggered by SQS Queues:

Each SQS queue triggers a specific AWS Lambda function responsible for image processing. There are three Lambda functions, each designed to process images in a different format.

### Lambda Functions Resize and Save to S3 Bucket:

The Lambda functions receive notifications from the SQS queues and proceed to resize the uploaded images in their respective formats. The resulting resized images are then saved to a different folder within the same S3 bucket.

### Validation of Processed Images:

After the Lambda functions have executed, you can validate the processed images in the designated folders within the S3 bucket. These folders should contain the resized copies of the originally uploaded images.

### Logs in Amazon CloudWatch:

Additionally, logs generated by the Lambda functions during image processing are available in Amazon CloudWatch. You can review these logs for insights into the execution and any potential issues encountered during the image processing workflow.

With this setup, we can process images in various formats simultaneously, making everything super quick and responsive. It's like having a backstage pass to parallel image processing – talk about a game-changer!

## Step 1: Create a Standard Amazon SNS Topic

### Setting up an Amazon SNS Topic

1. **Navigate to Simple Notification Service (SNS):**
   - Open the AWS Console and search for and select "Simple Notification Service."

2. **Choose Topics:**
   - From the options on the left, select "Topics."

3. **Create a New Topic:**
   - Click on "Create topic."

4. **Configure Topic Details:**
   - In the "Create topic" page under "Details," configure as follows:
     - *Type:* Choose "Standard."
     - *Name:* Enter a unique SNS topic name, like "resize-image-topic" 
   - Click "Create topic."

5. **Verify Creation:**
   - Look for the confirmation message. It should be something like:
     ```
     Topic resize-image-topic-1234 created successfully.
     ```

6. **Copy ARN and Topic Owner Values:**
   - In the "Details" section, copy the ARN (Amazon Resource Name) for your topic.
   - Also, grab the AWS account ID of the topic owner.
   - Save these values in notepad for future use.

7. **Note:**
   - Make sure to accurately copy the following values for reference in subsequent tasks of the lab. These details will be the keys to connecting different parts of our project seamlessly.
     - *Topic Name:* [Your Chosen Topic Name]
     - *Topic ARN (Amazon Resource Name):* [Copied ARN]
     - *AWS Account ID (Topic Owner):* [Copied AWS Account ID]

   These values are essential for configuring other components in the workflow. Keep them handy for a smooth setup process.

## Step 2: Create Three Amazon SQS Queues

### Step 2.1: Create an Amazon SQS Queue for the Thumbnail 🖼️

**Objective:** Create an SQS queue named thumbnail-queue for processing thumbnail images.

1. **Navigate to Simple Queue Service (SQS):**

2. **Create a New Queue:**
   - On the SQS home page, choose "Create queue."

3. **Configure Queue Details:**
   - On the "Create queue" page, in the "Details" section:
     - *Type:* Choose "Standard" (default).
     - *Name:* Enter `thumbnail-queue`.
     - Leave the default values for other configuration parameters.
   - Choose "Create queue."

4. **Verify Queue Creation:**
   - Check for the expected service output message:
     ```
     Queue thumbnail-queue created successfully.
     ```

### Step 2.2: Subscribe Thumbnail Queue to SNS Topic 💌

**Objective:** Subscribe the thumbnail-queue to the SNS topic.

1. **Access the SQS Queue Detail Page:**
   - On the queue's detail page, choose the "SNS subscriptions" tab.

2. **Subscribe to Amazon SNS Topic:**
   - Choose "Subscribe to Amazon SNS topic."

3. **Specify SNS Topic:**
   - In the "Use existing resource" section, choose the `resize-image-topic` SNS topic created earlier.
   - Choose "Save."

4. **Verify Subscription:**
   - Check for the expected service output message:
     ```
     Subscribed successfully to topic arn:aws:sns:region:account-id:resize-image-topic-xxxx.
     ```

### Step 2.3: Create and Subscribe Two More SQS Queues 🚀

**Objective:** Create two additional SQS queues (web-queue and mobile-queue) and subscribe them to the SNS topic.

1. **Create SQS Queue for Web-Sized Images:**
   - Follow the same steps as in Step 2.1, but name the queue `web-queue`.

2. **Subscribe Web Queue to SNS Topic:**
   - Follow the steps in Step 2.2 to subscribe `web-queue` to the `resize-image-topic`.

3. **Create SQS Queue for Mobile-Sized Images:**
   - Follow the same steps as in Step 2.1, but name the queue `mobile-queue`.

4. **Subscribe Mobile Queue to SNS Topic:**
   - Follow the steps in Step 2.2 to subscribe `mobile-queue` to the `resize-image-topic`.

5. **Verify Queue and Subscription Creation:**
   - Check for the expected service output messages:
     ```
     Queue web-queue created successfully.
     Subscribed successfully to topic arn:aws:sns:region:account-id:resize-image-topic-xxxx.

     Queue mobile-queue created successfully.
     Subscribed successfully to topic arn:aws:sns:region:account-id:resize-image-topic-xxxx.
     ```

### Step Complete:

You have successfully created three Amazon SQS queues and subscribed them to the `resize-image-topic` SNS topic for receiving image resizing notifications.

## Step 3: Create an Amazon S3 Event Notification

### Step 3.1: Configure Amazon SNS Access Policy 📝

**Objective:** Configure the Amazon SNS access policy to allow the Amazon S3 bucket to publish to the topic.

1. **Access the AWS Management Console:**

2. **Navigate to Simple Notification Service (SNS):**
   - From the left navigation menu, choose "Topics."
   - Select Resize Image Topic:
     - Choose the `resize-image-topic` topic created earlier.
     
4. **Edit Topic Access Policy:**
   - Choose "Edit."
   
5. **Access Policy Configuration:**
   - In the Access policy section, expand it if necessary.
   
6. **Update Access Policy JSON:**
   - Delete the existing content from the JSON editor.
   - Copy and paste the provided code block into the JSON Editor section (replace placeholders).

   ```json
   {
     "Version": "2012-10-17",
     "Id": "__default_policy_ID",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "Service": "s3.amazonaws.com"
         },
         "Action": "SNS:Publish",
         "Resource": "arn:aws:sns:region:account-id:resize-image-topic-xxxx",
         "Condition": {
           "ArnLike": {
             "aws:SourceArn": "arn:aws:s3:::your-ingest-bucket-name"
           }
         }
       }
     ]
   }
   
Replace placeholders:

Replace region with your AWS region, account-id with your AWS account ID.
Replace your-ingest-bucket-name with the name of your ingest S3 bucket.
Save Changes:

Choose "Save."
Note:

Ensure that you replace placeholders such as region, account-id, and your-ingest-bucket-name with your specific AWS region, account ID, and S3 bucket name.
This configuration allows the specified S3 bucket to publish events to the SNS topic.


## Step 3.2: Create an S3 Event Notification for Uploads to the Ingest S3 Bucket 🚀

2. **Navigate to Amazon S3:**

   - On the Buckets page, choose the bucket hyperlink with a name like `xxxxx-bucket-xxxxx`.

4. **Access Bucket Properties:**
   - Choose the "Properties" tab.

5. **Scroll to Event Notifications Section:**
   - Scroll down to the "Event notifications" section.

6. **Create Event Notification:**
   - Choose "Create event notification."

7. **Configure General Settings:**
   - In the General configuration section:
      - Event name: Enter `resize-image-event`.
      - Prefix - optional: Enter `ingest/`.
      - Suffix - optional: Enter `.jpg`.

8. **Configure Event Types:**
   - In the Event types section, select "All object create events."

9. **Configure Destination Settings:**
   - In the Destination section, configure the following:
      - Destination: Select "SNS topic."
      - Specify SNS topic: Select "Choose from your SNS topics."
      - SNS topic: Choose the `resize-image-topic` SNS topic from the dropdown menu.

10. **Alternative Option:**
   - If you prefer to specify an ARN, choose "Enter ARN" and enter the ARN you copied in Task 1.

11. **Save Changes:**
    - Choose "Save changes."

12. **Verify Notification Creation:**
    - Check for the expected service output message:
      ```csharp
      Successfully created event notification “resize-image-event”.
      ```

### Step Complete:
You have successfully created an Amazon S3 event notification for object create events with specific filters, directing notifications to the `resize-image-topic` SNS topic.


## Step 4: Create and Configure Three AWS Lambda Functions

### Step 4.1: Create a Lambda Function to Generate a Thumbnail

**Objective:** Create an AWS Lambda function with an SQS trigger to generate a thumbnail.


2. **Navigate to AWS Lambda:**

3. **Create Lambda Function:**
   - Choose "Create function."

4. **Configure Basic Information:**
   - In the "Create function" window, select "Author from scratch."
   - In the Basic information section, configure the following:
      - Function name: Enter `CreateThumbnail`.
      - Runtime: Choose `Python 3.9`.

5. **Change Default Execution Role:**
   - Expand the "Change default execution role" section.
   - Select "Use an existing role."
   - Choose the role with a name like `xxxxx-LabExecutionRole-xxxxx`.
     - Note: This role provides necessary permissions for Lambda to access Amazon S3 and Amazon SQS.

6. **Create Lambda Function:**
   - Choose "Create function."

7. **Verify Creation:**
   - Check for the expected service output message:
     ```vbnet
     Successfully created the function CreateThumbnail. You can now change its code and configuration. To invoke your function with a test event, choose “Test”.
     ```

### Step Complete:
You have successfully created an AWS Lambda function named CreateThumbnail with the necessary configuration.

## Step 4.2: Configure Lambda Function with SQS Trigger and Upload Deployment Package 

**Objective:** Add an SQS trigger to the Lambda function and upload the Python deployment package.

1. **Access the AWS Lambda Console:**
   - Open the AWS Lambda console.

2. **Navigate to CreateThumbnail Lambda Function:**
   - Select the "CreateThumbnail" Lambda function that you created.

3. **Add SQS Trigger:**
   - Choose "Add trigger."
   - Configure the following:
      - Select a trigger: Choose "SQS."
      - SQS Queue: Choose "thumbnail-queue."
      - Batch Size: Enter 1.
   - Scroll to the bottom of the page and choose "Add."

4. **Verify Trigger Addition:**
   - Check for the expected service output message:
     ```
     The trigger thumbnail-queue was successfully added to function CreateThumbnail. The trigger is in a disabled state.
     ```

5. **Configure Lambda Function Code:**
   - Choose the "Code" tab.
   - Instead of manually entering code, you'll upload it as a deployment package.

6. **Upload Deployment Package:**
   - Follow these steps to upload the deployment package:
      - Save the provided file (CreateThumbnail.zip) to your computer.
      - Open the "Upload from" menu and choose ".zip file."
      - Choose the "Upload" button, navigate to and select the deployment package you saved.
      - Choose "Save."

7. **Verify Package Upload:**
   - Check for the expected service output message:
     ```bash
     Successfully updated the function CreateThumbnail.
     ```
   - Note: The provided CreateThumbnail.zip file contains the Lambda function code. Do not copy this code; it's just an example to show what's in the zip file.

### Step Complete:
You have successfully configured the Lambda function with an SQS trigger and uploaded the deployment package.


## Step 4.3: Configure Lambda Function Runtime Settings and Description

**Objective:** Configure the runtime settings and description for the Lambda function.

1. **Access the AWS Lambda Console:**

2. **Navigate to CreateThumbnail Lambda Function:**
   - Select the "CreateThumbnail" Lambda function that you created.

3. **Edit Runtime Settings:**
   - In the "Runtime settings" section, choose "Edit."
   - In the "Handler" text field, enter `CreateThumbnail.handler`.
     - Caution: Make sure the Handler field is set to this value; otherwise, the Lambda function will not be found.
   - Choose "Save."

4. **Verify Runtime Settings Update:**
   - Check for the expected service output message:
     ```bash
     Successfully updated the function CreateThumbnail.
     ```

5. **Navigate to General Configuration:**
   - Choose the "Configuration" tab.
   - Choose "General configuration" from the panel on the left side of the screen.

6. **Edit Description:**
   - Choose "Edit."
   - In the "Description" text field, enter: "Create a thumbnail-sized image."
   - Leave the other settings at their default values.
   - Save Description Changes: Choose "Save."

7. **Verify Description Update:**
   - Check for the expected service output message:
     ```bash
     Successfully updated the function CreateThumbnail.
     ```

### Step Complete:
You have successfully configured the runtime settings and description for the Lambda function. 





