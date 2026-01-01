# Serverless Feedback Form Application

A **Serverless Feedback Form** application built entirely using **AWS serverless services**, allowing users to submit feedback along with optional file uploads. This project automates storage, notifications, and frontend deployment without the need to manage any servers.

---

## Table of Contents
- [Project Overview](#project-overview)
- [Key Features](#key-features)
- [Architecture Overview](#architecture-overview)
- [Error Handling & Lessons Learned](#error-handling--lessons-learned)
- [Technologies Used](#technologies-used)
- [Setup Instructions](#setup-instructions)
- [CI/CD Pipeline](#cicd-pipeline)
- [License](#license)

---

## Project Overview
This project is a **Serverless Feedback Form** where users can submit their **name, email, message**, and optionally a **file or PDF**. Upon submission:

- Feedback is stored in **DynamoDB**.
- Files are uploaded to **S3**.
- Admin receives **email notifications via SES**.
- The frontend is automatically deployed and served via **S3 + CloudFront** using **GitHub Actions**.

---

## Key Features

### Feedback Submission
- Users can submit feedback including **name, email, message**, and optional **file/PDF**.

### Automatic File Storage
- Uploaded files are stored securely in an **Amazon S3 bucket**.

### Feedback Storage
- Feedback details (name, email, message, file URL, timestamp) are stored in **DynamoDB**.

### Email Notifications
- Admin receives instant notifications through **Amazon SES** when new feedback is submitted.

### Serverless Architecture
- Backend is fully handled by **AWS Lambda**.
- API endpoints exposed via **API Gateway**.

### CI/CD Automation
- Frontend deployment is automated using **GitHub Actions**, syncing code to **S3** and invalidating **CloudFront** cache.

---

## Architecture Overview

The project uses a **fully serverless architecture**:

**Frontend**
- Static web hosted on **S3**.
- Served via **CloudFront** for global distribution and SSL.

**Backend**
- **AWS Lambda** handles all application logic.
- **API Gateway** exposes HTTP endpoints.
- **DynamoDB** stores feedback records.
- **S3** stores uploaded files.
- **SES** sends email notifications to the admin.

**CI/CD**
- GitHub Actions automatically deploys frontend changes to **S3** and invalidates **CloudFront** cache.

---

## Error Handling & Lessons Learned

**Issue:**  
Lambda could not write to DynamoDB → `ResourceNotFoundException`

**Cause:**  
- DynamoDB table region mismatch  
- Missing environment variables  
- Insufficient IAM permissions

**Solution:**  
1. Ensure DynamoDB table exists in the same region as Lambda.  
2. Set correct environment variables (`TABLE_NAME`, `REGION`).  
3. Attach proper IAM permissions (`dynamodb:PutItem`) to Lambda execution role.

---

## Technologies Used

- **AWS Services:** Lambda, S3, DynamoDB, SES, API Gateway, CloudFront, IAM  
- **CI/CD:** GitHub Actions  
- **Frontend:** HTML, CSS, JavaScript (static files)

---

## Setup Instructions

1. **AWS Configuration**
   - Create S3 buckets for file storage and frontend hosting.
   - Create a DynamoDB table (`Feedback-list`) for storing feedback.
   - Create and configure a Lambda function.
   - Configure Amazon SES for admin email notifications.
   - Create a REST API using API Gateway.
   - Attach proper IAM roles and permissions.

2. **Frontend Deployment**
   - Upload static frontend files to S3.
   - Enable static website hosting.
   - Serve via CloudFront.

3. **Environment Variables**
   - Set Lambda environment variables:
     ```text
     TABLE_NAME=Feedback-list
     BUCKET_NAME=<your-s3-bucket>
     ADMIN_EMAIL=<your-email>
     REGION=<aws-region>
     ```

---

## CI/CD Pipeline

- GitHub Actions workflow automatically deploys frontend updates to S3 and invalidates the CloudFront cache.
- Workflow example:
```yaml
name: Deploy Frontend to S3 + CloudFront

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Sync frontend to S3
        run: aws s3 sync ./frontend s3://${{ secrets.S3_BUCKET }} --delete

      - name: Invalidate CloudFront cache
        if: env.CLOUDFRONT_DIST_ID != ''
        env:
          CLOUDFRONT_DIST_ID: ${{ secrets.CLOUDFRONT_DIST_ID }}
        run: |
          aws cloudfront create-invalidation \
            --distribution-id $CLOUDFRONT_DIST_ID \
            --paths "/*"
