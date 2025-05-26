# Deploying a Secure Static Website on Amazon S3 with CloudFront, WAF, and Route 53

This project demonstrates how to deploy a secure, high-availability static website using the following AWS services:

- Amazon S3 (Simple Storage Service) for static website hosting
- Amazon CloudFront as a Content Delivery Network (CDN)
- AWS WAF for web application firewall protection
- Amazon Route 53 for DNS routing (optional)
- Amazon CloudWatch for monitoring and logging

---

## Architecture Diagram

![Architecture Diagram](Blank%20diagram.png)

---

## Project Overview

Users access the static website through a custom domain managed by **Amazon Route 53**, which routes traffic to **Amazon CloudFront**. **AWS WAF** is integrated with CloudFront to block malicious traffic. CloudFront fetches website files from an **Amazon S3** bucket. All activities and access logs are monitored via **Amazon CloudWatch**.

---

## Step-by-Step Deployment Guide

### Prerequisites

- AWS account with sufficient IAM permissions
- AWS CLI installed and configured
- Static website files (HTML, CSS, JS, images)
- (Optional) Registered domain for Route 53

### Step 1: Create an S3 Bucket for Website Hosting

```bash
aws s3api create-bucket --bucket your-bucket-name --region us-east-1
```

Enable static website hosting:

```bash
aws s3 website s3://your-bucket-name/ --index-document index.html --error-document 404.html
```

Upload your site content:

```bash
aws s3 sync ./website-files/ s3://your-bucket-name/ --acl public-read
```

> Note: For maximum security, remove public read and use CloudFront + Origin Access Control (OAC).

---

### Step 2: Create a CloudFront Distribution

- Go to the AWS Console → CloudFront → Create Distribution
- Set your S3 bucket (not the website endpoint) as the origin
- Enable **Origin Access Control (OAC)** to restrict S3 access
- Set Viewer Protocol Policy to **Redirect HTTP to HTTPS**
- Set **Default Root Object** to `index.html`
- (Optional) Add your custom domain and SSL certificate from AWS Certificate Manager

After deploying, copy the CloudFront domain name (e.g. `d12345678abcdef.cloudfront.net`)

---

### Step 3: Add AWS WAF to Protect Your Application

- Go to AWS Console → WAF & Shield
- Create a Web ACL
- Add **rules** for common attack protection (e.g., SQL injection, IP rate limiting)
- Associate the Web ACL with your CloudFront distribution

---

### Step 4: (Optional) Configure Amazon Route 53

If you have a registered domain:

1. Create a hosted zone in Route 53
2. Add an **A record (alias)** pointing to the CloudFront distribution
3. Wait for DNS propagation

Example DNS config:
```text
Type: A (Alias)
Name: www.example.com
Alias to: CloudFront distribution URL
```

---

### Step 5: Enable CloudWatch Logging

Enable logs for:
- CloudFront access logs (to an S3 bucket)
- WAF logs (via CloudWatch logs)
- S3 server access logs (optional)

Use CloudWatch Dashboards and Alarms to:
- Monitor 4xx/5xx errors
- Track request counts
- Trigger alerts on anomalies

---

## Validation

- Visit your domain or CloudFront URL
- Ensure site loads securely with HTTPS
- Test WAF blocking by simulating bad requests
- View logs and metrics in CloudWatch

---

## Useful Commands

Invalidate CloudFront cache:
```bash
aws cloudfront create-invalidation --distribution-id YOUR_ID --paths "/*"
```

Update S3 bucket policy (if using OAC):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-bucket-name/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::ACCOUNT_ID:distribution/DISTRIBUTION_ID"
        }
      }
    }
  ]
}
```

---

## 🏁 Conclusion

You now have a fully secure, scalable, and high-performing static website architecture using Amazon S3, CloudFront, AWS WAF, Route 53, and CloudWatch. This architecture protects your site from threats, ensures fast delivery, and provides observability for ongoing improvements.

Feel free to customize and extend with CI/CD, Lambda@Edge, or additional WAF rules.

---

**License:** MIT
