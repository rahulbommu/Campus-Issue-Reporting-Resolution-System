# Campus Issue Reporting & Resolution System - Complete AWS Deployment Guide

## 🏗️ Architecture Overview

```
User (Browser)
    ↓
S3 Static Website (HTML/JS)
    ↓
API Gateway (REST API)
    ↓
Lambda Functions (Python)
    ↓
DynamoDB (Issue Storage)
    ↓
SES (Email Notifications)
    ↓
CloudWatch (Logging)
```

---

## 📋 Prerequisites & Setup

### 1. AWS Account Setup

- Create AWS Free Tier account if you don't have one
- Ensure you're in a supported region (recommend `us-east-1` or `ap-south-1`)
- Have your AWS credentials ready

### 2. Local Setup

- Download all project files to your local machine
- Have a text editor ready for making small configuration changes

---

## 🚀 Step-by-Step AWS Deployment

### PHASE 1: DynamoDB Table Creation

#### Step 1.1: Create DynamoDB Table

1. **Login to AWS Console** → Navigate to **DynamoDB**
2. Click **"Create table"**
3. **Configure table settings:**
   ```
   Table name: CampusIssues
   Partition key: issueId (String)
   ```
4. **Table settings:**
   - Billing mode: **On-demand**
   - Leave all other settings as default
5. Click **"Create table"**
6. **Wait** for table status to show **"Active"** (usually 1-2 minutes)

#### Step 1.2: Note Down Table ARN

1. Click on your **CampusIssues** table
2. Go to **"General information"** tab
3. **Copy the Table ARN** - you'll need this later
   - Format: `arn:aws:dynamodb:region:account-id:table/CampusIssues`

---

### PHASE 2: SES Email Configuration

#### Step 2.1: Verify Email Addresses

1. **Navigate to AWS SES** (Simple Email Service)
2. **IMPORTANT:** Ensure you're in the same region as other services
3. Go to **"Verified identities"**
4. Click **"Create identity"**
5. **Select:** Email address
6. **Enter your email** (e.g., your Gmail, Yahoo, etc.)
7. Click **"Create identity"**
8. **Check your email** and click the verification link
9. **Repeat this process** for both sender and receiver emails (can be the same)

#### Step 2.2: Test Email Sending (Optional but Recommended)

1. Go to **"Test email sending"**
2. Send a test email to verify SES is working

---

### PHASE 3: IAM Role Configuration

#### Step 3.1: Create IAM Role for Lambda

1. **Navigate to AWS IAM** → **Roles**
2. Click **"Create role"**
3. **Select trusted entity:** AWS service
4. **Choose service:** Lambda
5. Click **"Next"**

#### Step 3.2: Attach Basic Lambda Policy

1. **Search for:** `AWSLambdaBasicExecutionRole`
2. **Select it** and click **"Next"**
3. **Role name:** `CampusIssueLambdaRole`
4. Click **"Create role"**

#### Step 3.3: Add Custom DynamoDB + SES Policy

