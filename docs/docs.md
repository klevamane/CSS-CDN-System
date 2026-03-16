# Automated CSS CDN with GitHub and AWS

This project implements an **automated CSS CDN pipeline** using **GitHub**, **GitHub Actions**, **Amazon S3**, and **Amazon CloudFront**.

Whenever CSS files are pushed to the repository, they are automatically deployed to S3 and served globally through a CDN.

The system is designed to:

- Automatically deploy CSS updates
- Serve CSS files globally through a CDN
- Maintain secure private storage
- Remain within **AWS Free Tier limits**
- Require minimal manual maintenance

---

# Architecture Overview


GitHub Repository
↓
GitHub Actions (CI/CD)
↓
AWS S3 Bucket (CSS Storage)
↓
CloudFront CDN (Edge Locations)
↓
End Users / Websites


---

# Core Functionality

- CSS files are stored in a GitHub repository.
- When changes are pushed, GitHub Actions automatically deploys them.
- Files are uploaded to Amazon S3.
- CloudFront serves them globally through a CDN.
- Cache invalidation ensures updates propagate quickly.
- Users can load CSS via a simple `<link>` tag.

Example usage:

```html
<link rel="stylesheet" href="https://YOUR_CLOUDFRONT_DOMAIN/styles/main.css">
Repository Structure

The repository stores all CSS files inside the styles directory.

repo/
│
├── styles/
│   ├── main.css
│   ├── components/
│   │   ├── buttons.css
│   │   └── cards.css
│   └── themes/
│       ├── dark.css
│       └── light.css
│
├── .github/
│   └── workflows/
│       └── deploy.yml
│
└── README.md
AWS Infrastructure Setup
1. Create an S3 Bucket

Create an S3 bucket to store CSS files.

Recommended settings:

Bucket name: cdnonaws

Region: eu-north-1

Block Public Access: Enabled

Object Ownership: Bucket owner enforced

Versioning: Optional

The bucket must remain private.

2. Create Folder Structure in S3

Create a folder called:

styles

Move your CSS files into this folder.

Example S3 structure:

cdnonaws
└── styles/
    └── main.css

Future structure example:

cdnonaws
└── styles/
    ├── main.css
    ├── components/
    │   ├── buttons.css
    │   └── cards.css
    └── themes/
        ├── dark.css
        └── light.css
3. Create a CloudFront Distribution

Create a distribution with the following configuration.

Origin Settings

Origin Domain:

cdnonaws.s3.eu-north-1.amazonaws.com

Origin Type:

S3

Origin Path:

(empty)

Important rules:

❌ /styles/*

❌ /styles

✅ leave empty

4. CloudFront Access to S3

CloudFront must be allowed to read the private bucket.

This is done using Origin Access Control (OAC).

Ensure the S3 bucket policy allows CloudFront to access objects.

Example bucket policy:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontAccess",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::cdnonaws/*"
    }
  ]
}
IAM Setup for GitHub Actions

Create an IAM user for deployments.

User name:

github_deployer

Permissions required:

AmazonS3FullAccess

cloudfront:CreateInvalidation

If using an inline policy, example:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "cloudfront:CreateInvalidation",
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}

Generate access keys:

AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY

GitHub Secrets

Add credentials to repository secrets.

Navigate to:

GitHub → Repository → Settings → Secrets → Actions

Add:

AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
GitHub Actions Workflow

Create the deployment workflow.

File:

.github/workflows/deploy.yml

Example configuration:

name: Deploy CSS to CDN

on:
  push:
    paths:
      - 'styles/**/*.css'

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v3
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: eu-north-1

      - name: Upload CSS files to S3
        run: |
          aws s3 sync ./styles s3://cdnonaws/styles --delete

      - name: Invalidate CloudFront cache
        run: |
          aws cloudfront create-invalidation \
            --distribution-id YOUR_DISTRIBUTION_ID \
            --paths "/*"
Deployment Process

Deployment flow:

Developer pushes CSS changes
        ↓
GitHub Actions pipeline runs
        ↓
Files uploaded to S3
        ↓
CloudFront cache invalidated
        ↓
Updated CSS served globally
Testing the CDN

After deployment completes:

Open the CDN URL.

Example:

https://YOUR_CLOUDFRONT_DOMAIN/styles/main.css

Expected result:

The CSS file loads successfully.

Response headers show CloudFront.

Example header:

x-cache: Hit from cloudfront
Example HTML Usage

Include CSS in a website:

<link rel="stylesheet" href="https://YOUR_CLOUDFRONT_DOMAIN/styles/main.css">

This allows websites to load styles directly from the CDN.

Free Tier Cost Control

This project is designed to operate within AWS Free Tier.

Typical limits:

CloudFront Free Tier:

1 TB data transfer/month

10 million HTTP requests/month

1000 invalidation paths/month

S3 Free Tier:

5 GB storage

20,000 GET requests

2,000 PUT requests

CSS files are small, so typical usage remains far below these limits.

Monitoring and Budget Alerts

To prevent unexpected costs:

Create an AWS budget.

Recommended configuration:

Monthly budget: $1

Alerts at:

50%

80%

90%

Enable billing alerts in AWS.

This ensures you receive notifications before exceeding free tier usage.

Maintenance Guide
Adding New CSS Files

Add files to the repository:

styles/new-file.css

Push to GitHub.

The pipeline will automatically deploy them.

Updating Existing CSS

Modify the file locally:

styles/main.css

Commit and push changes.

CloudFront invalidation will refresh cached content.

Handling Breaking Changes

Use versioned CSS files.

Example:

main.v1.css
main.v2.css

This allows clients to upgrade safely without breaking older integrations.

Security Best Practices

Keep the S3 bucket private

Do not expose AWS credentials

Store secrets only in GitHub Secrets

Restrict IAM permissions where possible

Access CSS only through CloudFront

Final Result

After setup:

CSS files are stored securely in S3

GitHub automatically deploys updates

CloudFront serves content globally

Websites load CSS via a CDN

The system runs automatically with minimal maintenance

Example final CDN URL:

https://YOUR_CLOUDFRONT_DOMAIN/styles/main.css