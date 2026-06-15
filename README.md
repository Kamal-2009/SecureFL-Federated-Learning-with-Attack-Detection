# 🔐 SecureFL — Secure Federated Learning with Anomaly Detection & SOC Dashboard

> A final-year project implementing a **privacy-preserving federated learning system** with real-time **Byzantine attack detection**, **trust scoring**, and a live **SOC monitoring dashboard**.

---

## 📌 Table of Contents

- [Overview](#overview)
- [Key Features](#key-features)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
  - [Backend Setup](#backend-setup)
  - [Client Setup](#client-setup)
  - [Dashboard Setup](#dashboard-setup)
- [How It Works](#how-it-works)
- [Metrics & Evaluation](#metrics--evaluation)
- [Screenshots](#screenshots)
- [Future Work](#future-work)
- [Author](#author)

---

## Overview

**SecureFL** is a federated learning framework designed to detect and mitigate **poisoning attacks** from malicious clients without compromising the privacy of honest participants. The system uses a custom aggregation strategy (`SecureFedAvg`) built on top of [Flower (flwr)](https://flower.dev/) that continuously monitors client model updates for suspicious behavior.

A real-time **SOC (Security Operations Center) Dashboard** built with Next.js displays live trust scores, security alerts, training metrics, and attack success rates — giving operators full visibility into the health of the federated training process.

**Ideal for:** Researchers, students, and engineers exploring the intersection of **federated learning**, **adversarial ML**, and **distributed system security**.

---

## Key Features

- 🛡️ **Byzantine Attack Detection** — Identifies malicious clients injecting poisoned gradients using a multi-signal approach
- 📐 **Cosine Similarity Analysis** — Compares each client's model update against the global average to detect divergence
- 🌲 **Isolation Forest Anomaly Detection** — ML-based outlier detection on flattened model weight vectors
- 📊 **Z-Score History Smoothing** — Tracks each client's anomaly score over the last 3 rounds for stable detection
- 🔒 **Trust Manager** — Dynamically adjusts per-client trust scores; automatically blocks clients falling below threshold
- 🔄 **Rollback Protection** — Skips aggregation rounds if the majority of clients are flagged as suspicious
- 📡 **FastAPI Backend** — Serves real-time alerts, trust scores, and training metrics to the dashboard
- 🖥️ **SOC Dashboard (Next.js)** — Live dark-mode dashboard with charts for accuracy, F1, precision, recall, and ASR

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                    FL Server (Flower)                │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────┐  │
│  │ Cosine Sim   │  │  Isolation   │  │  Z-Score  │  │
│  │  Analysis    │  │   Forest     │  │ Smoothing │  │
│  └──────┬───────┘  └──────┬───────┘  └─────┬─────┘  │
│         └─────────────────┼────────────────┘        │
│                    ┌──────▼──────┐                   │
│                    │Trust Manager│                   │
│                    │(Block/Allow)│                   │
│                    └──────┬──────┘                   │
│                    ┌──────▼──────┐                   │
│                    │  logs.json  │                   │
│                    └──────┬──────┘                   │
└───────────────────────────┼─────────────────────────┘
                            │
                   ┌────────▼────────┐
                   │  FastAPI (8000) │
                   └────────┬────────┘
                            │
                   ┌────────▼────────┐
                   │  Next.js SOC    │
                   │   Dashboard     │
                   └─────────────────┘

Clients: client1, client2, client3, client4, attacker
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| Federated Learning | [Flower (flwr)](https://flower.dev/) |
| Deep Learning | PyTorch |
| Anomaly Detection | scikit-learn (IsolationForest) |
| Backend API | FastAPI + Uvicorn |
| Dashboard | Next.js 14 + TypeScript + Tailwind CSS |
| Charts | Recharts |
| Data Processing | pandas, NumPy, TF-IDF (scikit-learn) |

---

## Project Structure

```
securefl-project/
├── backend/
│   ├── fl_server.py          # Core FL server with SecureFedAvg strategy
│   ├── anomaly_detector.py   # IsolationForest + Z-score detection
│   ├── security.py           # Cosine similarity computation
│   ├── trust_manager.py      # Per-client trust scoring & blocking
│   ├── api.py                # FastAPI endpoints (/alerts, /trust, /metrics)
│   ├── preprocess.py         # Data cleaning & TF-IDF feature extraction
│   └── train_local.py        # Local model training script
│
├── clients/
│   ├── client1.py – client4.py   # Honest federated learning clients
│   ├── attacker.py               # Simulated Byzantine attacker client
│   ├── model.py                  # Shared SimpleModel (PyTorch)
│   └── utils.py                  # Shared utilities
│
├── data/
│   ├── final_merged_dataset_updated.csv   # Raw dataset (not tracked in git)
│   └── processed/                         # Preprocessed .npz files (not tracked in git)
│
└── soc-dashboard/                 # Next.js dashboard
    ├── app/
    │   ├── page.tsx               # Main dashboard (alerts, trust, metrics)
    │   ├── clients/page.tsx       # Per-client trust & status view
    │   ├── metrics/page.tsx       # Training metrics page
    │   └── security/page.tsx      # Security events view
    ├── components/
    │   ├── Sidebar.tsx
    │   ├── Header.tsx
    │   ├── AlertBox.tsx
    │   └── Card.tsx
    └── lib/api.ts                 # API base config
```

---

## Getting Started

### Prerequisites

- Python 3.10+
- Node.js 18+
- pip

---

### Backend Setup

```bash
# 1. Navigate to the backend directory
cd securefl-project/backend

# 2. Install dependencies
pip install flwr torch scikit-learn fastapi uvicorn pandas numpy

# 3. Preprocess the dataset (generates .npz files for each client)
python preprocess.py

# 4. Start the FL server
python fl_server.py
```

---

### Client Setup

Open a separate terminal for each client:

```bash
cd securefl-project/clients

# Start honest clients
python client1.py
python client2.py
python client3.py
python client4.py

# Optionally start the attacker to simulate Byzantine behavior
python attacker.py
```

---

### Dashboard Setup

```bash
# 1. Start the FastAPI backend (in a separate terminal)
cd securefl-project/backend
uvicorn api:app --reload --port 8000

# 2. Install and run the Next.js dashboard
cd securefl-project/soc-dashboard
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) to view the SOC dashboard.

---

## How It Works

### 1. Federated Training (`SecureFedAvg`)
Each round, clients train locally and send model weight updates to the server. The server applies three layers of analysis before aggregation:

### 2. Multi-Signal Anomaly Detection

| Signal | Method | Flag Condition |
|---|---|---|
| Cosine Similarity | Pairwise similarity between all client updates | `avg_sim < threshold` |
| Isolation Forest | Unsupervised outlier detection on flattened weights | Prediction = `-1` (anomaly) |
| Z-Score History | Z-score of similarity over last 3 rounds | `avg_recent < -0.8` |

A client is flagged as **suspicious** if *any* signal triggers.

### 3. Trust Scoring
- Each client starts with `trust = 1.0`
- Suspicious behavior: `trust -= 0.5`
- Normal behavior: `trust += 0.05`
- Clients with `trust < 0.3` are **blocked** from contributing

### 4. Rollback Protection
If more than 50% of clients in a round are flagged, the entire aggregation round is **skipped** to protect the global model.

---

## Metrics & Evaluation

The system tracks the following metrics per round, displayed live on the dashboard:

| Metric | Description |
|---|---|
| **Accuracy** | Weighted federated accuracy across honest clients |
| **Precision** | Of all flagged clients, how many were actual attackers |
| **Recall** | Of all attackers, how many were caught |
| **F1 Score** | Harmonic mean of precision and recall |
| **ASR** | Attack Success Rate — 0.0 means the attack was blocked |

---

## Future Work

- [ ] Add differential privacy (DP-SGD) for gradient-level privacy
- [ ] Support for heterogeneous data distributions (non-IID simulation)
- [ ] Extend dashboard with historical trend analysis
- [ ] Integrate model explainability for suspicious update visualization
- [ ] Deploy backend and dashboard with Docker Compose

---

## Author

**Maida** — Bahria University, Final Year Project (2026)

> Built as a final year project exploring the security of federated learning systems against Byzantine clients.

---

## Keywords

`federated learning` `secure aggregation` `Byzantine fault tolerance` `anomaly detection` `poisoning attack` `isolation forest` `cosine similarity` `trust scoring` `SOC dashboard` `Flower flwr` `PyTorch` `FastAPI` `Next.js` `cybersecurity` `distributed machine learning` `privacy-preserving ML`