1. **Find your newly created role** → Click on it
2. Click **"Add permissions"** → **"Create inline policy"**
3. Click **"JSON"** tab
4. **Replace the content** with this exact policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["dynamodb:PutItem", "dynamodb:GetItem", "dynamodb:UpdateItem"],
      "Resource": "arn:aws:dynamodb:*:*:table/CampusIssues"
    },
    {
      "Effect": "Allow",
      "Action": "ses:SendEmail",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    }
  ]
}
```

5. Click **"Next"**
6. **Policy name:** `CampusIssueDynamoSESPolicy`
7. Click **"Create policy"**

---

### PHASE 4: Lambda Functions Deployment

#### Step 4.1: Create "Create Issue" Lambda Function

1. **Navigate to AWS Lambda**
2. Click **"Create function"**
3. **Configuration:**
   ```
   Function name: CreateCampusIssue
   Runtime: Python 3.11
   Architecture: x86_64
   Execution role: Use existing role → CampusIssueLambdaRole
   ```
4. Click **"Create function"**

#### Step 4.2: Upload Create Issue Code

1. **In the Lambda function page,** scroll to **"Code source"**
2. **Delete the default code**
3. **Copy and paste** the entire content from `lambdas/create_issue.py`
4. **CRITICAL:** Update the email addresses in the code:
   ```python
   SENDER = "your_verified_email@gmail.com"  # Your verified email
   RECEIVER = "your_verified_email@gmail.com"  # Same or different verified email
   ```
5. Click **"Deploy"**

#### Step 4.3: Configure Create Issue Function

1. Go to **"Configuration"** tab → **"General configuration"**
2. Click **"Edit"**
3. **Set timeout:** 30 seconds
4. **Set memory:** 256 MB
5. Click **"Save"**

#### Step 4.4: Create "Get Issue" Lambda Function

1. **Repeat Step 4.1** with these settings:
   ```
   Function name: GetCampusIssue
   Runtime: Python 3.11
   Execution role: CampusIssueLambdaRole
   ```
2. **Copy code** from `lambdas/get_issue.py`
3. **Deploy and configure** (same as steps 4.2-4.3)

#### Step 4.5: Create "Update Status" Lambda Function

1. **Repeat Step 4.1** with these settings:
   ```
   Function name: UpdateCampusIssueStatus
   Runtime: Python 3.11
   Execution role: CampusIssueLambdaRole
   ```
2. **Copy code** from `lambdas/update_status.py`
3. **Update email addresses** in the code (same as Step 4.2)
4. **Deploy and configure** (same as steps 4.2-4.3)

---

### PHASE 5: API Gateway Setup

#### Step 5.1: Create REST API

1. **Navigate to API Gateway**
2. Choose **"REST API"** → **"Build"**
3. **Create new API**
4. **Settings:**
   ```
   API name: CampusIssueAPI
   Description: API for campus issue reporting system
   Endpoint Type: Regional
   ```
5. Click **"Create API"**

#### Step 5.2: Create /report-issue Resource and Method

1. **Click on the root "/"** resource
2. **Actions** → **"Create Resource"**
3. **Resource settings:**
   ```
   Resource Name: report-issue
   Resource Path: /report-issue
   ☑ Enable API Gateway CORS
   ```
4. Click **"Create Resource"**

5. **Select the /report-issue resource**
6. **Actions** → **"Create Method"** → Select **"POST"**
7. **Method settings:**
   ```
   Integration type: Lambda Function
   ☑ Use Lambda Proxy integration
   Lambda Region: (your region)
   Lambda Function: CreateCampusIssue
   ```
8. Click **"Save"** → **"OK"** to give permission

#### Step 5.3: Create /issue-status/{issueId} Resource and Method

1. **Click on root "/"**
2. **Actions** → **"Create Resource"**
3. **Settings:**
   ```
   Resource Name: issue-status
   Resource Path: /issue-status
   ☑ Enable API Gateway CORS
   ```
4. Click **"Create Resource"**

5. **Select /issue-status**
6. **Actions** → **"Create Resource"**
7. **Settings:**
   ```
   Resource Name: issueId
   Resource Path: /{issueId}
   ☑ Enable API Gateway CORS
   ```
8. Click **"Create Resource"**

9. **Select /{issueId}**
10. **Actions** → **"Create Method"** → **"GET"**
11. **Method settings:**
    ```
    Integration type: Lambda Function
    ☑ Use Lambda Proxy integration
    Lambda Function: GetCampusIssue
    ```
12. **Save** → **"OK"**

#### Step 5.4: Create /update-status Resource and Method

1. **Click on root "/"**
2. **Actions** → **"Create Resource"**
3. **Settings:**
   ```
   Resource Name: update-status
   Resource Path: /update-status
   ☑ Enable API Gateway CORS
   ```
4. Click **"Create Resource"**

5. **Select /update-status**
6. **Actions** → **"Create Method"** → **"PUT"**
7. **Method settings:**
   ```
   Integration type: Lambda Function
   ☑ Use Lambda Proxy integration
   Lambda Function: UpdateCampusIssueStatus
   ```
8. **Save** → **"OK"**

#### Step 5.5: Enable CORS for All Methods

**For EACH method (POST, GET, PUT):**

1. **Select the method**
2. **Actions** → **"Enable CORS"**
3. **Leave default settings** and click **"Enable CORS and replace existing CORS headers"**
4. Click **"Yes, replace existing values"**

#### Step 5.6: Deploy API

1. **Actions** → **"Deploy API"**
2. **Deployment stage:** **"New Stage"**
3. **Stage name:** `prod`
4. Click **"Deploy"**
5. **🚨 IMPORTANT: Copy the Invoke URL** - you'll need this for the frontend!
   - Format: `https://xxxxxxxxxx.execute-api.region.amazonaws.com/prod`

---

### PHASE 6: Frontend S3 Deployment

#### Step 6.1: Create S3 Bucket

1. **Navigate to Amazon S3**
2. Click **"Create bucket"**
3. **Bucket settings:**

   ```
   Bucket name: campus-issue-system-[your-name]-[random-number]
   (e.g., campus-issue-system-john-12345)

   AWS Region: (same as other services)

   ❌ UNCHECK "Block all public access"
   ☑ I acknowledge that the current settings...
   ```

4. **Leave other settings default** and click **"Create bucket"**

#### Step 6.2: Configure S3 for Static Website Hosting

1. **Click on your bucket name**
2. Go to **"Properties"** tab
3. Scroll to **"Static website hosting"**
4. Click **"Edit"**
5. **Settings:**
   ```
   ☑ Enable
   Hosting type: Host a static website
   Index document: index.html
   Error document: index.html
   ```
6. Click **"Save changes"**

#### Step 6.3: Update Frontend with API URL

1. **Open** `frontend/index.html` in your text editor
2. **Find this line** (around line 155):
   ```javascript
   const API_BASE_URL =
     "https://YOUR_API_GATEWAY_ID.execute-api.YOUR_REGION.amazonaws.com/prod";
   ```
