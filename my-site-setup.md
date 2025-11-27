# My Perfect 2025 S3 + CloudFront Static Site – Full Setup Recap
(Everything done – private, HTTPS, custom domain, auto-updates ready)

## Core Setup (already working)
- S3 bucket: [your-bucket-name]
-123]
  → Static website hosting = DISABLED
  → Block all public access = ON (all 4 boxes checked)
  → Object Ownership = Bucket owner enforced (ACLs disabled)
  → Bucket policy = the one CloudFront auto-added (allows only your distribution)

- CloudFront distribution: [your-distribution-id e.g. E123ABCDEF]
  → Origin = your-bucket-name.s3.amazonaws.com (REST endpoint, chosen from dropdown)
  → Origin Access = Origin Access Control (OAC) – signed requests
  → Viewer protocol = Redirect HTTP → HTTPS
  → Default root object = index.html
  → Custom domain(s): example.com + www.example.com (optional)
  → SSL cert = ACM certificate in us-east-1 (free & auto-renewing)
  → Security protections / WAF = OFF (not needed)

Live URLs (both work with HTTPS):
https://d123abc456.cloudfront.net
https://example.com
https://www.example.com

## Updating the site (2025 automatic way)
Just use this GitHub Actions workflow (already ready to copy):

File: .github/workflows/deploy.yml
```yaml
name: Deploy to S3 + CloudFront

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-region: us-east-1
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

      - name: Sync to S3
        run: aws s3 sync . s3://your-bucket-name --delete --cache-control max-age=0

      - name: Invalidate CloudFront
        run: aws cloudfront create-invalidation --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID }} --paths "/*"
