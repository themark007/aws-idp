

# 📄 AWS Intelligent Document Processing (IDP) — CloudFormation Deployment Guide

> Deploy a fully serverless Intelligent Document Processing pipeline on AWS using CloudFormation, Textract, Lambda, and S3 — in under 10 minutes.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Deployment Steps](#deployment-steps)
  - [Step 0 — Create IAM Role with Admin Access](#step-0--create-iam-role-with-admin-access)
  - [Step 1 — Confirm Files in S3](#step-1--confirm-files-in-s3)
  - [Step 2 — Confirm Bucket Policy](#step-2--confirm-bucket-policy)
  - [Step 3 — Launch the CloudFormation Stack](#step-3--launch-the-cloudformation-stack)
  - [Step 4 — Fill Stack Parameters](#step-4--fill-stack-parameters)
  - [Step 5 — Configure Stack Options](#step-5--configure-stack-options)
  - [Step 6 — Acknowledge IAM Resources](#step-6--acknowledge-iam-resources)
  - [Step 7 — Monitor Deployment](#step-7--monitor-deployment)
- [Post-Deployment](#post-deployment)
- [Known Issues & Troubleshooting](#known-issues--troubleshooting)
- [CloudWatch Monitoring](#cloudwatch-monitoring)
- [License](#license)

---

## Overview

This repository contains the CloudFormation templates and instructions to deploy an **Intelligent Document Processing (IDP)** pipeline on AWS. The stack provisions:

- **OCR processing** via AWS Textract
- **Document classification** via Lambda
- **Rule validation** via Lambda
- **Web UI** for uploading and managing documents
- **Optional Knowledge Base** integration

**S3 templates required:**
```
pandey-idp (or pandey-idp-custom)
├── pattern2.template
└── idp-main.yaml
```

---

## Architecture

```
User (Web UI)
    │
    ▼
S3 (Document Upload)
    │
    ▼
Lambda: ClassificationFunction
    │
    ▼
AWS Textract (OCR)
    │
    ▼
Lambda: RuleValidationFunction
    │
    ▼
Lambda: OCRFunction
    │
    ▼
Output / Knowledge Base (Optional)
```

---

## Prerequisites

Before you begin, make sure you have:

- [ ] An active AWS account with **billing verification complete**
- [ ] AWS Textract enabled in your region *(see [Known Issues](#known-issues--troubleshooting))*
- [ ] An IAM role with `AdministratorAccess` for CloudFormation *(created in [Step 0](#step-0--create-iam-role-with-admin-access))*
- [ ] CloudFormation template files uploaded to your S3 bucket:
  - `pattern2.template`
  - `idp-main.yaml`
- [ ] IAM permissions to create stacks, Lambda functions, and IAM roles
- [ ] S3 bucket in region `us-west-2` (or update the URL accordingly)

---

## Deployment Steps

### Step 0 — Create IAM Role with Admin Access

> ⚠️ **Do this first.** CloudFormation needs to assume an IAM role with sufficient permissions to create all stack resources (Lambda, S3, Textract, ECR, Cognito, etc.). Using a role with `AdministratorAccess` ensures no permission-related failures during deployment.

#### 0a — Create the IAM Role

1. Go to **IAM → Roles → Create role**
2. Under **Trusted entity type**, select **AWS service**
3. Under **Use case**, select **CloudFormation**
4. Click **Next**

#### 0b — Attach AdministratorAccess Policy

1. In the **Add permissions** search box, type `AdministratorAccess`
2. Check the box next to **AdministratorAccess** (AWS managed policy)
3. Click **Next**

#### 0c — Name and Create the Role

Fill in the details:

| Field | Value |
|-------|-------|
| **Role name** | `IDPCloudFormationAdminRole` |
| **Description** | `Admin role for IDP CloudFormation stack deployment` |

Click **Create role**.

#### 0d — Copy the Role ARN

1. Go to **IAM → Roles → IDPCloudFormationAdminRole**
2. Copy the **ARN** — it looks like:

```
arn:aws:iam::123456789012:role/IDPCloudFormationAdminRole
```

> 💡 You will paste this ARN into **CloudFormation → Permissions** in Step 5 (Stack Options).

#### Role Trust Policy (Auto-created)

When you selected CloudFormation as the trusted entity, AWS automatically attached this trust policy to the role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudformation.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

> ✅ This allows CloudFormation to assume the role and act on your behalf during stack creation.

---

### Step 1 — Confirm Files in S3

1. Navigate to **S3** in the AWS Console
2. Open your bucket: `pandey-idp` or `pandey-idp-custom`
3. Verify both files are present:

| File | Status |
|------|--------|
| `pattern2.template` | ✅ Required |
| `idp-main.yaml` | ✅ Required |

> If either file is missing, upload it before proceeding.

---

### Step 2 — Confirm Bucket Policy

This step is **critical**. CloudFormation must be able to read the templates from S3.

1. Go to **S3 → Your Bucket → Permissions → Bucket Policy**
2. Ensure the following policy exists (replace `YOUR_BUCKET_NAME` with your actual bucket name):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFormationRead",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudformation.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
    }
  ]
}
```

3. Click **Save changes**

> ⚠️ Without this policy, CloudFormation will fail with an `Access Denied` error when reading nested templates.

---

### Step 3 — Launch the CloudFormation Stack

1. Navigate to **CloudFormation → Create Stack → With new resources (standard)**
2. Under **Specify template**, select **Amazon S3 URL**
3. Paste the following URL (replace `YOUR_BUCKET_NAME`):

```
https://YOUR_BUCKET_NAME.s3.us-west-2.amazonaws.com/idp-main.yaml
```

4. Click **Next**

---

### Step 4 — Fill Stack Parameters

Fill in the parameters as follows:

| Parameter | Value |
|-----------|-------|
| **Stack Name** | `IDP` |
| **Admin Email** | `your@email.com` |
| **Allowed Domain** | `gmail.com` (or your domain) |
| **Document Processing Pattern** | `Pattern2` |
| **EnableECRImageScanning** | `false` |
| **Maximum Concurrent Workflows** | `10` |
| **Knowledge Base** | `NONE` *(safer if Textract is not yet activated)* |

> 💡 **Tip:** If your AWS account is newly created and Textract isn't fully activated yet, set **Knowledge Base → NONE** to avoid activation-related failures.

Click **Next**.

---

### Step 5 — Configure Stack Options

1. Under **Permissions**, paste the IAM role ARN you copied in **Step 0d**:

```
arn:aws:iam::123456789012:role/IDPCloudFormationAdminRole
```

2. Leave all other options as their **defaults**

Click **Next**.

---

### Step 6 — Acknowledge IAM Resources

On the final review page, scroll to the bottom and check:

- [ ] **I acknowledge that AWS CloudFormation might create IAM resources**

Click **Create stack**.

---

### Step 7 — Monitor Deployment

1. Go to **CloudFormation → Stacks → IDP → Events tab**
2. Watch for these key Lambda resources to reach `CREATE_COMPLETE`:

| Resource | Expected Status |
|----------|----------------|
| `RuleValidationFunction` | ✅ `CREATE_COMPLETE` |
| `OCRFunction` | ✅ `CREATE_COMPLETE` |
| `ClassificationFunction` | ✅ `CREATE_COMPLETE` |

> 🎉 If all three show `CREATE_COMPLETE`, the memory/runtime issues from previous deployments are resolved.

The full stack typically completes within **5–15 minutes**.

---

## Post-Deployment

Once the stack status shows **`CREATE_COMPLETE`**:

1. Navigate to **CloudFormation → Stacks → IDP → Outputs tab**
2. Copy the **WebUI URL**
3. Open it in your browser
4. Log in with the **Admin Email** you provided
5. Upload a PDF document to test the pipeline
6. Monitor processing in **CloudWatch Logs** if needed (see below)

---

## Known Issues & Troubleshooting

### SubscriptionRequiredException (Textract)

**Error:**
```
SubscriptionRequiredException: Your AWS account is not subscribed to this service.
```

**Cause:** Your AWS account is not yet fully activated for Amazon Textract.

**Fix:**
- This resolves automatically — usually within **24 hours** of account creation or billing verification completion.
- You do **not** need to redeploy the stack. Once Textract activates, document processing will work automatically.
- Until then, uploading PDFs will result in Textract errors — this is expected.

**Checklist while waiting:**
- [ ] Billing verification complete?
- [ ] Account older than 24 hours?
- [ ] Textract available in `us-west-2`? Check the [AWS Regional Services List](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/).

---


## CloudWatch Monitoring

To monitor document processing after deployment:

1. Go to **CloudWatch → Log Groups**
2. Look for log groups named after your Lambda functions:
   - `/aws/lambda/IDP-OCRFunction-*`
   - `/aws/lambda/IDP-ClassificationFunction-*`
   - `/aws/lambda/IDP-RuleValidationFunction-*`
3. Click into a log group and select the latest **Log Stream**
4. Filter logs using: `ERROR` or `Textract` to quickly find issues

---

## Repository Structure

```
.
├── idp-main.yaml           # Main CloudFormation template (entry point)
├── pattern2.template       # Nested stack template for Pattern 2 processing
└── README.md               # This file
```

---