3. **Replace with your actual API Gateway Invoke URL** from Step 5.6:
   ```javascript
   const API_BASE_URL =
     "https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod";
   ```
4. **Save the file**

#### Step 6.4: Upload Frontend to S3

1. **In S3 bucket**, click **"Upload"**
2. **Add files** → Select your updated `index.html`
3. **Permissions:**
   - **Predefined ACLs:** Grant public-read access
4. Click **"Upload"**

#### Step 6.5: Set Bucket Policy for Public Access

1. Go to **"Permissions"** tab
2. Click **"Bucket policy"**
3. **Add this policy** (replace `YOUR-BUCKET-NAME`):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```

4. Click **"Save changes"**

#### Step 6.6: Get Website URL

1. Go to **"Properties"** tab
2. Scroll to **"Static website hosting"**
3. **Copy the Bucket website endpoint** URL
4. **🎉 This is your live website URL!**

---

### PHASE 7: Testing & Verification

#### Step 7.1: Test the Website

1. **Open your S3 website URL** in a browser
2. **Fill out the form** and submit an issue
3. **Check if:**
   - You get a success message with Issue ID
   - You receive an email notification
   - You can check the issue status using the Issue ID

#### Step 7.2: Verify DynamoDB

1. **Go to DynamoDB** → **CampusIssues** table
2. Click **"Explore table items"**
3. **Verify** that your test issue appears in the table

#### Step 7.3: Check CloudWatch Logs

1. **Go to CloudWatch** → **Log groups**
2. **Look for log groups** like:
   - `/aws/lambda/CreateCampusIssue`
   - `/aws/lambda/GetCampusIssue`
   - `/aws/lambda/UpdateCampusIssueStatus`
3. **Check logs** for any errors

---

## 🔧 Troubleshooting Guide

### Common Issues & Solutions

#### 1. "Access Denied" Error in Lambda

**Problem:** Lambda can't access DynamoDB or SES
**Solution:**

- Check if IAM role is correctly attached to Lambda functions
- Verify IAM policy has correct permissions
- Ensure table ARN in policy matches your actual table

#### 2. CORS Error in Browser

**Problem:** Browser blocks API calls
**Solution:**

- Ensure CORS is enabled for ALL API Gateway methods
- Re-deploy API after enabling CORS
- Check that Access-Control-Allow-Origin headers are present

#### 3. Email Not Sending

**Problem:** SES email notifications fail
**Solution:**

- Verify both sender and receiver email addresses in SES
- Check that emails in Lambda code match verified emails
- Ensure you're in the same region for all services

#### 4. 502/504 Gateway Errors

**Problem:** API Gateway can't connect to Lambda
**Solution:**

- Check Lambda function names match exactly in API Gateway
- Verify Lambda proxy integration is enabled
- Check Lambda function permissions

#### 5. Website Not Loading from S3

**Problem:** S3 website returns errors
**Solution:**

- Ensure bucket policy allows public read access
- Check that index.html is uploaded and public
- Verify static website hosting is enabled

---

## 💰 AWS Free Tier Limits

| Service         | Free Tier Limit               | Usage in Project            |
| --------------- | ----------------------------- | --------------------------- |
| **Lambda**      | 1M requests/month             | ✅ Very low usage           |
| **DynamoDB**    | 25GB storage, 25 RCU/WCU      | ✅ Minimal storage          |
| **API Gateway** | 1M API calls/month            | ✅ Low traffic expected     |
| **S3**          | 5GB storage, 20K GET requests | ✅ Single HTML file         |
| **SES**         | 62K emails/month              | ✅ Only notification emails |

**💡 Cost Estimate:** $0.00/month for typical usage

---

## 🎯 Next Steps & Enhancements

### Immediate Improvements

1. **Add user authentication** (AWS Cognito)
2. **Implement file upload** for issue attachments
3. **Add email templates** with better formatting
4. **Create admin dashboard** with all issues view

### Advanced Features

1. **Real-time notifications** (WebSocket API)
2. **Mobile app** using React Native + AWS Amplify
3. **Analytics dashboard** with issue trends
4. **Integration with campus systems**

---

## 📞 Support & Resources

### AWS Documentation

- [Lambda Developer Guide](https://docs.aws.amazon.com/lambda/)
- [API Gateway Developer Guide](https://docs.aws.amazon.com/apigateway/)
- [DynamoDB Developer Guide](https://docs.aws.amazon.com/dynamodb/)

### Useful Commands

```bash
# Check AWS CLI configuration
aws sts get-caller-identity

# List Lambda functions
aws lambda list-functions

# Check DynamoDB table
aws dynamodb describe-table --table-name CampusIssues
```

---

**🚀 Congratulations! Your Campus Issue Reporting System is now live on AWS!**

**Resume Bullet Point:**
_"Developed and deployed a serverless campus issue reporting system using AWS S3, API Gateway, Lambda, DynamoDB, and SES with secure IAM roles, REST APIs, and automated email notifications, fully optimized for AWS Free Tier with zero monthly costs."_
