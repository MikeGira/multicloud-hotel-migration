# Multi-Cloud Hotel App Migration — Project CLAUDE.md

## WHAT THIS PROJECT IS
Migration of a web application ("Luxxy COVID Testing System") from on-premises
to a multi-cloud architecture, completed as part of The Cloud Bootcamp's
Intensive Cloud Training (2023). Original live deployment has since expired
(AWS/GCP free tier). This project is being revived and extended.

**Original cert:** docs/Michael Twagirayezu - 2023-07-03 - Multi-cloud cert.pdf
**Project evidence:** docs/Project Evidence - ....pdf
**Architecture diagram:** docs/Solution Architecture - Intensive Cloud Training.png

## ORIGINAL ARCHITECTURE (2023)
| Layer | Technology |
|-------|-----------|
| App | Python/Flask (Luxxy COVID Testing System) |
| Container | Docker |
| Orchestration | GCP Kubernetes Engine (GKE) |
| Database | GCP Cloud SQL (MySQL) |
| File Storage | AWS S3 |
| IaC | Terraform |
| Credentials setup | aws_set_credentials.sh / gcp_set_project.sh |

## FILE STRUCTURE
```
multicloud-hotel-migration/
├── app/                          # Flask application (Python)
│   ├── app.py / config.py / models.py / forms.py / helpers.py
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── static/                   # CSS, JS, images (Bootstrap + jQuery)
│   └── templates/                # Jinja2 HTML templates
├── infrastructure/               # Terraform IaC
│   ├── main.tf                   # AWS provider config
│   ├── aws_variables.tf          # AWS region + credentials path
│   ├── gcp_variables.tf          # GCP project ID + region
│   ├── tcb_aws_storage.tf        # S3 bucket definition
│   ├── tcb_gcp_database.tf       # Cloud SQL instance definition
│   ├── aws_set_credentials.sh    # Sets up ~/.aws/credentials from CSV
│   └── gcp_set_project.sh        # Sets GCP project ID via gcloud
├── kubernetes/
│   └── luxxy-covid-testing-system.yaml   # GKE deployment manifest
├── db/
│   ├── create_table.sql          # Initial schema (MySQL)
│   └── db_dump.sql               # Full data dump for migration
└── docs/
    ├── certificate.pdf
    ├── project-evidence.pdf
    ├── Solution Architecture.png
    └── screenshots/              # Evidence: running app, K8s cluster, S3, etc.
```

## REQUIRED CREDENTIALS (never commit — use env vars or tfvars)
- AWS: Access Key ID + Secret Access Key (feed into aws_set_credentials.sh or export as env vars)
- GCP: Project ID (gcp_set_project.sh reads from `gcloud config get-value project`)
- GCP: Service Account JSON key for Terraform GCP provider

## HOW TO RE-DEPLOY (original stack)
```bash
# 1. AWS credentials
./infrastructure/aws_set_credentials.sh ./accessKeys.csv   # or use env vars

# 2. GCP project
gcloud config set project YOUR_PROJECT_ID
./infrastructure/gcp_set_project.sh

# 3. Provision infrastructure
cd infrastructure
terraform init
terraform plan
terraform apply

# 4. Build + push Docker image
docker build -t gcr.io/YOUR_PROJECT_ID/luxxy-app ./app
docker push gcr.io/YOUR_PROJECT_ID/luxxy-app

# 5. Deploy to GKE
kubectl apply -f kubernetes/luxxy-covid-testing-system.yaml

# 6. Migrate database
# Connect to Cloud SQL and run: db/create_table.sql then db/db_dump.sql

# 7. Upload PDFs to S3
# (original mission3 data — optional for demo purposes)
```

## ═══════════════════════════════════════════════════
## REVIVAL OPTIONS — PICK ONE (OR ALL) WHEN READY
## ═══════════════════════════════════════════════════

Mike wants to explore all 4 options. They are not mutually exclusive.

### Option 1 — GitHub Repo + README as Portfolio Proof (Recommended first step)
The Terraform + K8s + Dockerfile code IS the demonstration. Add an architecture
diagram and screenshots to the README. Most senior engineers and recruiters
understand that infra portfolios live in code, not running instances.
**Effort:** Low. **Cost:** $0. **Portfolio value:** High.
**Action:** Polish the README, add architecture diagram, push to GitHub.

### Option 2 — Oracle Cloud Free Tier (Keep it LIVE at $0 forever)
Oracle has a genuinely permanent free tier that never expires:
- 2 ARM VMs (combined 4 OCPUs, 24GB RAM) — enough to run k3s
- 50GB block storage, 1 load balancer
**Plan:** Deploy k3s on Oracle ARM VMs, redeploy the Flask app and MySQL there.
It is no longer technically "multi-cloud" but the app stays live with a real URL.
The original Terraform + GKE code stays in the repo as evidence of the real thing.
**Effort:** Medium. **Cost:** $0 forever. **Portfolio value:** High (live URL).
**Action:** Create Oracle Cloud account → provision ARM VMs → install k3s → deploy.

### Option 3 — Migrate App to Vercel + Supabase (Gold Standard Stack)
Redeploy the hotel app on Mike's proven production stack (free, permanent).
Keep the original Terraform IaC code in the repo.
- Convert Flask → Vercel Serverless Functions (or keep as Docker on Railway/Render free tier)
- Replace Cloud SQL → Supabase PostgreSQL (free tier, RLS)
- Replace AWS S3 → Supabase Storage or Cloudflare R2 (free tier, no egress fees)
Portfolio card links to the live app AND the GitHub IaC code.
**Effort:** Medium-High (Flask → serverless conversion). **Cost:** $0. **Portfolio value:** Very high.
**Note:** Railway and Render both have free tiers that run Docker containers — easier
than a full Flask-to-serverless rewrite.

### Option 4 — Loom/YouTube Walkthrough Video
Record a 2-3 min demo of the running app + narrate the architecture.
Link it from the portfolio card. Very common and well-respected for
infrastructure projects. Pairs well with Options 1, 2, or 3.
**Effort:** Low (if original screenshots/recordings exist). **Cost:** $0.
**Action:** Record screen walkthrough using existing docs/screenshots as slides.

### Option 5 — Redo on AWS + GCP (Re-run the original)
Spin up new free-tier accounts (or use existing ones) and re-run the
original Terraform deployment. AWS free tier is 12 months from account
creation; GCP gives $300 credit to new accounts.
**Risk:** Free tiers expire again. **Good for:** Refreshing the hands-on skill,
capturing new screenshots, recording a fresh demo video.
**Tip:** Combine with Option 4 — run it, record it, tear it down.

## SECURITY NOTES
- NEVER commit accessKeys.csv or any .tfvars with real values
- NEVER commit GCP service account JSON keys
- Use environment variables or a secrets manager for all credentials
- .gitignore is already configured to block all of the above
- Run `git log` before pushing to verify no secrets slipped in

## SUGGESTED NEXT SESSION ORDER
1. Push current cleaned-up code to GitHub (Option 1 foundation)
2. Decide which live-hosting option to pursue
3. If Option 2: Create Oracle Cloud account and start k3s setup
4. If Option 3: Start with Railway (easiest Docker path) or Vercel Functions
5. If Option 5: Spin up new GCP free trial, re-run Terraform, record video
