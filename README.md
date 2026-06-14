# ☁️ Cloud Resume Challenge

<div align="center">

[![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com/)
[![Terraform](https://img.shields.io/badge/Terraform-623CE4?style=for-the-badge&logo=terraform&logoColor=white)](https://terraform.io/)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org/)
[![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)](https://github.com/features/actions)

**A serverless resume website built from scratch on AWS Free Tier — with real production debugging, infrastructure as code, and automated CI/CD.**

[🌐 Live Demo](https://YOUR_CLOUDFRONT_URL) • [📸 Screenshots](#-project-gallery) • [🏗️ Architecture](#-architecture) • [🚀 Quick Deploy](#-quick-deploy)

</div>

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Live Demo](#-live-demo)
- [Architecture](#-architecture)
- [What I Built](#-what-i-built)
- [The Real Challenges (And How I Solved Them)](#-the-real-challenges-and-how-i-solved-them)
- [Tech Stack](#-tech-stack)
- [Project Gallery](#-project-gallery)
- [Quick Deploy](#-quick-deploy)
- [What I Learned](#-what-i-learned)
- [Connect With Me](#-connect-with-me)

---

## 🎯 Overview

This isn't just a static resume page. It's a **full-stack serverless application** deployed on AWS that demonstrates:

- ✅ **Static website hosting** with global CDN distribution
- ✅ **Serverless API** with atomic database operations
- ✅ **Infrastructure as Code** for reproducible deployments
- ✅ **CI/CD pipeline** for automated updates
- ✅ **Production debugging** — because real cloud work isn't copy-paste

**Every visitor increments a live counter.** Refresh the page and watch it grow. That single number represents a journey through S3, CloudFront, API Gateway, Lambda, and DynamoDB — all working in harmony.

---

## 🌐 Live Demo

> **⚡ Currently Live:** [https://YOUR_CLOUDFRONT_URL](https://YOUR_CLOUDFRONT_URL)
>
> *Built entirely on AWS Free Tier. Zero monthly cost.*

```
┌─────────────────────────────────────────┐
│  👀 This resume has been viewed         │
│     1,247 times                         │
│                                         │
│  [Refresh the page — the count grows]   │
└─────────────────────────────────────────┘
```

---

## 🏗️ Architecture

```
<img width="796" height="482" alt="cloud resume challenge architecture diagram" src="https://github.com/user-attachments/assets/36fa3a6e-871d-46ad-bdd7-fe06f09ed724" />


**Data Flow:**
1. Visitor loads resume from **CloudFront edge location** (low latency)
2. JavaScript calls `GET /prod/count` via **API Gateway**
3. **Lambda** executes Python code with **Boto3**
4. **DynamoDB** atomically increments `visitor_count` (no race conditions)
5. New count returns to browser and displays in real-time

---

## 🛠️ What I Built

### Frontend
- **Single-file architecture** — HTML, CSS, and JavaScript embedded in one `index.html`
- **Dark theme** with CSS variables, hover animations, and responsive grid layout
- **Font Awesome icons** for visual polish
- **Fetch API** calling the backend asynchronously
- **Mobile-responsive** design with media queries

### Backend
- **AWS Lambda** (Python 3.11) — serverless compute, zero idle cost
- **DynamoDB** (On-Demand) — NoSQL database with atomic increment operations
- **API Gateway** (REST) — managed API with CORS configuration
- **JSON serialization** with custom Decimal encoder for DynamoDB compatibility

### Infrastructure
- **Amazon S3** — static website hosting with public read policy
- **CloudFront** — global CDN with HTTPS and origin access identity
- **Route 53** — custom domain with ACM SSL certificate *(optional)*
- **IAM** — least-privilege roles and policies

### Automation
- **Terraform** — entire infrastructure defined as code
- **GitHub Actions** — CI/CD pipeline for automated deployments
- **S3 backend** — remote state management for team collaboration

---

## 🔥 The Real Challenges (And How I Solved Them)

> *"Tell me about a time you debugged something in production."* — Every cloud interview ever

This section is why this README stands out. These aren't tutorial steps. These are **real problems I hit and solved**.

### Challenge 1: "Missing Authentication Token" (API Gateway)

**Symptom:** API returned `{"message":"Missing Authentication Token"}`

**Root Cause:** API Gateway resource path mismatch. The deployed stage was `prod` but the resource path wasn't correctly mapped to `/count`.

**Fix:**
- Verified API Gateway resource tree: `/` → `/count` → `GET`
- Confirmed stage deployment: `prod` with full path `/prod/count`
- **Key insight:** The invoke URL must include the stage name AND the resource path:
  ```
  https://{api-id}.execute-api.us-east-1.amazonaws.com/prod/count
  ```
- Redeployed API after resource changes

**Lesson:** API Gateway paths are explicit. Every segment matters.

---

### Challenge 2: "504 Gateway Timeout" (CloudFront → S3)

**Symptom:** CloudFront returned `504 Gateway Timeout ERROR`

**Root Cause:** CloudFront origin domain was configured as the S3 **bucket endpoint** (`s3.amazonaws.com`) instead of the S3 **website endpoint** (`s3-website-us-east-1.amazonaws.com`). The bucket endpoint requires signed requests; the website endpoint serves static content directly.

**Fix:**
- Deleted the broken CloudFront distribution
- Created new distribution with **manual origin domain entry**:
  ```
  stanley-resume-bucket.s3-website-us-east-1.amazonaws.com
  ```
  *(typed manually — NOT selected from dropdown)*
- Set **Origin Protocol Policy** to `HTTP only` (S3 website doesn't support HTTPS)
- Set **Viewer Protocol Policy** to `Redirect HTTP to HTTPS`
- Configured **Default Root Object** as `index.html`

**Lesson:** S3 has two endpoints: REST API (bucket) and static website. CloudFront for static sites needs the website endpoint.

---

### Challenge 3: CORS Errors (Browser Security)

**Symptom:** Browser console showed `Access-Control-Allow-Origin` errors. JavaScript fetch blocked.

**Root Cause:** API Gateway CORS headers not configured. Lambda returned CORS headers, but API Gateway method response didn't map them.

**Fix:**
- Enabled CORS on `/count` resource in API Gateway:
  - `Access-Control-Allow-Origin: *`
  - `Access-Control-Allow-Headers: Content-Type`
  - `Access-Control-Allow-Methods: GET`
- Added header mappings in **Method Response** and **Integration Response**
- Redeployed API stage after CORS changes

**Lesson:** CORS is a contract between browser and server. Both sides must agree.

---

### Challenge 4: Lambda Permission Denied (DynamoDB)

**Symptom:** Lambda returned 500 error. CloudWatch Logs showed `AccessDeniedException`.

**Root Cause:** IAM role attached to Lambda didn't have DynamoDB write permissions.

**Fix:**
- Attached `AmazonDynamoDBFullAccess` policy to Lambda execution role
- Verified trust policy allowed `lambda.amazonaws.com` to assume the role
- Retested Lambda in console — count incremented successfully

**Lesson:** IAM is the gatekeeper. Every AWS service needs explicit permission to touch another.

---

## 💻 Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Frontend** | HTML5, CSS3, JavaScript (Vanilla) | Resume UI, animations, fetch API |
| **CDN** | CloudFront | Global content delivery, HTTPS termination |
| **Storage** | S3 | Static website hosting, object storage |
| **API** | API Gateway (REST) | Managed API endpoint, CORS handling |
| **Compute** | Lambda (Python 3.11) | Serverless backend logic |
| **Database** | DynamoDB (On-Demand) | NoSQL counter storage, atomic operations |
| **IaC** | Terraform | Infrastructure as code, state management |
| **CI/CD** | GitHub Actions | Automated deployment pipeline |
| **Security** | IAM, ACM | Least-privilege access, SSL certificates |

---

## 📸 Project Gallery

### Architecture Diagram
<img width="796" height="482" alt="cloud resume challenge architecture diagram" src="https://github.com/user-attachments/assets/8f38181a-9a01-412f-8020-82042fea689a" />

### Live Resume (Dark Theme)
<img width="1139" height="657" alt="my resume is live for everyone to view" src="https://github.com/user-attachments/assets/27debfd5-08c9-4519-a89f-5baf4a518e7b" />


### Visitor Counter in Action
<img width="1366" height="720" alt="FULL STACK CLOUD RESUME IS LIVE S3 frontend+API Gateway+Lambda +Dynamo DB 3" src="https://github.com/user-attachments/assets/eb4ba39b-f59b-4311-969b-e065b1fb71b4" />


---

## 🚀 Quick Deploy

Want to see this running in your own AWS account? Five minutes, zero cost.

### Prerequisites
- AWS account (Free Tier eligible)
- AWS CLI configured (`aws configure`)
- Terraform installed (`terraform init`)

### Deploy

```bash
# Clone the repo
git clone https://github.com/stanleyjnrkanzara-wq/Cloud-Resume-Challenge.git
cd Cloud-Resume-Challenge

# Initialize Terraform
cd terraform
terraform init

# Review the plan
terraform plan

# Deploy everything
terraform apply

# Upload frontend to S3
aws s3 cp ../index.html s3://$(terraform output -raw bucket_name)/index.html

# Get your CloudFront URL
echo "Live at: $(terraform output -raw cloudfront_url)"
```

### Destroy (Clean Up)

```bash
terraform destroy
```

*All resources removed. Zero ongoing cost.*

---

## 🎓 What I Learned

### Technical Skills
- **AWS Core Services:** S3, CloudFront, API Gateway, Lambda, DynamoDB, IAM, CloudWatch
- **Serverless Patterns:** Event-driven architecture, stateless compute, managed databases
- **Infrastructure as Code:** Terraform modules, remote state, variable management
- **CI/CD Pipelines:** GitHub Actions workflows, secrets management, automated testing
- **API Design:** RESTful endpoints, CORS configuration, proxy integration, stage management
- **Security:** IAM policies, least privilege, origin access identity, SSL/TLS

### Debugging Skills
- **Systematic troubleshooting:** Isolating layers (DNS → CDN → API → Compute → Database)
- **Log analysis:** CloudWatch Logs for Lambda, API Gateway execution logs
- **Network debugging:** Browser DevTools, curl, API Gateway test console
- **Configuration management:** Understanding AWS service interactions and dependencies

### Cloud Mindset
- **Cost optimization:** Free Tier awareness, on-demand billing, resource cleanup
- **Documentation:** README as living document, architecture diagrams, runbooks
- **Automation:** If you do it twice, script it. If you do it three times, automate it.

---

## 📊 Cost Breakdown

| Service | Usage | Monthly Cost |
|---------|-------|-------------|
| S3 | Static hosting, < 1MB storage | **$0.00** |
| CloudFront | < 10GB transfer | **$0.00** (Free Tier: 1TB) |
| API Gateway | < 1M requests | **$0.00** (Free Tier: 1M) |
| Lambda | < 1M invocations | **$0.00** (Free Tier: 1M) |
| DynamoDB | On-Demand, < 25GB | **$0.00** (Free Tier: 25GB) |
| CloudWatch | Basic logs | **$0.00** |
| **TOTAL** | | **$0.00/month** |

---

## 🔗 Connect With Me

I'm actively seeking cloud engineering and DevOps opportunities. Let's talk!

<div align="center">

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/stanley-jnr-kanzara-0081133a8?utm_source=share_via&utm_content=profile&utm_medium=member_android)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/stanleyjnrkanzara-wq)
[![Email](https://img.shields.io/badge/Email-EA4335?style=for-the-badge&logo=gmail&logoColor=white)](mailto:stanleyjnrkanzara@gmail.com)

</div>

---

## 🏆 Challenge Origin

This project follows the [Cloud Resume Challenge](https://cloudresumechallenge.dev/) by [Forrest Brazeal](https://twitter.com/forrestbrazeal) — a hands-on certification for cloud engineers that proves you can build, not just study.

> *"The Cloud Resume Challenge is a multi-step resume project which helps build and demonstrate skills fundamental to pursuing a career in Cloud. The project was created by Forrest Brazeal, formerly a Cloud Architect at InSciCo, now at Google Cloud."*

---

<div align="center">

**Built with ❤️ and AWS Free Tier**

⭐ Star this repo if you found it helpful!

</div>
