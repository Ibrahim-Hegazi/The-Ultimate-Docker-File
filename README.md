I've updated the README by removing all sections that reference components or configurations no longer present in your current docker-compose.yml and .env files. Here's the cleaned-up version:

---

# Ultimate Data Engineering Sandbox Docker Compose Setup

A comprehensive, production-like data engineering development environment with Apache Airflow, Kafka, Spark, PostgreSQL, and more - all containerized for easy setup and collaboration.

This complete stack provides everything needed for data engineering projects: orchestration (Airflow), streaming (Kafka), processing (Spark), storage (PostgreSQL), and development environments (Jupyter/Python) - all pre-configured to work together seamlessly.

---

## 📌 Table of Contents

- [🔎 Project Overview](#-project-overview)
- [🚀 Features & Capabilities](#-features--capabilities)
- [🏗️ System Architecture](#️-system-architecture)
- [📋 Prerequisites & Installation](#-prerequisites--installation)
- [🚦 Quick Start Guide](#-quick-start-guide)
- [📁 Complete Project Structure & Service Description](#-complete-project-structure--service-description)
- [🔍 Understanding the .gitignore File](#-understanding-the-gitignore-file)
- [🔧 Service Configuration](#-service-configuration)
- [📂 Volume Structure](#-volume-structure)
- [🌐 Network & Communication](#-network--communication)
- [🧪 Development Environments](#-development-environments)
- [📊 Monitoring & Management UIs](#-monitoring--management-uis)
- [💾 Data Persistence](#-data-persistence)
- [🔄 Service Dependencies](#-service-dependencies)
- [🔑 Service Credentials Reference](#-service-credentials-reference)
- [🔌 Connection Strings Reference](#-connection-strings-reference)
- [📝 Step-by-Step pgAdmin Setup](#-step-by-step-pgadmin-setup)
- [❓ Common Mistakes to Avoid](#-common-mistakes-to-avoid)
- [🚀 Quick Reference: Where to Put Your Code](#-quick-reference-where-to-put-your-code)
- [🐛 Troubleshooting Guide](#-troubleshooting-guide)
- [🔒 Security Considerations](#-security-considerations)
- [📈 Performance Optimization](#-performance-optimization)
- [🧩 Extending the Stack](#-extending-the-stack)
- [📚 Learning Resources](#-learning-resources)
- [📝 Final Notes](#-final-notes)

---

## 🔎 Project Overview

This Docker Compose setup creates a complete data engineering sandbox environment that mirrors real-world production architectures. It's designed for:

- **Data Engineers** building and testing ETL/ELT pipelines
- **Data Scientists** needing Spark access with Kafka streaming data
- **Data Analysts** working with SQL databases and BI preparation
- **Students & Learners** exploring modern data stack technologies
- **Team Collaboration** with consistent, reproducible environments

The stack separates concerns clearly:
- **PostgreSQL-airflow**: Airflow's metadata (never use for application data!)
- **PostgreSQL-app**: Your business data warehouse
- **Kafka + Schema Registry**: Event streaming backbone
- **Airflow (webserver + scheduler + worker)**: Workflow orchestration
- **python-dev**: General-purpose development with PySpark and Jupyter
- **pgAdmin + Kafka UI**: Visual management tools

## 🚀 Features & Capabilities

### Core Data Engineering Features

| Feature | Components | Use Case |
|---------|------------|----------|
| **Workflow Orchestration** | Airflow (CeleryExecutor) | Schedule and monitor DAGs, distributed task execution |
| **Stream Processing** | Kafka (KRaft mode) | Real-time data ingestion and event streaming |
| **Batch Processing** | PySpark, Jupyter | Large-scale data transformations, ML preparation |
| **Data Warehousing** | PostgreSQL (x2) | Transactional metadata + analytical data |
| **Schema Management** | Schema Registry | Avro/Protobuf schema versioning for Kafka |
| **Development Environments** | Python, Jupyter | Isolated coding spaces with shared volumes |

### Development & Debugging Features

- **Hot-reload DAGs**: Edit Airflow DAGs and see changes immediately
- **Persistent pip packages**: Installed Python packages survive container restarts
- **Shared data volumes**: All services can access the same data files
- **Multiple UIs**: Web interfaces for every component
- **Pre-installed connectors**: Airflow Spark/Kafka providers ready to use

### Production-Like Characteristics

- **Separate metadata vs. data databases** (crucial for production)
- **Distributed Airflow** (webserver, scheduler, worker)
- **Persistent volumes** for all stateful services
- **Healthchecks** ensuring proper startup order
- **KRaft Kafka** (no Zookeeper dependency)

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              HOST MACHINE                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │   Port 5432  │  │   Port 5433  │  │   Port 6379  │  │   Port 9092  │    │
│  │   Postgres   │  │   Postgres   │  │    Redis     │  │    Kafka     │    │
│  │   Airflow    │  │     App      │  │              │  │              │    │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘    │
│                                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐    │
│  │   Port 8080  │  │   Port 8888  │  │   Port 5050  │  │   Port 8090  │    │
│  │   Airflow    │  │   Jupyter    │  │   pgAdmin    │  │   Kafka UI   │    │
│  │   Webserver  │  │   Lab        │  │              │  │              │    │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘    │
└─────────────────────────────────────────────────────────────────────────────┘
                                   │
                          ┌────────┴────────┐
                          │   data-net      │
                          │   (bridge)      │
                          └────────┬────────┘
                                   │
┌─────────────────────────────────────────────────────────────────────────────┐
│                           CONTAINER NETWORK                                 │
│                                                                             │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐               │
│  │  postgres-   │────▶│    redis     │────▶│    broker    │               │
│  │   airflow    │     │              │     │   (Kafka)    │               │
│  └──────────────┘     └──────────────┘     └───────┬──────┘               │
│         │                                           │                      │
│         ▼                                           ▼                      │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐               │
│  │   airflow-   │     │    schema-   │     │   kafka-ui   │               │
│  │   webserver  │     │   registry   │     │              │               │
│  └──────────────┘     └──────────────┘     └──────────────┘               │
│         │                      │                      │                    │
│         ▼                      ▼                      ▼                    │
│  ┌──────────────┐     ┌──────────────┐                                   │
│  │   airflow-   │     │  python-dev  │                                   │
│  │   scheduler  │     │  (Jupyter)   │                                   │
│  └──────────────┘     └──────────────┘                                   │
│         │                      │                                          │
│         ▼                      ▼                                          │
│  ┌──────────────┐     ┌──────────────┐                                   │
│  │   airflow-   │     │ postgres-app │                                   │
│  │    worker    │────▶│              │                                   │
│  └──────────────┘     └──────────────┘                                   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Data Flow Description

| Step | From | To | Description |
|------|------|-----|-------------|
| 1 | Data Generator | Kafka | Simulated real-time data streams |
| 2 | Kafka | PySpark | Stream processing & transformations |
| 3 | PySpark | postgres-app | Batch load processed data |
| 4 | Airflow DAGs | postgres-app | Scheduled ETL jobs |
| 5 | Airflow Workers | Spark Jobs | Distributed task execution |
| 6 | All Services | Shared Volumes | Code and data persistence |

---

## 📋 Prerequisites & Installation

### Required Software

Before starting, ensure you have the following installed:

| Software | Version | Purpose |
|----------|---------|---------|
| **Docker** | 20.10.0 or higher | Container runtime |
| **Docker Compose** | 2.0.0 or higher | Multi-container orchestration |
| **Git** | Latest | Version control |
| **WSL 2** (Windows only) | Latest | Linux kernel for Docker |

### System Requirements

- **RAM**: At least 8GB allocated to Docker (16GB recommended)
- **Disk Space**: 20GB free space (for images and data)
- **CPU**: 4 cores minimum (8 cores recommended)

### Port Availability

Ensure these ports are free on your host machine:
- `5432` - PostgreSQL Airflow
- `5433` - PostgreSQL Application
- `6379` - Redis
- `8080` - Airflow Webserver
- `8888` - Jupyter Lab
- `5050` - pgAdmin
- `8090` - Kafka UI
- `9092` - Kafka Broker
- `4040` - Spark UI

### Windows Users: WSL2 Setup (Required for Docker Desktop)

If you're on Windows, you need to install WSL2 before Docker Desktop:

#### Step 1: Enable WSL (Run as Administrator)

Open **PowerShell or Command Prompt as Administrator** and run:

```powershell
# Enable WSL feature
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# Enable Virtual Machine Platform
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# Restart your computer
Restart-Computer
```

#### Step 2: Set WSL2 as Default

After restart, open PowerShell as Administrator again:

```powershell
wsl --set-default-version 2
```

#### Step 3: Install Linux Distribution

Open Microsoft Store and install **Ubuntu 20.04 LTS** or **Ubuntu 22.04 LTS**

#### Step 4: Launch Ubuntu and Create User

- Launch Ubuntu from Start Menu
- Wait for installation to complete
- Create a username and password when prompted

#### Step 5: Install Docker Desktop

1. Download [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop/)
2. Run installer
3. During installation, ensure **"Use WSL 2 instead of Hyper-V"** is checked
4. After installation, open Docker Desktop
5. Go to Settings → Resources → WSL Integration
6. Enable integration with your Ubuntu distribution

#### Step 6: Verify Installation

```bash
# In your Ubuntu terminal or PowerShell
docker --version
docker-compose --version
wsl --list --verbose  # Should show your Ubuntu with version 2
```

### Linux/Mac Users

For Linux or Mac users, simply install Docker Desktop or Docker Engine directly from the [official Docker website](https://docs.docker.com/get-docker/).

### Recommended Knowledge

- Basic Docker commands (`docker-compose up`, `docker ps`, etc.)
- Understanding of data engineering concepts
- Familiarity with the tools in the stack

---

## 🚦 Quick Start Guide

### Step 1: Clone and Prepare

```bash
# Create project directory
mkdir data-engineering-sandbox
cd data-engineering-sandbox

# Save docker-compose.yml and .env files in this directory
# (copy the provided files)
```

### Step 2: Create Directory Structure

```bash
# Create all required directories (these will be mounted as volumes)
mkdir -p postgres-airflow/data
mkdir -p postgres-app/data
mkdir -p airflow/{dags,logs,plugins,scripts}
mkdir -p pgadmin
mkdir -p kafka/data
mkdir -p dev_code
mkdir -p checkpoint
mkdir -p data
mkdir -p sql_scripts
mkdir -p postgres/backups
```

### Step 3: Pull Docker Images

```bash
# Pull each image separately to avoid timeout issues
docker pull postgres:15
docker pull redis:7-alpine
docker pull confluentinc/cp-kafka:7.6.1
docker pull confluentinc/cp-schema-registry:7.6.1
docker pull dpage/pgadmin4:7
docker pull apache/airflow:2.7.3-python3.11
docker pull jupyter/pyspark-notebook:spark-3.4.1
docker pull provectuslabs/kafka-ui:latest
```

### Step 4: Start the Stack

```bash
# Start all services
docker-compose up -d

# View logs (optional)
docker-compose logs -f

# Check service status
docker-compose ps

# Optional Specific Logs for airflow
docker-compose logs -f airflow-webserver

# Optional Specific Logs for python-dev
docker-compose logs -f python-dev
```

### Step 5: Verify Installation

Access each service's UI to verify they're running:

| Service | URL | Credentials |
|---------|-----|-------------|
| Airflow | http://localhost:8080 | admin / admin |
| Jupyter Lab | http://localhost:8888 | token: devtoken |
| pgAdmin | http://localhost:5050 | admin@example.com / admin |
| Kafka UI | http://localhost:8090 | (no login) |
| Spark UI | http://localhost:4040 | (appears when Spark runs) |

### Step 6: Stop the Stack

```bash
# Stop all services (preserves data)
docker-compose down

# Stop and remove volumes (WARNING: deletes all data!)
docker-compose down -v
```

---

## 📁 Complete Project Structure & Service Description

### Your Workspace Directory Structure
```
your-project/
│
├── 📄 docker-compose.yml          # Service definitions and configuration
├── 📄 .env                        # Environment variables (credentials, paths)
├── 📄 .gitignore                  # Specifies which files Git should ignore
│
├── 📁 postgres-airflow/           # Airflow metadata database
│   └── 📁 data/                    # PostgreSQL binary files ⚠️ DO NOT EDIT
│
├── 📁 postgres-app/                # Application database
│   └── 📁 data/                     # PostgreSQL binary files ⚠️ DO NOT EDIT
│
├── 📁 pgadmin/                     # pgAdmin settings and saved connections
│
├── 📁 airflow/                     # Airflow core files
│   ├── 📁 dags/                      # Your DAG definitions (Python files)
│   ├── 📁 logs/                       # All Airflow logs (scheduler, tasks, webserver)
│   ├── 📁 plugins/                    # Custom Airflow plugins (operators, hooks)
│   └── 📁 scripts/                    # Helper scripts imported by DAGs
│
├── 📁 dev_code/                    # Development code for Jupyter/Python
│
├── 📁 checkpoint/                   # Spark streaming checkpoints (fault recovery)
│
├── 📁 data/                         # Shared data files (CSV, Parquet, JSON)
│   └── 📄 .gitkeep                   # Placeholder to keep empty folder in Git
│
├── 📁 sql_scripts/                  # SQL scripts for database operations
│
├── 📁 postgres/                      # PostgreSQL backups
│   └── 📁 backups/                    # SQL backups from pgAdmin
│
└── 📁 kafka/                        # Kafka data persistence
    └── 📁 data/                       # Kafka topics and messages ⚠️ BINARY FILES
```

---

## 🔍 Understanding the `.gitignore` File

The `.gitignore` file is a critical component of any data engineering project. It tells Git which files and directories to **explicitly ignore** - meaning they won't be committed to the repository or tracked for changes.

### Why We Need a `.gitignore` File

In data engineering projects, we deal with many types of files that should **never** be committed to version control:

---

#### 1. Binary Database Files 🗄️

```
/postgres-airflow/data/
/postgres-app/data/
/kafka/data/
```

**Why ignore?** These contain raw database files that are:
- **Machine-specific** (can't be shared across environments)
- **Extremely large** (can be GBs or TBs)
- **Binary format** (Git can't show meaningful diffs)
- **Constantly changing** (every query modifies them)

---

#### 2. Log Files 📋

```
/airflow/logs/
*.log
```

**Why ignore?** Logs are:
- **Auto-generated** (no need to version control)
- **Rapidly growing** (can fill up your repo quickly)
- **Development-specific** (different for each user)
- **Better viewed** through Docker or logging systems

---

#### 3. Large Datasets 📊

```
/data/*.csv
/data/*.parquet
/data/*.json
/data/*.avro
```

**Why ignore?** Data files:
- **Can be massive** (GBs of CSV data)
- **Change frequently** (new data daily)
- **Belong in data lakes**, not Git repos
- **Should be downloaded separately** or generated

---

#### 4. Python Cache Files 🐍

```
__pycache__/
*.pyc
```

**Why ignore?** These are:
- **Auto-generated bytecode**
- **Python version-specific**
- **Recreated automatically** when running code
- **Clutter** that distracts from real code

---

#### 5. IDE and Editor Files 💻

```
.vscode/
.idea/
*.swp
```

**Why ignore?** These are:
- **User-specific settings**
- **Different for every team member**
- **Can cause conflicts** if shared
- **Better kept local**

---

#### 6. Checkpoint Files 🔄

```
/checkpoint/
```

**Why ignore?** Spark checkpoints are:
- **Binary state information**
- **Streaming-specific**
- **Recoverable** from Kafka offsets
- **Different for each run**

---

### The Smart Pattern: `!data/.gitkeep`

One of the most important patterns in our `.gitignore` is:

```gitignore
# Ignore all data files in the /data directory
/data/*.csv
/data/*.parquet
/data/*.json
# ... etc

# BUT keep the folder structure!
!data/.gitkeep
```

This solves a common Git problem: **Git doesn't track empty folders**.

---

#### How `.gitkeep` Works:

**Without `.gitkeep`:**

```bash
mkdir data
git add data  # Git says: "nothing added"
# The empty data folder NEVER makes it to the repo
```

**With `.gitkeep`:**

```bash
touch data/.gitkeep
git add data/.gitkeep  # Git adds the file, which brings the folder
# Now when someone clones, they get an empty data/ folder!
```

**Combined with `.gitignore`:**
- The `.gitkeep` file is the **ONLY** file in `data/` that gets committed
- When users add their own CSV/Parquet files, Git ignores them
- The folder structure is preserved for everyone

---

### What This Means for You

When you clone this repository, you'll see:

```
your-project/
├── data/
│   └── .gitkeep          # <-- The only file in Git
```

After you start working and add your datasets:

```
your-project/
├── data/
│   ├── .gitkeep          # <-- Still in Git (but hidden by .gitignore)
│   ├── sales.csv         # <-- Your local file (ignored by Git)
│   ├── customers.parquet # <-- Your local file (ignored by Git)
│   └── backup.zip        # <-- Your local file (ignored by Git)
└── ...
```

When you run `git status`, you'll see:

```bash
$ git status
On branch main
nothing to commit, working tree clean  # Your data files don't show up!
```

---

### Benefits of This Approach

✅ **Clean Repository** - Only code and configuration, no binary bloat  
✅ **Fast Clones** - Smaller repo size means faster downloads  
✅ **No Conflicts** - Binary files won't cause merge conflicts  
✅ **Preserved Structure** - Empty folders remain for guidance  
✅ **Developer-Friendly** - Clear where to put data files  
✅ **Production-Ready** - Follows data engineering best practices  

---

### Key Takeaway

The `.gitignore` file isn't just about "ignoring files" - it's about **curating what belongs in your repository**. For a data engineering project:

- ✅ **DO commit**: Code, configuration, documentation, small samples
- ❌ **DON'T commit**: Databases, logs, large datasets, binaries

This keeps your repository focused on what matters: the **logic** and **configuration** that defines your data pipelines, not the **data itself** which belongs in data lakes or object storage.

---

## 📁 Repository Contents

This repository includes:

✅ **Configuration files** - `docker-compose.yml`, `.env` (with default credentials for learning)  
✅ **Code directories** - `airflow/dags/`, `dev_code/`, etc.  
✅ **Documentation** - This README  
✅ **Git configuration** - `.gitignore` to keep the repo clean  

The following are **excluded** (via `.gitignore`) to keep the repository clean and efficient:
- Binary database files (`postgres-*/data/`, `kafka/data/`)
- Log files (`airflow/logs/`)
- Large datasets (`data/*.csv`, `data/*.parquet`, etc.)
- Python cache files (`__pycache__/`)
- IDE settings (`.vscode/`, `.idea/`)
- Spark checkpoints (`/checkpoint/`)

To get started, clone the repo and run:

```bash
# Create all required directories (the .gitkeep files preserve structure)
mkdir -p postgres-airflow/data postgres-app/data kafka/data checkpoint pgadmin airflow/logs data

# Start the stack
docker-compose up -d
```

---

### 📊 Complete Service Reference Table

| Service | Service Type | What It Does | Code Location (Host) | Input Directory | Output Directory | Port | Access URL | When to Use |
|---------|--------------|--------------|---------------------|-----------------|------------------|------|------------|-------------|
| **postgres-airflow** | Database | Stores Airflow metadata (DAG runs, task states, connections, variables) | N/A | `./postgres-airflow/data/` (DB files) | Same as input | `5432` | `localhost:5432` | **NEVER write directly** - Airflow owns this DB exclusively |
| **postgres-app** | Database | Stores your application/business data (data warehouse) | N/A | `./postgres-app/data/` (DB files) | Same as input | `5433` | `localhost:5433` | Store results from Spark jobs, API data, dimensional models |
| **redis** | Message Broker | Handles communication between Airflow components (Celery backend) | N/A | No persistent storage | No persistent storage | `6379` | `localhost:6379` | Internal Airflow communication only (ephemeral) |
| **pgadmin** | Management UI | Web interface to manage both PostgreSQL databases | `./pgadmin/` (settings) | `./pgadmin/` | Same as input | `5050` | `http://localhost:5050` | Manual DB inspection, run SQL queries, import/export data |
| **airflow-webserver** | Orchestration | Airflow UI, DAG parser, trigger DAG runs, visual interface | `./airflow/dags/`<br>`./airflow/plugins/`<br>`./airflow/scripts/`<br>`./dev_code/` | `./airflow/dags/` (DAGs)<br>`./airflow/plugins/`<br>`./airflow/scripts/`<br>`./dev_code/` | `./airflow/logs/` (logs) | `8080` | `http://localhost:8080` | Monitor DAGs, trigger runs, check task status, view logs |
| **airflow-scheduler** | Orchestration | Schedules when to run DAGs based on time/triggers | Same as webserver | Same as webserver | Same as webserver | None | N/A | Internal - runs continuously in background |
| **airflow-worker** | Orchestration | Executes individual tasks from DAGs | Same as webserver | Same as webserver | Same as webserver | None | N/A | Internal - executes your Python/Spark/SQL tasks |
| **broker** | Streaming | Kafka message broker for real-time data streams | N/A | `./kafka/data/` (persistence) | Same as input | `9092` | `localhost:9092` | Real-time data ingestion, event streaming, pub/sub |
| **schema-registry** | Streaming | Manages Kafka message schemas (Avro/Protobuf/JSON) | N/A | No persistent storage | No persistent storage | `8081` | `http://localhost:8081` | When using structured messages with schema validation |
| **kafka-ui** | Management UI | Visual interface for Kafka topics, messages, consumer groups | N/A | No persistent storage | No persistent storage | `8090` | `http://localhost:8090` | Monitor Kafka, view messages, create topics, check consumer lag |
| **python-dev** | Development | Jupyter Lab with PySpark for interactive development | `./dev_code/`<br>`./data/`<br>`./checkpoint/` | `./dev_code/` (notebooks/code)<br>`./data/` (input data)<br>`./checkpoint/` | `./dev_code/` (saved notebooks)<br>`./data/` (results)<br>`./checkpoint/` (state) | `8888`<br>`4040` | `http://localhost:8888?token=devtoken`<br>`http://localhost:4040` | Develop Spark jobs, test transformations, explore data, build ML features |

---

### 📂 Detailed Directory Reference

| Directory | Purpose | File Types | Read By | Written By | Backup Priority | Notes |
|-----------|---------|------------|---------|------------|-----------------|-------|
| `./postgres-airflow/data/` | Airflow metadata (database files) | PostgreSQL binary files | postgres-airflow | postgres-airflow | ⭐⭐⭐ CRITICAL - Airflow state | NEVER touch manually - use `pg_dump` for backup |
| `./postgres-app/data/` | Application data (database files) | PostgreSQL binary files | postgres-app | postgres-app | ⭐⭐⭐ CRITICAL - Business data | NEVER touch manually - use SQL for queries |
| `./pgadmin/` | pgAdmin settings & saved connections | JSON config files | pgadmin | pgadmin | ⭐ Low - can recreate | Share to give teammates your DB connections |
| `./airflow/dags/` | DAG definitions (your pipelines) | `.py` Python files | All Airflow services | YOU (code editor) | ⭐⭐⭐ CRITICAL - Your workflows | Hot-reloads automatically - changes appear in UI |
| `./airflow/logs/` | All Airflow logs | `.log` text files | YOU (debugging) | All Airflow services | ⭐⭐ Medium - debugging | Organized by `dag_id/task_id/date/` |
| `./airflow/plugins/` | Custom Airflow plugins | `.py` Python files | All Airflow services | YOU (code editor) | ⭐⭐ Medium - custom code | Restart webserver after adding |
| `./airflow/scripts/` | Helper utilities for DAGs | `.py`, `.sh` files | All Airflow services | YOU (code editor) | ⭐⭐ Medium - utilities | Import via `sys.path.append('/opt/airflow/scripts')` |
| `./dev_code/` | Jupyter notebooks and Python code | `.ipynb`, `.py` files | python-dev, Airflow services | YOU (Jupyter UI/code editor) | ⭐⭐⭐ CRITICAL - Development code | Access via Jupyter Lab at port 8888 |
| `./checkpoint/` | Spark streaming checkpoints | Binary checkpoint files | python-dev | python-dev | ⭐⭐ Medium - streaming state | CRITICAL for recovery - NEVER delete while streaming |
| `./data/` | Shared data files | `.csv`, `.parquet`, `.json`, `.txt` | python-dev, Airflow services | python-dev, YOU | ⭐⭐⭐ CRITICAL - processed data | Perfect for sample datasets and intermediate results |
| `./sql_scripts/` | SQL scripts for database operations | `.sql` files | pgadmin, postgres services | YOU | ⭐ Medium - reusable queries | Mounted at `/scripts` in pgadmin |
| `./postgres/backups/` | SQL backups from pgAdmin | `.sql`, `.backup` files | YOU (via pgAdmin) | pgadmin | ⭐⭐⭐ CRITICAL - database backups | Created via pgAdmin backup tool |
| `./kafka/data/` | Kafka topics and messages | Kafka binary logs | broker | broker | ⭐⭐⭐ CRITICAL - streaming data | Without this volume, ALL Kafka data is lost on restart |

---

### 📋 Complete Service Reference (Alternate View)

| Service | Code Location (Host) | Input Directory (Host → Container) | Output Directory (Container → Host) | Port (Host:Container) | Access URL | Purpose |
|---------|---------------------|------------------------------------|-------------------------------------|----------------------|------------|---------|
| **postgres-airflow** | N/A (database only) | `./postgres-airflow/data` → `/var/lib/postgresql/data` | Same as input (DB files written here) | `5432:5432` | `localhost:5432` | Airflow metadata storage |
| **postgres-app** | N/A (database only) | `./postgres-app/data` → `/var/lib/postgresql/data` | Same as input (DB files written here) | `5433:5432` | `localhost:5433` | Application data warehouse |
| **redis** | N/A (in-memory DB) | No persistent storage | No persistent storage | `6379:6379` | `localhost:6379` | Message broker for Celery |
| **pgadmin** | `./pgadmin/`<br>`./sql_scripts/`<br>`./data/`<br>`./postgres/backups/` | `./pgadmin` → `/var/lib/pgadmin` (settings)<br>`./sql_scripts` → `/scripts`<br>`./data` → `/data`<br>`./postgres/backups` → `/backups` | Same as input (settings saved)<br>`./postgres/backups` ← `/backups` | `5050:80` | `http://localhost:5050` | PostgreSQL management UI |
| **airflow-webserver** | `./airflow/dags/`<br>`./airflow/plugins/`<br>`./airflow/scripts/`<br>`./dev_code/`<br>`./data/` | `./airflow/dags` → `/opt/airflow/dags`<br>`./airflow/plugins` → `/opt/airflow/plugins`<br>`./airflow/scripts` → `/opt/airflow/scripts`<br>`./dev_code` → `/opt/airflow/dev_code`<br>`./data` → `/data` | `./airflow/logs` ← `/opt/airflow/logs` | `8080:8080` | `http://localhost:8080` | Airflow web UI & DAG parser |
| **airflow-scheduler** | Same as webserver | Same as webserver | Same as webserver | N/A | N/A | Schedules DAG runs |
| **airflow-worker** | Same as webserver | Same as webserver | Same as webserver | N/A | N/A | Executes task instances |
| **broker** | `./kafka/data/` | `./kafka/data` → `/var/lib/kafka/data` | Same as input (Kafka logs) | `9092:9092` | `localhost:9092` | Kafka message broker (KRaft mode) |
| **schema-registry** | N/A | No persistent storage | No persistent storage | `8081:8081` | `http://localhost:8081` | Kafka schema management |
| **kafka-ui** | N/A | No persistent storage | No persistent storage | `8090:8080` | `http://localhost:8090` | Kafka management UI |
| **python-dev** | `./dev_code/`<br>`./data/`<br>`./checkpoint/` | `./dev_code` → `/home/jovyan/work`<br>`./data` → `/home/jovyan/data`<br>`./checkpoint` → `/home/jovyan/checkpoint` | `./dev_code` ← `/home/jovyan/work`<br>`./data` ← `/home/jovyan/data`<br>`./checkpoint` ← `/home/jovyan/checkpoint` | `8888:8888`<br>`4040:4040` | `http://localhost:8888?token=devtoken`<br>`http://localhost:4040` | Jupyter Lab with PySpark |

---

### 📁 Summary of Key Folders

| Host Folder | Purpose | Used By | Access Inside Container | Key Notes |
|-------------|---------|---------|------------------------|-----------|
| `./postgres-airflow/data/` | Airflow metadata database files | postgres-airflow | `/var/lib/postgresql/data` | ⚠️ BINARY FILES - NEVER edit manually |
| `./postgres-app/data/` | Application database files (data warehouse) | postgres-app | `/var/lib/postgresql/data` | ⚠️ BINARY FILES - Query via SQL only |
| `./pgadmin/` | pgAdmin settings & saved connections | pgadmin | `/var/lib/pgadmin` | Share to give teammates your DB connections |
| `./airflow/dags/` | Airflow DAG definitions (your pipelines) | All Airflow services | `/opt/airflow/dags` | ✨ Hot-reload - changes appear automatically |
| `./airflow/logs/` | Airflow logs (scheduler, tasks, webserver) | All Airflow services | `/opt/airflow/logs` | Essential for debugging failed tasks |
| `./airflow/plugins/` | Custom Airflow plugins | All Airflow services | `/opt/airflow/plugins` | Restart webserver after adding |
| `./airflow/scripts/` | Helper scripts imported by DAGs | All Airflow services | `/opt/airflow/scripts` | Add to `sys.path` in DAGs |
| `./dev_code/` | Jupyter notebooks and Python code | python-dev, Airflow services | Jupyter: `/home/jovyan/work`<br>Airflow: `/opt/airflow/dev_code` | Access via Jupyter Lab at port 8888 |
| `./checkpoint/` | Spark streaming checkpoints | python-dev | `/home/jovyan/checkpoint` | ⚠️ CRITICAL for recovery - don't delete |
| `./data/` | Shared data files (CSV, Parquet, JSON) | python-dev, Airflow services, pgadmin | Jupyter: `/home/jovyan/data`<br>Airflow: `/data`<br>pgadmin: `/data` | Perfect for sample datasets |
| `./sql_scripts/` | SQL scripts for database operations | pgadmin, postgres services | pgadmin: `/scripts` | Mounted for easy SQL execution |
| `./postgres/backups/` | Database backups from pgAdmin | pgadmin | `/backups` | Created via pgAdmin backup tool |
| `./kafka/data/` | Kafka topics and messages | broker | `/var/lib/kafka/data` | ⚠️ Without this, ALL Kafka data is lost on restart |

---

## 🔧 Service Configuration

### Database Services

#### PostgreSQL for Airflow (`postgres-airflow`)
```yaml
# Purpose: Airflow metadata storage only
# DO NOT USE for application data!
- Host port: 5432
- Internal port: 5432
- Database: airflow
- Username: airflow
- Password: airflow
- Volume: ./postgres-airflow/data
```

**Critical Warning**: This database contains ALL Airflow state:
- DAG run history
- Task instances
- Connections, variables, and XComs
- User accounts and permissions

**NEVER store business data here** - it's for Airflow's internal use only!

#### PostgreSQL for Application (`postgres-app`)
```yaml
# Purpose: Your business data warehouse
# Use this for all application data
- Host port: 5433
- Internal port: 5432
- Database: app_data
- Username: app_user
- Password: app_password
- Volume: ./postgres-app/data
```

This is your data warehouse. Design your star schemas, fact tables, and dimensions here.

### Message Broker Services

#### Redis
```yaml
# Purpose: Celery message broker and result backend
- Host port: 6379
- Internal port: 6379
- No persistence (in-memory only)
- Used by: Airflow workers
```

Redis is ephemeral - if it restarts, queued tasks are lost. This is acceptable because:
- Airflow tracks task state in PostgreSQL
- Failed tasks can be retried
- For production, consider Redis with persistence

#### Kafka (`broker`)
```yaml
# Purpose: Event streaming platform
- Host port: 9092 (external)
- Internal port: 29092 (container-to-container)
- Controller port: 29093 (KRaft internal)
- Volume: ./kafka/data
- Mode: KRaft (no Zookeeper!)
```

**Key Kafka Configuration:**
- **KRaft Mode**: No Zookeeper dependency - simpler deployment
- **Dual Listeners**: 
  - `PLAINTEXT://broker:29092` for internal services
  - `PLAINTEXT_HOST://localhost:9092` for host connections
- **Single node** - for development only (production needs 3+ brokers)

### Airflow Orchestration Services

The Airflow setup uses CeleryExecutor for distributed task execution:

#### Airflow Webserver
- **UI**: http://localhost:8080
- **Purpose**: Web interface, DAG visualization, manual triggers
- **Features**: Auto-creates admin user on first start

#### Airflow Scheduler
- **Purpose**: Monitors DAGs and schedules tasks
- **Stateless**: Can be scaled horizontally

#### Airflow Worker
- **Purpose**: Executes scheduled tasks
- **Scalable**: Add more workers for parallel processing
- **Access**: Can read/write to all shared volumes

### Management UI Services

#### pgAdmin
- **URL**: http://localhost:5050
- **Email**: admin@example.com
- **Password**: admin
- **Purpose**: Visual PostgreSQL management
- **Volume**: ./pgadmin (saves server connections)
- **Additional mounts**: 
  - `./sql_scripts:/scripts` - for running SQL scripts
  - `./data:/data` - for importing CSV files
  - `./postgres/backups:/backups` - for database backups

To connect to databases from pgAdmin:
- **Airflow DB**: Host: `postgres-airflow`, Port: 5432
- **App DB**: Host: `postgres-app`, Port: 5432

#### Kafka UI
- **URL**: http://localhost:8090
- **Purpose**: Kafka cluster management
- **Features**: 
  - View topics and messages
  - Monitor consumer groups
  - Create/delete topics
  - View schema registry subjects

#### Schema Registry
- **URL**: http://localhost:8081 (API only)
- **Purpose**: Store and retrieve Avro/Protobuf schemas
- **Integration**: Used by Kafka producers/consumers

---

## 📂 Volume Structure

### Code & Configuration Volumes (INPUT)

These volumes contain code that services execute:

| Volume | Host Path | Container Path | Purpose |
|--------|-----------|----------------|---------|
| AIRFLOW_DAGS_VOLUME | ./airflow/dags | /opt/airflow/dags | Airflow DAG definitions |
| AIRFLOW_PLUGINS_VOLUME | ./airflow/plugins | /opt/airflow/plugins | Custom Airflow plugins |
| AIRFLOW_SCRIPTS_VOLUME | ./airflow/scripts | /opt/airflow/scripts | Helper Python scripts |
| DEV_CODE_VOLUME | ./dev_code | /opt/airflow/dev_code (Airflow)<br>/home/jovyan/work (Jupyter) | Shared development code |
| SQL_SCRIPTS_VOLUME | ./sql_scripts | /scripts (pgAdmin)<br>/scripts (postgres) | SQL scripts for database operations |

### Data Volumes (INPUT/OUTPUT)

These volumes store persistent data:

| Volume | Host Path | Container Path | Content Type |
|--------|-----------|----------------|--------------|
| POSTGRES_AIRFLOW_VOLUME | ./postgres-airflow/data | /var/lib/postgresql/data | Binary (PostgreSQL files) |
| POSTGRES_APP_VOLUME | ./postgres-app/data | /var/lib/postgresql/data | Binary (PostgreSQL files) |
| PGADMIN_VOLUME | ./pgadmin | /var/lib/pgadmin | Settings (JSON) |
| KAFKA DATA | ./kafka/data | /var/lib/kafka/data | Binary (Kafka logs) |
| CHECKPOINT_VOLUME | ./checkpoint | /home/jovyan/checkpoint | Spark streaming state |
| DATA_VOLUME | ./data | /home/jovyan/data (Jupyter)<br>/data (Airflow, pgAdmin) | CSV/Parquet files |
| AIRFLOW_LOGS_VOLUME | ./airflow/logs | /opt/airflow/logs | Text logs |
| BACKUPS_VOLUME | ./postgres/backups | /backups (pgAdmin) | SQL database backups |

### Special Volumes (Pip Cache)

```yaml
volumes:
  pip-cache:      # Docker-managed volume for pip downloads
```

These Docker-managed volumes persist Python packages across container restarts, so you don't have to reinstall every time.

---

## 🌐 Network & Communication

### Internal Network: `data-net`

All services communicate over the `data-net` bridge network using **container hostnames**:

| Service | Hostname | Internal URL |
|---------|----------|--------------|
| postgres-airflow | postgres-airflow | postgresql://airflow:airflow@postgres-airflow:5432/airflow |
| postgres-app | postgres-app | postgresql://app_user:app_password@postgres-app:5432/app_data |
| redis | redis | redis://redis:6379/0 |
| broker | broker | broker:29092 (Kafka) |
| schema-registry | schema-registry | http://schema-registry:8081 |

### Connection Examples

**From Airflow to Kafka:**
```python
# In your DAG code
KafkaConsumer(
    bootstrap_servers='broker:29092',  # NOT localhost:9092!
    ...
)
```

**From Jupyter to PostgreSQL:**
```python
# In your notebook
engine = create_engine('postgresql://app_user:app_password@postgres-app:5432/app_data')
```

**From Host to Services:**
- Always use `localhost` and the **host port**
- Example: Kafka on `localhost:9092`, PostgreSQL on `localhost:5433`

---

## 🧪 Development Environments

### Python Development (`python-dev`)

**Purpose**: Interactive data exploration and Spark development with Jupyter Lab

**Access**: http://localhost:8888 (token: devtoken)

**Pre-installed**:
- Spark 3.4.1 with Kafka integration
- Jupyter Lab interface
- Common Python packages (confluent-kafka, fastapi, uvicorn, pyarrow, streamlit, plotly, dash, pytest)

**Sample notebook**:
```python
# Read from Kafka
df = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "broker:29092") \
    .option("subscribe", "test-topic") \
    .load()

# Process and write to console
query = df.selectExpr("CAST(value AS STRING)") \
    .writeStream \
    .format("console") \
    .start()
```

**Automatic package installation**:
The container automatically installs packages from `COMMON_PYTHON_PACKAGES` in the .env file on startup.

---

## 📊 Monitoring & Management UIs

### Airflow UI (http://localhost:8080)

**Default credentials**: admin / admin

Key sections:
- **DAGs**: View and trigger your pipelines
- **Admin → Connections**: Set up external system connections
- **Admin → Variables**: Store configuration values
- **Browse → Task Instances**: Debug failed tasks

### pgAdmin (http://localhost:5050)

**Login**: admin@example.com / admin

To add database connections:
1. Right-click "Servers" → "Register" → "Server"
2. **Airflow DB**:
   - Name: Airflow Metadata
   - Host: postgres-airflow
   - Port: 5432
   - Database: airflow
   - Username: airflow
   - Password: airflow
3. **App DB**:
   - Name: Application Data
   - Host: postgres-app
   - Port: 5432
   - Database: app_data
   - Username: app_user
   - Password: app_password

**Additional Features**:
- Run SQL scripts from `/scripts` directory
- Import CSV files from `/data` directory
- Create backups to `/backups` directory

### Kafka UI (http://localhost:8090)

Features:
- **Topics**: Create, delete, browse messages
- **Consumer Groups**: Monitor offsets and lag
- **Schema Registry**: View registered schemas

### Jupyter Lab (http://localhost:8888)

**Token**: devtoken (from .env)

Navigate to `/work` to access your ./dev_code directory.

### Spark UI (http://localhost:4040)

Only appears when a Spark application is running. Monitor:
- Running jobs and stages
- Executor memory and tasks
- SQL query plans

---

## 💾 Data Persistence

### What Persists Across Restarts

| Data Type | Persists? | Location | Notes |
|-----------|-----------|----------|-------|
| PostgreSQL data | ✅ Yes | ./postgres-*/data | All database content |
| Kafka messages | ✅ Yes | ./kafka/data | Topics and offsets |
| Airflow logs | ✅ Yes | ./airflow/logs | Task and system logs |
| Airflow metadata | ✅ Yes | postgres-airflow | DAG runs, connections |
| Jupyter notebooks | ✅ Yes | ./dev_code | Your code |
| Python packages | ✅ Yes | Docker volumes | pip-cache |
| pgAdmin settings | ✅ Yes | ./pgadmin | Saved connections |
| Spark checkpoints | ✅ Yes | ./checkpoint | Streaming state |
| Database backups | ✅ Yes | ./postgres/backups | SQL dumps from pgAdmin |

### What's Ephemeral (Lost on Restart)

| Data Type | Reason |
|-----------|--------|
| Redis data | In-memory broker, designed for queues |
| Queued tasks | Will be retried by Airflow |
| Container logs | Use `docker logs` or volume mounts |
| Jupyter kernel state | Save notebooks to persist |

---

## 🔄 Service Dependencies

### Startup Order (with healthchecks)

```
postgres-airflow (healthy) ──┬──> redis (healthy) ──┬──> airflow-webserver
                              │                       │
postgres-app ─────────────────┘                       ├──> airflow-scheduler
                                                       │
broker ───────────────────────┬──> schema-registry ──┘
                              │
                              └──> kafka-ui
                                   python-dev
```

### Healthcheck Details

Each service has custom healthchecks:

- **PostgreSQL**: `pg_isready -U username`
- **Redis**: `redis-cli ping` (expects PONG)
- **Airflow**: No explicit healthcheck (but depends on others)

### Manual Dependency Checks

If a service fails to start, check its dependencies:

```bash
# Check if PostgreSQL is ready
docker exec postgres-airflow pg_isready -U airflow

# Check if Redis responds
docker exec redis redis-cli ping

# Check Kafka connectivity
docker exec broker kafka-topics --bootstrap-server broker:29092 --list
```

---

## 🔑 Service Credentials Reference Table

| Service | Username | Password / Token | Database | Port | Access URL/Connection |
|---------|----------|------------------|----------|------|----------------------|
| **postgres-airflow** | `airflow` | `airflow` | `airflow` | `5432` | `localhost:5432` (host)<br>`postgres-airflow:5432` (container) |
| **postgres-app** | `app_user` | `app_password` | `app_data` | `5433` | `localhost:5433` (host)<br>`postgres-app:5432` (container) |
| **pgadmin** | `admin@example.com` | `admin` | N/A | `5050` | `http://localhost:5050` |
| **airflow-webserver** | `admin` | `admin` | N/A | `8080` | `http://localhost:8080` |
| **redis** | No authentication | No authentication | N/A | `6379` | `localhost:6379` |
| **broker** (Kafka) | No authentication | No authentication | N/A | `9092` | `localhost:9092` |
| **schema-registry** | No authentication | No authentication | N/A | `8081` | `http://localhost:8081` |
| **kafka-ui** | No authentication | No authentication | N/A | `8090` | `http://localhost:8090` |
| **python-dev** (Jupyter) | `devtoken` (token) | N/A | N/A | `8888`<br>`4040` | `http://localhost:8888?token=devtoken`<br>`http://localhost:4040` (Spark UI) |

---

## 🔌 Connection Strings Reference

| Connection Type | Connection String / Configuration | When to Use |
|-----------------|-----------------------------------|-------------|
| **Airflow → postgres-airflow** | `postgresql+psycopg2://airflow:airflow@postgres-airflow:5432/airflow` | Airflow metadata connection (already configured) |
| **Application → postgres-app** (from container) | `postgresql://app_user:app_password@postgres-app:5432/app_data` | From python-dev or airflow-worker |
| **Application → postgres-app** (from host) | `postgresql://app_user:app_password@localhost:5433/app_data` | From your local machine (Python scripts, BI tools) |
| **Airflow → redis** | `redis://redis:6379/0` | Celery broker URL (already configured) |
| **Airflow → Kafka** | `broker:29092` | Internal connection from Airflow to Kafka |
| **Host → Kafka** | `localhost:9092` | External connection from your machine to Kafka |
| **schema-registry → Kafka** | `broker:29092` | Internal connection for schema registry |
| **kafka-ui → Kafka** | `broker:29092` | Internal connection for Kafka UI |
| **kafka-ui → schema-registry** | `http://schema-registry:8081` | Internal connection for schema registry UI |
| **python-dev → Kafka** | `broker:29092` | Spark streaming from Kafka inside container |
| **python-dev → postgres-app** | `jdbc:postgresql://postgres-app:5432/app_data` | JDBC connection for Spark |

---

## ⚠️ Important Notes

- All credentials shown are **default development values** - **CHANGE IN `.env` FOR PRODUCTION!**
- Redis, Kafka, and schema-registry have **no authentication** by default (development only - add SASL/SSL for production)
- Jupyter uses a **token** (`devtoken`) instead of username/password - pass in URL or Jupyter Lab login screen
- Internal services communicate via **container names** (e.g., `postgres-airflow`, not `localhost`)
- External access from your host machine uses **localhost** with the exposed ports
- The `pip-cache` Docker volume persists Python packages across container restarts

---

## 📝 Step-by-Step pgAdmin Setup

1. **Open pgAdmin**: http://localhost:5050

2. **Login**: 
   - Email: `admin@example.com`
   - Password: `admin`

3. **Add New Server for Airflow Metadata**:

   **General tab:**
   - Name: `Airflow Metadata` (or any name you prefer)

   **Connection tab:**
   - Host name/address: `postgres-airflow` ⚠️ **NOT localhost!**
   - Port: `5432`
   - Maintenance database: `postgres`
   - Username: `airflow`
   - Password: `airflow`
   - Save password? ✅ Yes

4. **Add New Server for Application Data**:

   **General tab:**
   - Name: `Application Data` (or any name you prefer)

   **Connection tab:**
   - Host name/address: `postgres-app` ⚠️ **NOT localhost!**
   - Port: `5432`
   - Maintenance database: `postgres`
   - Username: `app_user`
   - Password: `app_password`
   - Save password? ✅ Yes

5. **Save both servers** - You should now see both databases in the left sidebar

### Using pgAdmin with Mounted Directories

- **Run SQL scripts**: Navigate to Tools → Query Tool, then open files from `/scripts`
- **Import CSV**: Right-click a table → Import/Export → Use `/data/filename.csv`
- **Create backups**: Right-click a database → Backup → Save to `/backups`

---

## ❗ Common Mistakes to Avoid

**DON'T use `localhost` as the hostname in pgAdmin!** 

Since pgAdmin is running in a Docker container, `localhost` would refer to the pgAdmin container itself, not your host machine. This is a common source of "Connection refused" errors.

**Always use the Docker service names:**
- `postgres-airflow` - for the Airflow metadata database
- `postgres-app` - for your application database

These resolve correctly because all services are on the same Docker network (`data-net`).

**Other Common Mistakes:**
- Storing business data in `postgres-airflow`
- Committing `.env` to git
- Using `localhost` inside containers (use service names)
- Forgetting to create directories before starting

---

## 🚀 Quick Reference: Where to Put Your Code

| What You're Building | Put It Here | Access Via |
|---------------------|-------------|------------|
| **Airflow DAG** (production pipeline) | `./airflow/dags/` | Airflow UI @ port 8080 |
| **Jupyter notebook** (exploratory analysis) | `./dev_code/` | Jupyter @ port 8888 |
| **Spark job** (batch/streaming) | `./dev_code/` | Airflow or Jupyter |
| **Python script** (utility) | `./dev_code/` | Can be imported by Airflow DAGs |
| **FastAPI app** (REST API) | `./dev_code/` | Run with uvicorn inside python-dev |
| **Kafka producer** (test data) | `./dev_code/` | Run inside python-dev container |
| **Sample data** (CSV/Parquet) | `./data/` | Access from any container at `/data` |
| **SQL scripts** | `./sql_scripts/` | Run from pgAdmin at `/scripts` |
| **Custom Airflow plugin** | `./airflow/plugins/` | Restart webserver after adding |
| **Helper scripts for DAGs** | `./airflow/scripts/` | Import with `sys.path.append('/opt/airflow/scripts')` |

---

## 🐛 Troubleshooting Guide

### Common Issues and Solutions

#### 1. Port Already in Use

**Error**: `port is already allocated` or `address already in use`

**Solution**: 
```bash
# Find what's using the port
sudo lsof -i :8080  # Linux/Mac
netstat -ano | findstr :8080  # Windows

# Stop the conflicting service or change port in .env
```

#### 2. Docker Permission Denied

**Error**: `permission denied while trying to connect to the Docker daemon socket`

**Solution**:
```bash
# Add user to docker group (Linux)
sudo usermod -aG docker $USER
newgrp docker

# Or run commands with sudo
sudo docker-compose up
```

#### 3. Volume Permission Issues

**Error**: PostgreSQL or Kafka can't write to volumes

**Solution**:
```bash
# Fix permissions on host directories
chmod -R 777 postgres-*/data
chmod -R 777 kafka/data
```

#### 4. Airflow Can't Connect to PostgreSQL

**Error**: `psycopg2.OperationalError: could not connect to server`

**Check**:
```bash
# Verify PostgreSQL is healthy
docker-compose ps postgres-airflow

# Test connection from Airflow container
docker exec airflow-webserver python -c "
from sqlalchemy import create_engine
engine = create_engine('postgresql://airflow:airflow@postgres-airflow:5432/airflow')
connection = engine.connect()
print('Connection successful!')
"
```

#### 5. Kafka Connection Refused

**Error**: `Failed to connect to broker at broker:29092`

**Check**:
```bash
# Verify broker is running
docker-compose ps broker

# Check if advertised listeners are correct
docker exec broker cat /etc/kafka/server.properties | grep advertised

# Test from another container
docker exec airflow-worker kafka-topics --bootstrap-server broker:29092 --list
```

#### 6. Services Start but Exit Immediately

**Solution**:
```bash
# Check logs for the specific service
docker-compose logs <service-name>

# Common causes:
# - Missing directories (run the mkdir commands)
# - Environment variables not set (check .env file)
# - Insufficient memory (increase Docker memory limit)
```

### Debugging Commands

```bash
# View all logs
docker-compose logs -f

# View logs for specific service
docker-compose logs -f airflow-webserver

# Enter a container for debugging
docker exec -it airflow-webserver bash

# Check resource usage
docker stats

# Verify network connectivity
docker exec airflow-webserver ping postgres-airflow
docker exec airflow-webserver curl -I http://schema-registry:8081

# List all containers
docker-compose ps

# Restart a specific service
docker-compose restart airflow-worker
```

---

## 🔒 Security Considerations

### Default Credentials (CHANGE IN PRODUCTION!)

| Service | Username | Password | Change In Production? |
|---------|----------|----------|----------------------|
| Airflow | admin | admin | ⚠️ **MUST CHANGE** |
| pgAdmin | admin@example.com | admin | ⚠️ **MUST CHANGE** |
| PostgreSQL (airflow) | airflow | airflow | ⚠️ **MUST CHANGE** |
| PostgreSQL (app) | app_user | app_password | ⚠️ **MUST CHANGE** |
| Jupyter | (token) | devtoken | ⚠️ **MUST CHANGE** |

### Environment Variables (.env)

The `.env` file contains sensitive information:
- Database passwords
- Airflow Fernet key (encrypts connections/variables)
- API keys and tokens

**Best Practices**:
1. **Never commit .env to git** (add to .gitignore)
2. Use different passwords for development vs. production
3. Rotate secrets periodically
4. Consider using Docker secrets or HashiCorp Vault for production

### Network Security

- All services are on an isolated bridge network (`data-net`)
- Only necessary ports are exposed to host
- Internal communication uses plaintext (development only)
- For production, add TLS/SSL and authentication

### Volume Security

Host-mounted volumes have the same permissions as the host user. In production:
- Use Docker volumes instead of bind mounts
- Set proper ownership (UID/GID matching container users)
- Restrict access to sensitive directories

---

## 📈 Performance Optimization

### Resource Allocation

**Minimum Requirements**:
- 8GB RAM allocated to Docker
- 4 CPU cores
- 20GB free disk space

**Recommended**:
- 16GB RAM
- 8 CPU cores
- 50GB SSD storage

### Tuning Tips

#### PostgreSQL
```sql
-- Increase shared buffers for better performance
ALTER SYSTEM SET shared_buffers = '2GB';
ALTER SYSTEM SET effective_cache_size = '6GB';
ALTER SYSTEM SET work_mem = '32MB';
```

#### Kafka
```bash
# In broker environment or server.properties
KAFKA_NUM_PARTITIONS=3  # Default partitions
KAFKA_LOG_RETENTION_HOURS=168  # Keep data for 7 days
```

#### Airflow
```bash
# In airflow.cfg or environment variables
AIRFLOW__CORE__PARALLELISM=32
AIRFLOW__CORE__DAG_CONCURRENCY=16
AIRFLOW__SCHEDULER__SCHEDULER_HEARTBEAT_SEC=5
```

### Scaling Strategies

**Add more Airflow workers**:
```yaml
# In docker-compose.yml (add replica)
airflow-worker:
  deploy:
    replicas: 3
```

**Kafka cluster** (production):
```yaml
# Add multiple broker instances
broker-1: ...
broker-2: ...
broker-3: ...
```

---

## 🧩 Extending the Stack

### Adding New Services

To add a service (e.g., Trino, Dremio, MinIO):

```yaml
# Example: Adding MinIO (S3-compatible storage)
minio:
  image: minio/minio:latest
  container_name: minio
  ports:
    - "9000:9000"  # API
    - "9001:9001"  # Console
  environment:
    MINIO_ROOT_USER: minioadmin
    MINIO_ROOT_PASSWORD: minioadmin
  volumes:
    - ./minio/data:/data
  command: server /data --console-address ":9001"
  networks:
    - data-net
```

### Installing Additional Python Packages

**In Airflow**:
Add packages to `AIRFLOW_EXTRA_PIP_PACKAGES` in .env:
```bash
AIRFLOW_EXTRA_PIP_PACKAGES="apache-airflow-providers-apache-spark==4.1.1 pyspark==3.4.1 pandas requests beautifulsoup4"
```

**In Jupyter/Python-dev**:
Add packages to `COMMON_PYTHON_PACKAGES` in .env:
```bash
COMMON_PYTHON_PACKAGES="confluent-kafka==2.3.0 fastapi==0.104.1 uvicorn==0.24.0 pyarrow==10.0.1 streamlit==1.28.0 plotly==5.18.0 dash==2.14.0 pytest==7.4.0 new-package==1.0.0"
```

**Manual installation**:
```bash
# Access the container and pip install
docker exec -it python-dev pip install new-package
```

**Persistent packages**:
The `pip-cache` volume keeps installations across restarts.

### Custom Airflow Connections

Add via UI (Admin → Connections) or environment variables:
```bash
# In .env
AIRFLOW_CONN_MY_POSTGRES=postgresql://user:pass@postgres-app:5432/mydb
AIRFLOW_CONN_MY_KAFKA=kafka://broker:29092
```

### Adding Spark Packages

For PySpark, modify the `PYSPARK_SUBMIT_ARGS` in docker-compose for python-dev:
```yaml
environment:
  - PYSPARK_SUBMIT_ARGS=--packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.4.1,org.postgresql:postgresql:42.6.0 pyspark-shell
```

---

## 📚 Learning Resources

### Technology Documentation

- **Apache Airflow**: https://airflow.apache.org/docs/
- **Apache Kafka**: https://kafka.apache.org/documentation/
- **Apache Spark**: https://spark.apache.org/docs/latest/
- **PostgreSQL**: https://www.postgresql.org/docs/
- **Docker Compose**: https://docs.docker.com/compose/

### Sample Projects to Try

#### 1. Real-time Dashboard Pipeline
- Generate fake user events with a Python script in `./dev_code`
- Stream to Kafka
- Process with PySpark (aggregate per minute)
- Write to PostgreSQL
- Query from Jupyter

#### 2. Batch ETL with Airflow
- Place CSV files in `./data`
- Create Airflow DAG that:
  - Reads CSV with PythonOperator
  - Cleans data with Pandas
  - Loads to PostgreSQL (postgres-app)
  - Runs quality checks

#### 3. Kafka to PostgreSQL Sink
- Create Kafka topic
- Build PySpark streaming job in Jupyter
- Write to PostgreSQL with `foreachBatch`
- Monitor with Kafka UI

#### 4. Airflow Spark Integration
- Write PySpark job and save in `./dev_code`
- Create Airflow DAG with PythonOperator that runs the Spark job
- Pass dynamic dates as arguments
- Monitor in Spark UI

### Common Patterns

**PythonOperator with shared code**:
```python
from airflow import DAG
from airflow.operators.python import PythonOperator
import sys
sys.path.append('/opt/airflow/scripts')  # Your scripts volume
from my_helpers import process_data

def process_task():
    process_data('/opt/airflow/data/input.csv', '/opt/airflow/data/output/')

with DAG(...):
    t1 = PythonOperator(
        task_id='process_data',
        python_callable=process_task
    )
```

**Kafka to PostgreSQL streaming**:
```python
# In Jupyter or Spark job
df = spark \
  .readStream \
  .format("kafka") \
  .option("kafka.bootstrap.servers", "broker:29092") \
  .option("subscribe", "events") \
  .load()

def write_to_postgres(df, epoch_id):
    df.write \
      .format("jdbc") \
      .option("url", "jdbc:postgresql://postgres-app:5432/app_data") \
      .option("dbtable", "streaming_events") \
      .option("user", "app_user") \
      .option("password", "app_password") \
      .mode("append") \
      .save()

query = df.writeStream \
  .foreachBatch(write_to_postgres) \
  .outputMode("append") \
  .start()
```

---

## 📝 Final Notes

This Docker Compose setup is designed to be your **ultimate data engineering playground**. It balances:

- **Completeness**: All major data engineering tools in one stack
- **Reality**: Mirrors production patterns (separate metadata/data, distributed Airflow)
- **Usability**: Everything pre-configured to work together
- **Flexibility**: Easy to extend with new services
- **Persistence**: Your work survives restarts

### Best Practices Summary

✅ **DO**:
- Use `postgres-app` for all business data
- Put DAGs in `./airflow/dags/` (auto-reloads)
- Save notebooks in `./dev_code/`
- Use `broker:29092` for internal Kafka connections
- Create `.gitignore` with all data/volume paths

❌ **DON'T**:
- Store business data in `postgres-airflow`
- Commit `.env` to git
- Use `localhost` inside containers (use service names)
- Forget to create directories before starting

### Support & Contributing

If you encounter issues:
1. Check the [Troubleshooting Guide](#-troubleshooting-guide)
2. Search GitHub issues
3. Ask team members (we've probably seen it!)

To contribute improvements:
- Update this README
- Add example DAGs to `airflow/dags/examples/`
- Share useful generator scripts in `dev_code/examples/`
- Document workarounds for common issues

---

**Happy Data Engineering!** 🚀📊
