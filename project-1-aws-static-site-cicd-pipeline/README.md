# Project-1: AWS Static Site CI/CD Pipeline

![GitHub](https://img.shields.io/badge/source-GitHub-blue?logo=github)
![AWS](https://img.shields.io/badge/AWS-orange)
![S3](https://img.shields.io/badge/AWS-S3-yellow)
![CodeBuild](https://img.shields.io/badge/AWS-CodeBuild-green)
![CodePipeline](https://img.shields.io/badge/AWS-CodePipeline-orange?logo=amazon-aws)
![CodeDeploy](https://img.shields.io/badge/AWS-CodeDeploy-red)
![CloudFront](https://img.shields.io/badge/CDN-CloudFront-yellow)


## 📘 Project Title
**Automating Static Website Deployment from GitHub to AWS using CodePipeline, CodeBuild, S3, and CloudFront**


## Table of Contents
- [Overview](#-overview)
- [Architecture](#-architecture)
- [AWS Services Used](#-aws-services-used)
- [Step-by-Step Implementation](#-step-by-step-implementation)
- [CI/CD in Action](#-cicd-in-action)
- [Cost Estimation](#-cost-estimation)
- [Key Takeaways](#-key-takeaways)
- [Repository](#-repository)
- [Conclusion](#-conclusion)
- [Image Placeholders](#-image-placeholders)


## Overview

This project demonstrates a **fully automated CI/CD pipeline** to deploy a **static website** (HTML, CSS, JS) from **GitHub** to **Amazon S3** using **AWS CodePipeline**, **CodeBuild**, and **CodeDeploy**.

The deployment is further optimized and secured using **CloudFront**, **ACM (Amazon Certificate Manager)**, and **WAF (Web Application Firewall)**.

**Goal:** Zero-manual deployment → Code push on GitHub → Automatically deployed live with HTTPS and CDN.


## Architecture

![AWS Static Site CI/CD Architecture](./assets/01-architecture.png)

### Flow Summary
1. **Developer** writes code and pushes to **GitHub**.  
2. **CodePipeline** detects the push (via Webhook trigger).  
3. **CodeBuild** fetches and builds the project.  
4. **CodeDeploy** extracts and uploads artifacts to **Amazon S3**.  
5. **S3** serves as a static web host.  
6. **CloudFront** provides CDN and caching.  
7. **ACM** provides SSL for HTTPS.  
8. **WAF** protects against common web exploits.


## AWS Services Used

| Service | Role |
|----------|------|
| **GitHub** | Stores source code |
| **Amazon S3** | Hosts the static website |
| **AWS CodeBuild** | Builds (zips) website code |
| **AWS CodeDeploy** | Deploys the build artifacts |
| **AWS CodePipeline** | Automates end-to-end CI/CD process |
| **AWS CloudFront** | Content Delivery Network (CDN) |
| **AWS Certificate Manager (ACM)** | Provides HTTPS certificate |
| **AWS WAF** | Secures against web attacks |
| **AWS IAM** | Manages permissions & roles |
| **AWS Route 53 (Optional)** | Custom domain DNS mapping |


## Step-by-Step Implementation

### Step 1: Setup GitHub Repository

- Store static site files in GitHub (`index.html`, CSS, JS, images, etc.)
- Example structure:
```bash
  ├── index.html
  ├── style.css
  ├── script.js
  ├── images/
  └── buildspec.yml
```

**buildspec.yml**

```yaml
version: 0.2

phases:
  install:
    commands:
      - echo "No installation needed for static HTML"

  pre_build:
    commands:
      - echo "Preparing build..."

  build:
    commands:
      - echo "Building static HTML site"
      - mkdir -p output
      # Copy all files to output directory
      - cp -r * output/ || true
      # Remove buildspec from artifacts
      - rm -f output/buildspec.yml

  post_build:
    commands:
      - echo "Build complete - $(ls -la output/)"
      - echo "Invalidating CloudFront cache..."
      - aws cloudfront create-invalidation --distribution-id E7WP5ANCCTP81 --paths "/*"

artifacts:
  files:
    - '**/*'
  base-directory: output
  discard-paths: no
```

![Static Site files](./assets/02-site-files.png)
![Static Site files on GitHub](./assets/03-site-files-on-github.png)


### Step 2: Create S3 Bucket

1. Go to **AWS Console → S3 → Create Bucket**
2. Name it (e.g.) `static-site-cicd-pipeline-bucket`

![Create Bucket](./assets/04-bucket-name.png)

3. Allow public access by unchecking **“Block all public access”** and create bucket.

![Allow public access to bucket](./assets/05-bucket-public-access.png)

![Bucket creation](./assets/06-bucket-creation.png)

4. Enable **Static Website Hosting**
   * Bucket → Properties → Static website hosting → Enable
   * Index document → `index.html`

![Enable Static Site Hosting](./assets/07-bucket-site-hosting.png)

5. Add bucket policy; replace "Resource" with bucket's ARN:

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::aws-static-site-demo/*"
  }]
}
```

![Add Bucket Policy](./assets/08-bucket-policy.png)


### Step 3: Create AWS CodeBuild Project

1. Navigate to **CodeBuild → Create Build Project**

![Create Build Project](./assets/09-name-build.png)

2. Source Provider → **GitHub**

![Click Manage account credentials](./assets/10-repo-connect.png)

![Choose Github App](./assets/11-choose-github.png)

3. Connect your repo and authorize access.

![Give name to connection](./assets/12-init-github-connection.png)

![Select the repo and authorise](./assets/13-select-repo.png)

![Connect the repo](./assets/14-connect-repo.png)

![Save the connection](./assets/15-save-connection.png)

![Credential added successfully message](./assets/16-repo-connection-success.png)

4. Let AWS create a new **service role** automatically.
5. Set **buildspec.yml** to root of repo.

![Service role creation and buildspec addition](./assets/17-role-and-buildspec.png)

6. Use environment:

   * OS: Amazon Linux 2
   * Runtime: Standard

![Server environment setup](./assets/18-environment.png)

7. Create the project

![Create project](./assets/19-build-project-ready.png)

8. Add CloudFront access permissions to the new user role created

![Add permission to service role](./assets/20-role-policy-attach.png)


### Step 4: Setup AWS CodePipeline

1. Go to **CodePipeline → Create Pipeline → Build custom pipeline**

![Create pipeline](./assets/22-create-pipeline.png)

2. Source stage:

   * Provider: **GitHub**
   * Branch: `main`
   * Trigger: Webhook
  
![Choose github source](./assets/23-source-github.png)

3. Build stage:

   * Provider: **CodeBuild**
   * Choose the build project from Step 3

![Build Options](./assets/24-build-stage.png)

4. Deploy stage:

   * Provider: **Amazon S3**
   * Bucket: `static-site-cicd-pipeline-bucket`
   * Check “Extract file before deploy”

![Deploy options](./assets/25-deploy-stage.png)

5. Review and **Create Pipeline**
6. Give administrator privileges to the new service role created

![Open Service Role](./assets/26-pipeline-role.png)
![Attach admin permissions](./assets/27-pipeline-role-permission.png)

7. Click "Release Change" and Run the pipeline which adds necessary files to host the satic site into the target storage bucket

![Execute pipeline](./assets/28-pipeline-execution.png)

![Deploy files added to S3](./assets/29-files-in-bucket.png)


### Step 5: Verify Deployment

* Open your **S3 website endpoint:**

  ```
  http://static-site-cicd-pipeline-bucket.s3-website.ap-south-1.amazonaws.com
  ```

![Open deploy url](./assets/30-site-url.png)

* You should see your website live!

![Hosted site](./assets/31-Deployed-site-unsecure.png)

- Even though hosted, our site is unsecure and may have high latency for users across the globe (as hosted in one region). To tackle these issues, we use CloudFront CDN for secure and low latency distribution across the globe.


### Step 6: Setup CloudFront CDN

1. Go to **CloudFront → Create Distribution**
2. (Optional) Add custom domain: `www.myawsproject.com` from Route53

![Distribution](./assets/32-create-distribution.png)

3. Origin domain → your S3 bucket website endpoint.

![Attach to S3 endpoint](./assets/33-select-bucket.png)

![Choose website endpoint](./assets/34-origin.png)

4. Enable:

   * “Compress objects automatically”
   * “Redirect HTTP to HTTPS”

5. Click **Create Distribution**.

- Once ready, access:
```
https://dxxxxxxxxx.cloudfront.net
```

![Deployed site url](./assets/35-hosted-site-url.png)

- This deployed static site is now secure with HTTPS (cloudfront domain comes with security)

![Hosted Site](./assets/36-deployed-site-secure.png)


### Step 7: Add Security (ACM + WAF) - If having custom domain

#### ACM

* Go to **AWS Certificate Manager (ACM)**
* Request a certificate for your domain (`*.myawsproject.com`)
* Validate via DNS or Email.

#### WAF

* Navigate to **AWS WAF & Shield**
* Associate WAF Web ACL with your CloudFront distribution.
* Enable common protections: SQL Injection, XSS, IP Rate Limiting, etc.


## CI/CD in Action

Now whenever a developer **pushes new code to GitHub**:

1. **CodePipeline** auto-triggers via Webhook.
2. **CodeBuild** rebuilds and zips the files.
3. **CodeDeploy** deploys the build to S3.
4. **CloudFront cache** invalidates automatically, showing new content globally.

![Change code and commit](./assets/37-commit-changes.png)

![Pipeline reruns](./assets/38-redeploy.png)

![New changes](./assets/39-redeployed-site.png)


## Cost Estimation (Approximate)

| AWS Service                | Monthly Cost (Free Tier Eligible)       |
| -------------------------- | --------------------------------------- |
| Amazon S3 (Static Hosting) | $0.50 – $2                              |
| AWS CodePipeline           | $1 per active pipeline                  |
| AWS CodeBuild              | $0.00 (free tier) – few cents per build |
| AWS CloudFront             | $0.02 / GB (first 50 GB free)           |
| AWS ACM                    | Free                                    |
| AWS WAF                    | ~$5 (optional)                          |
| Route 53 (optional)        | ~$1 per hosted zone                     |

> *Most components are covered under AWS Free Tier for basic usage.*


## Key Takeaways

* 🚀 Fully automated CI/CD pipeline on AWS
* 🔄 Continuous integration directly from GitHub
* 🌍 Global delivery through CloudFront CDN
* 🔐 HTTPS enabled using ACM
* 🧱 Security hardened with WAF
* 🧩 Reusable for production or portfolio projects


## Repository

The code used for the project is at the [GitHub Repository – aws-static-site-cicd-pipeline](https://github.com/dsaikiran01/aws-static-site-cicd-pipeline)


## Conclusion

This project showcases **end-to-end DevOps automation** using **AWS services** to host and manage a **static web application** with speed, scalability, and security.


**If you found this project helpful, give the repo a star! ⭐**
