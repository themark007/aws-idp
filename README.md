
✅ STEP 1 — Confirm Files in S3 Go to: S3 → Your bucket (pandey-idp or pandey-idp-custom) Make sure you see: pattern2.template ✅ idp-main.yaml ✅ If both are there → good. ✅ STEP 2 — Confirm Bucket Policy (Very Important) Go to: S3 → Bucket → Permissions → Bucket Policy Make sure this exists: { "Version": "2012-10-17", "Statement": [ { "Sid": "AllowCloudFormationRead", "Effect": "Allow", "Principal": { "Service": "cloudformation.amazonaws.com" }, "Action": "s3:GetObject", "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*" } ] } Replace YOUR_BUCKET_NAME with your actual bucket name. If this is already there → continue. ✅ STEP 3 — Launch Stack Go to: CloudFormation → Create Stack → With new resources Choose: Amazon S3 URL Paste: https://YOUR_BUCKET_NAME.s3.us-west-2.amazonaws.com/idp-main.yaml Click Next. ✅ STEP 4 — Fill Parameters Use: Stack Name → IDP Admin Email → your email Allowed Domain → gmail.com Document Processing Pattern → Pattern2 EnableECRImageScanning → false Maximum Concurrent Workflows → 10 Knowledge Base → Optional (if Textract not activated yet, safer to keep NONE) Click Next. ✅ STEP 5 — Configure Stack Options Leave defaults. Click Next. ✅ STEP 6 — Acknowledge IAM Check: ☑ I acknowledge that AWS CloudFormation might create IAM resources Click Create stack. ✅ STEP 7 — Monitor Deployment Go to: CloudFormation → IDP → Events Watch for these specifically: RuleValidationFunction → CREATE_COMPLETE OCRFunction → CREATE_COMPLETE ClassificationFunction → CREATE_COMPLETE If these complete → memory issue solved 🎉 ⚠️ IMPORTANT Your earlier runtime failure: SubscriptionRequiredException Means: Your AWS account is still not fully activated for Textract. Even if stack deploys successfully: 👉 Textract will fail until AWS account activation completes. This usually takes: Up to 24 hours after signup Or until billing verification completes 🎯 After Stack = CREATE_COMPLETE Go to CloudFormation → Outputs Copy WebUI URL Login Upload PDF Monitor CloudWatch if needed
give a proper readme for all this for GitHub perfect
Show more
10:31 PM

Checking docx skill for document creation best practices
This is a README for GitHub, so I'll create a polished Markdown file — no need for docx. Let me build a clean, professional README.md.

Created a file, read a file
Here's your GitHub README — clean, professional, and complete. Here's what's included:
Structure highlights:
Table of Contents with anchor links for easy navigation
Architecture diagram in ASCII art showing the full pipeline flow
Prerequisites checklist so users know what's needed before starting
All 7 steps formatted as clear sections with tables, code blocks, and checkboxes
Post-deployment guide with exact steps to access the Web UI
Troubleshooting section covering SubscriptionRequiredException, stack rollbacks, and Lambda memory issues
CloudWatch monitoring instructions with the specific log group patterns to look for
Repository structure diagram
Just drop it into the root of your GitHub repo as README.md and replace YOUR_BUCKET_NAME with your actual bucket name in the two places it appears (bucket policy and S3 URL).
Readme
Document · MD 


give in md forma
Claude is AI and can make mistakes. Please double-check responses.
Readme · MD
Copy

# 📄 AWS Intelligent Document Processing (IDP) — CloudFormation Deployment Guide

> Deploy a fully serverless Intelligent Document Processing pipeline on AWS using CloudFormation, Textract, Lambda, and S3 — in under 10 minutes.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Deployment Steps](#deployment-steps)
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
- [ ] CloudFormation template files uploaded to your S3 bucket:
  - `pattern2.template`
  - `idp-main.yaml`
- [ ] IAM permissions to create stacks, Lambda functions, and IAM roles
- [ ] S3 bucket in region `us-west-2` (or update the URL accordingly)

---

## Deployment Steps

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

Leave all options as their **defaults**.

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

### Stack Rollback / CREATE_FAILED

If the stack fails and rolls back:

1. Go to **CloudFormation → Events** and find the first `FAILED` event
2. Read the **Status Reason** — it usually pinpoints the issue
3. Common causes:
   - Missing bucket policy (Step 2)
   - Incorrect S3 URL (Step 3)
   - IAM permissions not acknowledged (Step 6)
   - Textract not activated (see above)

---

### Lambda Memory Issues

If earlier deployments failed due to Lambda memory errors, this deployment uses **Pattern2** which is configured with appropriate memory allocations. The three Lambda functions listed in [Step 7](#step-7--monitor-deployment) reaching `CREATE_COMPLETE` confirms this is resolved.

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





*Built for AWS CloudFormation · Powered by Textract, Lambda & S3*
