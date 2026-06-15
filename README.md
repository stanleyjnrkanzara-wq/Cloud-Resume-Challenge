# ☁️ Cloud Resume Challenge

<div align="center">

[![AWS](https://img.shields.io/badge/AWS-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)](https://aws.amazon.com/)
[![Terraform](https://img.shields.io/badge/Terraform-623CE4?style=for-the-badge&logo=terraform&logoColor=white)](https://terraform.io/)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org/)
[![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)](https://github.com/features/actions)

**A serverless resume website built on AWS Free Tier with a live visitor counter.**

[🌐 Live Demo](https://d1x27lc5lrvrkm.cloudfront.net) • [💻 Source Code](https://github.com/stanleyjnrkanzara-wq/Cloud-Resume-Challenge)

</div>

---

## 🎯 What I Built

A full-stack serverless resume that tracks every visitor. **Refresh the page and watch the counter grow.**

```
Browser → CloudFront (CDN + HTTPS) → S3 (static site)
              ↓
         JavaScript calls API
              ↓
    API Gateway → Lambda (Python) → DynamoDB (counter)
```

**Every layer is real, every service is managed, and the bill is $0.00.**

---

## 🏗️ Architecture

```mermaid
graph TB
    User["👤 Visitor<br/>Your Browser"]
    CF["⚡ CloudFront<br/>Global CDN + HTTPS"]
    S3["📁 S3 Bucket<br/>Static Site Hosting"]
    APIGW["🔗 API Gateway<br/>REST Endpoint"]
    Lambda["🐍 Lambda<br/>Python 3.11"]
    DDB["📊 DynamoDB<br/>Atomic Counter"]
    
    GH["🐙 GitHub<br/>Source Code"]
    GHA["🤖 GitHub Actions<br/>CI/CD Pipeline"]
    TF["🏗️ Terraform<br/>Infrastructure as Code"]
    
    User -->|Visit| CF
    CF -->|Serve| S3
    S3 -->|fetch /count| APIGW
    APIGW -->|invoke| Lambda
    Lambda -->|increment| DDB
    DDB -->|return count| Lambda
    Lambda -->|response| APIGW
    APIGW -->|JSON| S3
    S3 -->|display| User
    
    GH -->|push| GHA
    GHA -->|deploy| TF
    TF -->|provision| CF
    
    style User fill:#9370DB,stroke:#333,stroke-width:2px
    style CF fill:#9370DB,stroke:#333,stroke-width:2px
    style S3 fill:#9370DB,stroke:#333,stroke-width:2px
    style APIGW fill:#9370DB,stroke:#333,stroke-width:2px
    style Lambda fill:#9370DB,stroke:#333,stroke-width:2px
    style DDB fill:#9370DB,stroke:#333,stroke-width:2px
    style GH fill:#9370DB,stroke:#333,stroke-width:2px
    style GHA fill:#9370DB,stroke:#333,stroke-width:2px
    style TF fill:#9370DB,stroke:#333,stroke-width:2px
```

| Layer | Service | Purpose |
|-------|---------|---------|
| **Frontend** | HTML/CSS/JS + S3 | Static resume with dark theme |
| **CDN** | CloudFront | Global HTTPS delivery |
| **API** | API Gateway | REST endpoint at `/prod/count` |
| **Compute** | Lambda (Python) | Serverless visitor counter |
| **Database** | DynamoDB | Atomic increment, no race conditions |
| **IaC** | Terraform | Infrastructure as code |
| **CI/CD** | GitHub Actions | Push-to-deploy pipeline |

---

## 🐛 What Broke & How I Fixed It

### 🔴 504 Gateway Timeout — CloudFront couldn't reach S3

**Problem:** S3 has two endpoints: a bucket API endpoint and a static website endpoint. I had pointed CloudFront to the bucket endpoint, which requires signed requests.

**Solution:** 
```
✓ Use the website endpoint: s3-us-east-1.amazonaws.com/bucket-name
✓ Enable static website hosting in S3 bucket settings
✓ CloudFront now gets unsigned public access
```

---

### 🔴 Missing Authentication Token — API Gateway path mismatch

**Problem:** The invoke URL needs the full path including stage and resource. I was missing the `/prod/count` path.

**Solution:**
```bash
# ❌ Wrong
https://abc123.execute-api.us-east-1.amazonaws.com/

# ✅ Correct
https://abc123.execute-api.us-east-1.amazonaws.com/prod/count
                                                   ╰─ stage
                                                         ╰─ resource
```

---

### 🔴 CORS blocked the frontend from calling the API

**Problem:** Browsers enforce cross-origin security by default, blocking API calls.

**Solution:**
```
✓ Enable CORS on the API Gateway /count resource
✓ Add header: Access-Control-Allow-Origin: *
✓ Map headers in both Method Response & Integration Response
✓ Test with curl: curl -H "Origin: ..." https://api-endpoint
```

---

## 💰 Cost Breakdown

| Service | Usage | Cost |
|---------|-------|------|
| S3 | ~1 MB storage | $0.00 |
| CloudFront | < 1 GB transfer | $0.00 |
| API Gateway | < 1,000 requests | $0.00 |
| Lambda | < 1,000 invocations | $0.00 |
| DynamoDB | On-demand, 1 item | $0.00 |
| **Total** | | **$0.00/month** |

Entirely within AWS Free Tier.

---

## 🚀 Quick Deploy

```bash
git clone https://github.com/stanleyjnrkanzara-wq/Cloud-Resume-Challenge.git
cd Cloud-Resume-Challenge/terraform
terraform init && terraform apply
aws s3 cp ../index.html s3://$(terraform output -raw bucket_name)/index.html
```

**Cleanup (zero cost):**
```bash
terraform destroy  # Clean up when done
```

---

## 👨‍💻 About Me

Cloud & DevOps enthusiast. AWS Certified Cloud Practitioner. Built this to prove I can ship production infrastructure, not just study certifications.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/stanley-jnr-kanzara-0081133a8)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat&logo=github&logoColor=white)](https://github.com/stanleyjnrkanzara-wq)
[![Email](https://img.shields.io/badge/Email-EA4335?style=flat&logo=gmail&logoColor=white)](mailto:stanleyjnrkanzara@gmail.com)

**📍 Pretoria, South Africa** | Open to cloud and DevOps roles
