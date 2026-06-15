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
## Architecture

```mermaid
flowchart LR
    subgraph Frontend
        A[Visitor 🧑‍💻] --> B[CloudFront CDN<br/>HTTPS | Edge Cache]
        B --> C[S3 Bucket<br/>Static Website]
    end
    
    B --> D[API Gateway<br/>Stage: prod | /count]
    
    subgraph API
        D --> E[Lambda<br/>Python 3.11 | Boto3]
    end
    
    subgraph Data
        E --> F[DynamoDB<br/>Atomic Increment]
    end
    
    subgraph Automation
        G[GitHub] --> H[GitHub Actions]
        H --> I[Terraform]
        I --> B
    end
    
    style A fill:#e9ecef
    style B fill:#FF9900,color:#fff
    style C fill:#FF9900,color:#fff
    style D fill:#FF9900,color:#fff
    style E fill:#FF9900,color:#fff
    style F fill:#fff3cd



## What I Built

A full-stack serverless resume that tracks every visitor. Refresh the page and watch the counter grow.

```
Browser → CloudFront (CDN + HTTPS) → S3 (static site)
              ↓
         JavaScript calls API
              ↓
    API Gateway → Lambda (Python) → DynamoDB (counter)
```

**Every layer is real, every service is managed, and the bill is $0.00.**

---

## Architecture

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

## What Broke & How I Fixed It

**504 Gateway Timeout — CloudFront couldn't reach S3.**

S3 has two endpoints: a bucket API endpoint and a static website endpoint. I had pointed CloudFront to the bucket endpoint, which requires signed requests. The fix was using the website endpoint (`s3-website-us-east-1.amazonaws.com`) with HTTP-only origin protocol.

**Missing Authentication Token — API Gateway path mismatch.**

The invoke URL needs the full path: `https://{id}.execute-api.us-east-1.amazonaws.com/prod/count`. The stage (`prod`) and resource (`/count`) must both be present. I rebuilt the API resource tree, redeployed the stage, and the counter started working.

**CORS blocked the frontend from calling the API.**

Browsers enforce cross-origin security. I enabled CORS on the API Gateway `/count` resource with `Access-Control-Allow-Origin: *`, mapped the headers in both Method Response and Integration Response, then redeployed.

---

 Cost

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

 Quick Deploy

```bash
git clone https://github.com/stanleyjnrkanzara-wq/Cloud-Resume-Challenge.git
cd terraform
terraform init && terraform apply
aws s3 cp ../index.html s3://$(terraform output -raw bucket_name)/index.html
```

```bash
terraform destroy  # Clean up when done — zero cost
```

---

## About Me

Cloud & DevOps enthusiast. AWS Certified Cloud Practitioner. Built this to prove I can ship, not just study.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/stanley-jnr-kanzara-0081133a8)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat&logo=github&logoColor=white)](https://github.com/stanleyjnrkanzara-wq)
[![Email](https://img.shields.io/badge/Email-EA4335?style=flat&logo=gmail&logoColor=white)](mailto:stanleyjnrkanzara@gmail.com)

Based in Pretoria, South Africa. Open to cloud and DevOps roles.

