# 🛡️ Enterprise SOC Command Center & Training Platform

[![Python Version](https://img.shields.io/badge/Python-3.9%2B-blue.svg)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100%2B-green.svg)](https://fastapi.tiangolo.com)
[![Platform Version](https://img.shields.io/badge/Enterprise%20SOC-v2.5%20PRO-navy.svg)](#)
[![Security Hardened](https://img.shields.io/badge/Security-RBAC%20%7C%20JWT%20%7C%20AES--256-darkgreen.svg)](#)

A high-performance, enterprise-grade Security Operations Center (SOC) platform and analyst simulation environment. Features real-time multi-source telemetry ingestion, Sigma rule execution, Scikit-Learn ML anomaly detection, automated SOAR playbooks, STIX2 threat intelligence enrichment, and a dark-mode Web Command Center interface.

---

## 📸 Executive Command Center

![Enterprise SOC Command Center Overview](docs/dashboard_overview.png)

---

## 🎯 Key Capabilities & Features

- **Real-Time Telemetry Processing**: Normalizes and deduplicates telemetry streams (Windows Events, Linux Syslog, Network pcaps/Zeek, Wazuh, Suricata) with high throughput.
- **Dual Detection Engine**:
  - **Rule-Based Engine**: Executes custom Sigma and YARA-like behavioral detection rules.
  - **Machine Learning Engine**: Employs Scikit-learn anomaly detection for zero-day threat identification.
- **11 Integrated SOC Workspaces**:
  1. **Executive Overview**: High-level command center displaying total ingested alerts, unresolved incidents, monitored assets, and EPS metrics.
  2. **Alert Stream & Triage**: Live queue of security alerts with interactive triage, severity badges, and risk scores.
  3. **Detection Rules Engine**: Catalog for reviewing, enabling, or tuning active detection rules.
  4. **Incidents & Cases**: Incident lifecycle management (Containment, Eradication, Post-Incident Review).
  5. **Investigation Graph**: Interactive node-and-link topological visualization of threat relationships and entity graphs.
  6. **Threat Hunting**: Structured query workspace for pro-active threat discovery across historical logs.
  7. **Asset Inventory**: Enterprise asset tracking with posture indicators, IP/hostname mapping, and agent status.
  8. **MITRE ATT&CK Matrix**: Tactical matrix mapping active detections to MITRE ATT&CK tactics & techniques.
  9. **SOC Health Monitor**: Component health, pipeline latency, and telemetry ingestion pipeline statuses.
  10. **SOAR Playbooks**: Automated response orchestration (IP blocking, account isolation, process termination) with full execution logs.
  11. **Reports & Audit**: Audit trail tracking user actions and authenticated CSV/JSON report exports.
- **Enterprise Security Hardening**:
  - Strict **Role-Based Access Control (RBAC)** across 7 distinct role tiers.
  - **JWT Bearer Token Authentication** with session expiration and automated token revocation.
  - **Secure Report Downloading**: Authenticated Blob-based file transfers (no sensitive tokens in URL query strings).
  - Rate limiting, anti-SSRF protection, and strict Pydantic input validation.

---

## 🏗️ Architecture & Pipeline Flow

```text
  Windows Agent / Linux Syslog / Wazuh / Suricata / Zeek
                            │
                            ▼
              REST Ingestion API (Port 8001)
                            │
              ┌─────────────┴─────────────┐
              ▼                           ▼
        Validation Layer            Rate Limiter
              │                           │
              ▼                           ▼
      Normalization Layer            Auth Middleware
              │
              ▼
      Deduplication Layer
              │
        ┌─────┴─────────────┐
        ▼                   ▼
    Event Store       Detection Engine (Rules + ML)
  (SQLite/OpenSearch)       │
                            ▼
                       Alert Engine
                            │
                            ▼
                       SOAR Engine
                            │
                            ▼
              SOC Web Dashboard (Port 8002)
```

---

## 💻 Installation & Environment Setup

### 1. Prerequisites
- **Operating System**: Linux (Ubuntu/Debian/RHEL), macOS, or WSL2.
- **Python**: Version 3.9 or higher (3.10+ recommended).
- **Virtual Environment**: `venv` module.

### 2. Repository Setup
Clone the repository and navigate into the project directory:
```bash
git clone https://github.com/your-org/soc-lab.git
cd "Soc Lab"
```

### 3. Create & Activate Virtual Environment
```bash
python3 -m venv venv
source venv/bin/activate
```

### 4. Install Dependencies
```bash
pip install --upgrade pip
pip install -r requirements.txt
```

### 5. Configure Environment Variables
Copy the example environment configuration to `.env`:
```bash
cp .env.example .env
```

Edit `.env` and set your secret credentials:
```env
JWT_SECRET=
LAB_ADMIN_PASSWORD=YourStrongPass123!
PLATFORM_MODE=lab
DATABASE_URL=sqlite:///soc_data.db
```

### 6. Database Initialization & Credentials Generation
Initialize the SQLite database and create default RBAC lab accounts:
```bash
python3 -c "from src.database import Database; db = Database(); print('✅ DB Initialized')"
```
> **Note**: On first run, a `lab_credentials.txt` file is automatically generated in the project root containing credentials for all lab personas.

---

## 🚀 How to Run the Platform

### Option A: Automated Production Startup (Recommended)

Start all services (REST API, Web UI, and Continuous Lab Threat Generator) with a single command:

```bash
./production_start.sh
```

To gracefully stop all background services:
```bash
./production_stop.sh
```

---

### Option B: Manual Service Launch

If you prefer launching components manually in separate terminals:

1. **Start the REST API Backend** (Port 8001):
   ```bash
   source venv/bin/activate
   set -a; source .env; set +a
   uvicorn src.api:app --host 0.0.0.0 --port 8001 --reload
   ```

2. **Start the Web UI Dashboard** (Port 8002):
   ```bash
   source venv/bin/activate
   uvicorn src.web_ui:app --host 0.0.0.0 --port 8002 --reload
   ```

3. **Start the Continuous Threat Simulation Feed**:
   ```bash
   source venv/bin/activate
   python scripts/generate_realistic_alerts.py --continuous
   ```

---

## 🌐 Web Dashboard & API Endpoints

Once started, access the platform via your browser or HTTP client:

| Resource | URL | Description |
|---|---|---|
| **Web Dashboard** | `http://localhost:8002` | Dark-mode 11-workspace Command Center |
| **API Documentation** | `http://localhost:8001/docs` | Interactive Swagger/OpenAPI UI |
| **API Health Check** | `http://localhost:8001/health` | Backend status check endpoint |
| **Alerts API** | `http://localhost:8001/api/v1/alerts` | Paginated alert feed endpoint |
| **Incidents API** | `http://localhost:8001/api/v1/incidents` | Incident management endpoint |

---

## 🔐 Default Access & RBAC Roles

Log in to the Web Dashboard at `http://localhost:8002` using administrative credentials:
- **Default Username**: `admin`
- **Default Password**: Set via `LAB_ADMIN_PASSWORD` in `.env` (e.g. `soc-lab-admin-change-me`)

Additional predefined role accounts available in `lab_credentials.txt`:

| Username | Role | Key Capabilities |
|---|---|---|
| `analyst_l1` | SOC Analyst L1 | Alert triage, initial status tagging |
| `analyst_l2` | SOC Analyst L2 | Incident escalation, case creation, telemetry ingestion |
| `threat_hunter` | Threat Hunter | Execution of threat hunting queries & IoC lookup |
| `responder` | Incident Responder | Case closure, SOAR playbook execution |
| `engineer` | Detection Engineer | Sigma rule management, MITRE matrix tuning |
| `manager` | SOC Manager | Audit log access, executive metrics, report downloads |
| `readonly` | Read Only | View-only access across telemetry & alerts |

---

## ⚠️ Troubleshooting Common Issues

### ❌ `[Errno 98] Address already in use`
This error occurs when attempting to launch `uvicorn` on port 8001 or 8002 while another process is already listening on that port.

**Solution**:
Run the included stop script to kill existing instances:
```bash
./production_stop.sh
```
Or manually terminate python/uvicorn background processes:
```bash
pkill -f "uvicorn"
```

---

## 🧪 Testing & Validation Suite

Run the full automated test suite to verify system integrity across all platform modules:

```bash
# Run entire test suite
PYTHONPATH=. ./venv/bin/python -m pytest tests/ -v

# Run Dashboard & Web UI tests specifically
PYTHONPATH=. ./venv/bin/python -m pytest tests/test_dashboard.py -v

# Run Phase 9 Analyst Simulation & SOAR tests
PYTHONPATH=. ./venv/bin/python -m pytest tests/test_phase9.py -v
```

---

## 📁 Repository Structure Overview

```text
.
├── docs/                      # Technical documentation & architecture guides
│   ├── dashboard_overview.png # Dashboard screenshot
│   ├── LAB_SETUP.md           # Detailed lab deployment guide
│   └── ...                    # Topic-specific guides (SOAR, Ingestion, etc.)
├── scripts/                   # Simulation feeds & utility scripts
│   ├── generate_realistic_alerts.py
│   └── ...
├── src/                       # Core application source code
│   ├── api.py                 # FastAPI backend routes & middleware
│   ├── web_ui.py              # Web Command Center UI server & HTML/JS
│   ├── database.py            # SQLite ORM & migration logic
│   ├── security.py            # JWT auth, hashing & RBAC permission checks
│   ├── telemetry/             # Multi-source adapters, normalizers & search
│   ├── detection/             # Rule evaluation engine & ML detectors
│   ├── soar.py                # Playbook engine & response actions
│   └── threat_hunting.py      # Query engine for analyst threat hunting
├── tests/                     # Comprehensive Pytest suite (Phases 1 - 9)
├── production_start.sh        # One-click platform startup script
├── production_stop.sh         # One-click platform shutdown script
├── .env.example               # Template environment configuration
└── README.md                  # System overview & operational documentation
```
