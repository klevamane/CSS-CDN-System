# 📦 CSS CDN Repository

This repository serves as the **single source of truth for all CSS assets** used across our applications and email templates.  
It is designed to integrate with an automated CI/CD pipeline that deploys CSS files to an **AWS S3 + CloudFront CDN**, ensuring low-latency, globally accessible stylesheets with zero or minimal costs (via AWS Free Tier).

---

## 🚀 Key Features

- **Centralized CSS management:** All stylesheets live in this repository.
- **Automated deployments:** Push to `main` triggers GitHub Actions to validate, minify, and deploy CSS to AWS S3 + CloudFront.
- **Consistent branding:** Applications and email templates load CSS directly from the CDN, ensuring all users see the latest approved styles.
- **Zero maintenance overhead:** No need to manually update embedded styles across applications or emails.
- **Cost-efficient:** Fully optimized for AWS Free Tier usage.

---

## 📂 Repository Structure

```
styles
├── main.css # General site/app styles
├── email.css # Email template styles
└── components.css # Reusable component styles
.github/workflows
└── deploy.yml # CI/CD pipeline
```

---

## 🛠️ How to Update CSS

### 🔀 Developer Workflow

1. **Clone the repo**
    ```bash
    git clone https://github.com/your-org/css-cdn.git
    cd css-cdn
    ```

2. **Create a feature branch**
    ```bash
    git checkout -b update-button-styles
    ```

3. **Edit CSS files in `/styles`**

4. **Commit & push changes**
    ```bash
    git add .
    git commit -m "Update button styles for consistency"
    git push origin update-button-styles
    ```

5. **Open a Pull Request**
    - PR must be reviewed & approved.

6. **Merge to `main`**
    - Automatically triggers deployment.

---

## 🚀 Deployment Pipeline (CI/CD)

- **Triggered by:** Push or merge to `main`
- **Runs on:** GitHub Actions
- **Steps:**
  1. Validate CSS (`stylelint`)
  2. Minify CSS (`cssnano`)
  3. Upload to AWS S3
  4. Invalidate AWS CloudFront cache
  5. Send notifications on success or failure

✅ **Average end-to-end deploy time:** < 5 minutes.

---

## 🌍 How to Use the CDN

### 💻 For Applications

Reference CSS via stable CDN URL:

```html
<link rel="stylesheet" href="https://cdn.yourdomain.com/styles/main.css">
```

## 💸 Cost Optimization & AWS Free Tier
his system is designed to stay within AWS Free Tier:

AWS Service	Free Tier Usage Plan
- S3	<5GB, ~20,000 GET + 2,000 PUT reqs/mo
- CloudFront	1TB data transfer out + 10M requests/mo
- GitHub Actions	2,000 CI/CD minutes/mo

- Alerts are set to notify when usage hits 80% of free tier limits.


## Monitoring & Alerts (Future Considerations)
Deployment notifications: via Slack/email.

AWS budgets: warn at 80% usage.

Performance goals:

- < 100ms global CDN response times

- 99.9% uptime

- Cache hit ratio > 95%

## Future Enhancements

- 🔄 Staging CDN: Preview CSS before production.

- 🏷️ Versioned CSS: e.g. /styles/main.v1.2.css.

- 💅 SASS/LESS compilation support.

- 📊 Usage analytics dashboard.

- 🔒 Advanced security headers & AWS Shield DDoS protection.



© 2025 kleverman. All rights reserved.

