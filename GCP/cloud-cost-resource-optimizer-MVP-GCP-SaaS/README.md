# Cloud Cost & Resource Optimizer — MVP (GCP SaaS)

A lightweight SaaS tool for **real-time FinOps and cloud cost optimization** on Google Cloud Platform.

This MVP focuses on analyzing **real infrastructure and billing data** to generate actionable recommendations, **without requiring Terraform or IaC**.

---

## 🚀 Overview

Cloud Cost & Resource Optimizer helps engineering teams, CTOs, and FinOps teams to:

* Analyze real-time GCP costs and resource utilization
* Identify underutilized or misconfigured resources
* Detect anomalies or sudden cost spikes
* Generate actionable reports and alerts
* Reduce monthly cloud spend and improve ROI

Designed for startups, SMEs, and enterprises using GCP who want a simple and cost-effective FinOps tool.

---

## 🧱 Architecture

### 1. Backend

* Connects to **Google Cloud Billing Export (BigQuery)**
* Uses **Cloud Asset Inventory API** to fetch all resources
* Calculates costs, detects unused/overprovisioned resources
* Stores results for historical trend analysis
* Generates JSON outputs and PDF reports

### 2. Frontend

* Web dashboard (React / Next.js)
* Visualizes costs, trends, and resource utilization
* Downloads PDF or CSV reports
* Optional integration with Slack/Teams for alerts

### 3. SaaS Hosting

* Hosted externally (Vendor-hosted SaaS)
* URL public for clients to log in
* Simple authentication (OAuth2 or API key)
* Minimal operational cost (~5–10€/month)

---

## ✨ Features

### MVP Features
* Cost breakdown by project, service, resource type, and labels
* Underutilized VM / SQL / Storage detection
* Alerts for cost anomalies
* Simple PDF and dashboard reports
* API endpoint for integration (optional)

### Upgraded Features (Post-MVP)

* Multi-account support
* Historical trend analysis
* AI-generated recommendations
* Benchmarking vs industry standards
* Advanced reporting and dashboard customization

---

## 📦 Example Output

```
Project: production-gcp

Resource Summary:
  Compute Engine VM-1: $48.20/mo
  Cloud SQL db-prod: $98.00/mo
  GCS Bucket archive: $12.50/mo

Alerts:
  - VM-1 idle 40% of time, consider downsizing
  - GCS Bucket archive overprovisioned, consider lifecycle rules

Total Cost: $158.70/mo
Potential Savings: $31/mo (~19%)
```

---

## 🔧 Installation / Developer Setup

Clone the repository and configure credentials:

```bash
git clone https://github.com/YOUR_ORG/cloud-cost-optimizer
cd cloud-cost-optimizer
pip install -r requirements.txt
```

Run the analysis:

```bash
python run_analysis.py --project my-gcp-project
```

---

## 📝 License

* Backend and frontend code: MIT License
* Uses Google Cloud APIs under standard GCP terms

---

## 📄 Attribution / Legal Notice

```
This product uses Google Cloud APIs (Billing Export, Cloud Asset Inventory) under their standard terms of service.
```
