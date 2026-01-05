# Campus Issue Reporting System - Quick Deployment Checklist

## ✅ Pre-Deployment Checklist

### AWS Account Setup
- [ ] AWS Free Tier account created
- [ ] Region selected (recommend `us-east-1`)
- [ ] AWS Console access verified

---

## 🚀 Deployment Steps (Follow in Order)

### Phase 1: DynamoDB Setup
- [ ] **Create table:** `CampusIssues`
- [ ] **Partition key:** `issueId` (String)  
- [ ] **Billing mode:** On-demand
- [ ] **Table status:** Active ✅
- [ ] **Copy table ARN** for later use

### Phase 2: Email Setup (SES)
- [ ] **Navigate to SES** in same region
- [ ] **Verify sender email** address
- [ ] **Verify receiver email** address (can be same)
- [ ] **Check email** and click verification links
- [ ] **Test email sending** (optional)

### Phase 3: IAM Role Creation
- [ ] **Create role:** `CampusIssueLambdaRole`
- [ ] **Attach policy:** `AWSLambdaBasicExecutionRole`
- [ ] **Add custom inline policy** from `iam/lambda_policy.json`
- [ ] **Policy name:** `CampusIssueDynamoSESPolicy`

### Phase 4: Lambda Functions (Create all 3)

#### Function 1: CreateCampusIssue
- [ ] **Runtime:** Python 3.11
- [ ] **Role:** CampusIssueLambdaRole
- [ ] **Code:** Copy from `lambdas/create_issue.py`
- [ ] **Update email addresses** in code
- [ ] **Deploy** function
- [ ] **Timeout:** 30 seconds
- [ ] **Memory:** 256 MB

#### Function 2: GetCampusIssue  
- [ ] **Runtime:** Python 3.11
- [ ] **Role:** CampusIssueLambdaRole
- [ ] **Code:** Copy from `lambdas/get_issue.py`
- [ ] **Deploy** function
- [ ] **Configure** timeout and memory

#### Function 3: UpdateCampusIssueStatus
- [ ] **Runtime:** Python 3.11
- [ ] **Role:** CampusIssueLambdaRole
- [ ] **Code:** Copy from `lambdas/update_status.py`
- [ ] **Update email addresses** in code
- [ ] **Deploy** function
- [ ] **Configure** timeout and memory

### Phase 5: API Gateway Setup

#### Create REST API
- [ ] **API name:** `CampusIssueAPI`
- [ ] **Endpoint type:** Regional

#### Create Resources & Methods
- [ ] **Resource:** `/report-issue` → **Method:** POST → **Lambda:** CreateCampusIssue
- [ ] **Resource:** `/issue-status/{issueId}` → **Method:** GET → **Lambda:** GetCampusIssue
- [ ] **Resource:** `/update-status` → **Method:** PUT → **Lambda:** UpdateCampusIssueStatus

#### Configure Methods
- [ ] **Enable Lambda Proxy Integration** for all methods
- [ ] **Enable CORS** for all methods
- [ ] **Deploy API** to `prod` stage
- [ ] **Copy Invoke URL** 📝 (IMPORTANT!)

### Phase 6: Frontend Deployment

#### S3 Bucket Setup
- [ ] **Create bucket:** `campus-issue-system-[yourname]-[number]`
- [ ] **Disable** "Block all public access"
- [ ] **Enable static website hosting**
- [ ] **Index document:** `index.html`

#### Update & Upload Frontend
- [ ] **Edit** `frontend/index.html`
- [ ] **Replace API_BASE_URL** with your API Gateway Invoke URL
- [ ] **Upload** `index.html` to S3 bucket
- [ ] **Set file permissions:** Grant public-read access
- [ ] **Add bucket policy** for public read access

#### Get Website URL
- [ ] **Copy S3 website endpoint URL** 📝
- [ ] **Test website** in browser

---

## 🧪 Testing Checklist

### Functional Testing
- [ ] **Open website URL** in browser
- [ ] **Submit test issue** via form
- [ ] **Verify success message** and Issue ID received
- [ ] **Check email** for notification
- [ ] **Test issue status lookup** using Issue ID
- [ ] **Test admin status update** functionality

### Backend Verification  
- [ ] **Check DynamoDB table** for test data
- [ ] **Review CloudWatch logs** for any errors
- [ ] **Verify SES emails** are being sent

### Error Handling
- [ ] **Test invalid Issue ID** lookup
- [ ] **Test form validation**
- [ ] **Check CORS** functionality

---

## 🚨 Common Issues & Quick Fixes

### Lambda Errors
- **Problem:** Access denied to DynamoDB/SES
- **Fix:** Check IAM role attached and policies

### CORS Errors
- **Problem:** Browser blocks API calls
- **Fix:** Enable CORS for ALL methods, re-deploy API

### Email Not Working
- **Problem:** SES emails not sending
- **Fix:** Verify email addresses in SES console

### API Gateway 502 Error
- **Problem:** Can't connect to Lambda
- **Fix:** Check Lambda function names in API Gateway

### Website Not Loading
- **Problem:** S3 website returns errors
- **Fix:** Check bucket policy and static hosting settings

---

## 📋 Final Verification

### Deployment Complete ✅
- [ ] **Website loads** correctly
- [ ] **Issue submission** works
- [ ] **Email notifications** received
- [ ] **Status checking** works
- [ ] **Admin updates** work
- [ ] **No console errors** in browser
- [ ] **CloudWatch logs** show successful executions

### Documentation Ready ✅
- [ ] **API Gateway Invoke URL** documented
- [ ] **S3 Website URL** documented
- [ ] **Email addresses** configured
- [ ] **Test Issue ID** for demo

---

## 🎯 Success Metrics

| Component | Status | URL/ID |
|-----------|--------|---------|
| **DynamoDB Table** | ✅ Active | CampusIssues |
| **Lambda Functions** | ✅ Deployed | 3 functions |
| **API Gateway** | ✅ Live | `https://xxxxx.execute-api.region.amazonaws.com/prod` |
| **S3 Website** | ✅ Public | `http://bucket-name.s3-website-region.amazonaws.com` |
| **SES Emails** | ✅ Verified | Sender & receiver emails |

---

**🎉 Congratulations! Your Campus Issue Reporting System is now live!**

**Demo Instructions:**
1. Share the S3 website URL
2. Submit a test issue
3. Show the email notification received
4. Demonstrate status checking with Issue ID
5. Show admin status update functionality

**Resume Bullet:**
*"Built and deployed a complete serverless campus issue reporting system on AWS using Lambda, API Gateway, DynamoDB, S3, and SES with automated email notifications, achieving zero monthly costs within Free Tier limits."*