# 🐳 Multi-Tier Microservices Architecture with Docker & Docker Compose

[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
[![Docker Compose](https://img.shields.io/badge/Docker_Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://docs.docker.com/compose/)
[![Node.js](https://img.shields.io/badge/Node.js_20-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)](https://nodejs.org/)
[![Express.js](https://img.shields.io/badge/Express.js-000000?style=for-the-badge&logo=express&logoColor=white)](https://expressjs.com/)
[![Python](https://img.shields.io/badge/Python_3.11-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask_3.0-000000?style=for-the-badge&logo=flask&logoColor=white)](https://flask.palletsprojects.com/)
[![Linux](https://img.shields.io/badge/Alpine_Linux-0D597F?style=for-the-badge&logo=alpinelinux&logoColor=white)](https://alpinelinux.org/)

---

## 📌 Overview

This project demonstrates a production-grade, containerized multi-tier microservices application built with **Docker** and orchestrated with **Docker Compose**. 

It showcases key DevOps and full-stack engineering principles:
* **Microservices Decoupling**: Separation of presentation layer (Node.js/Express) and business logic layer (Python/Flask).
* **Container Orchestration**: Declarative multi-container deployment using Docker Compose.
* **Internal Service Discovery & DNS**: Inter-container communication resolved dynamically via Docker's embedded DNS on an isolated custom bridge network (`http://backend:5000`).
* **Optimized Container Builds**: Lightweight Alpine Linux-based images (`node:20-alpine` and `python:3.11-alpine`) for reduced attack surface and minimal image footprints.
* **Dependency & Lifecycle Management**: Automated startup sequencing ensuring backend services initialize before consumer services.

---

## 🏗️ Architecture & Communication Flow

```
                     ┌─────────────────────────────────────────┐
                     │               Host Machine              │
                     └────────────────────┬────────────────────┘
                                          │
                        Browser Request   │   Port 3000:3000
                                          ▼
                ┌───────────────────────────────────────────────────┐
                │ Docker Network: frontend-network (Bridge)        │
                │                                                   │
                │   ┌────────────────────┐                          │
                │   │  frontend Service  │ (Node.js 20 Alpine)      │
                │   │  Container: :3000  │                          │
                │   └─────────┬──────────┘                          │
                │             │                                     │
                │             │ HTTP POST                           │
                │             │ http://backend:5000/process         │
                │             ▼ (DNS Resolution via Docker Engine)  │
                │   ┌────────────────────┐                          │
                │   │  backend Service   │ (Python 3.11 Alpine)     │
                │   │  Container: :5000  │                          │
                │   └────────────────────┘                          │
                └───────────────────────────────────────────────────┘
```

### Request Lifecycle
1. **User Request**: The client opens `http://localhost:3000` to interact with the frontend UI.
2. **Frontend Ingestion**: Express processes user input from the HTML form.
3. **Internal API Call**: The frontend dispatches an asynchronous HTTP POST request using `Axios` to `http://backend:5000/process`.
4. **Service Discovery**: Docker's internal DNS resolver maps the hostname `backend` directly to the backend container's assigned internal IP.
5. **Payload Processing**: The Flask API ingests the JSON payload, computes the response, and returns it to Express.
6. **Client Render**: Express renders the formatted response back to the client.

---

## 🛠️ Tech Stack & Container Specifications

| Layer | Technology | Base Image | Port | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Frontend** | Node.js / Express | `node:20-alpine` | `3000` | Serves client assets and proxies requests to backend |
| **Backend** | Python / Flask | `python:3.11-alpine` | `5000` | Microservice processing application data |
| **Orchestrator** | Docker Compose | Specification v3+ | — | Multi-container networking, volume, and service management |
| **Networking** | Docker Bridge Network | Custom (`frontend-network`) | — | Isolated inter-container communication layer |

---

## 📁 Repository Structure

```text
.
├── backend/
│   ├── app.py              # Flask REST application & endpoint handlers
│   ├── Dockerfile          # Lean Python Alpine container definition
│   └── requirements.txt    # Python runtime dependencies (Flask 3.0.2)
├── frontend/
│   ├── app.js              # Express web server & Axios backend proxy
│   ├── Dockerfile          # Lean Node.js Alpine container definition
│   ├── package.json        # Node dependencies & metadata
│   ├── package-lock.json   # Deterministic lockfile
│   └── public/
│       └── index.html      # Frontend UI interface
├── .gitignore              # Ignored paths (node_modules, caches, venv)
├── docker-compose.yml      # Orchestration config (services, networks, ports)
└── README.md               # Project documentation
```

---

## 🚀 Quick Start Guide

### Prerequisites
Make sure you have installed on your machine:
* [Docker Desktop](https://www.docker.com/products/docker-desktop/) (includes Docker Engine and Docker Compose)
* [Git](https://git-scm.com/)

Verify your installation:
```bash
docker --version
docker compose version
```

### 1. Clone the Repository
```bash
git clone https://github.com/<your-username>/docker-compose-microservices.git
cd docker-compose-microservices
```

### 2. Launch the Application
Build and run the entire stack in detached mode:
```bash
docker compose up --build -d
```

> **Note**: The `--build` flag ensures Docker builds fresh images from the source `Dockerfile`s on initial start or code changes.

### 3. Verify Container Status
Check that both services are healthy and running:
```bash
docker compose ps
```

You should see output similar to:
```text
NAME                        IMAGE                 COMMAND                  SERVICE    STATUS    PORTS
docker_compose-backend-1    docker_compose-backend    "python app.py"          backend    running   0.0.0.0:5000->5000/tcp
docker_compose-frontend-1   docker_compose-frontend   "docker-entrypoint.s…"   frontend   running   0.0.0.0:3000->3000/tcp
```

### 4. Access the Application
* **Frontend Web UI**: [http://localhost:3000](http://localhost:3000)
* **Backend Direct Healthcheck**: [http://localhost:5000](http://localhost:5000)

---

## 🔌 API Endpoints Reference

### Backend Microservice (`http://localhost:5000`)

| Method | Endpoint | Description | Request Body | Response |
| :--- | :--- | :--- | :--- | :--- |
| `GET` | `/` | Health check route | None | `Hello, World from Flaskk!` |
| `POST` | `/process` | Processes received payload | `{"name": "string"}` | `Hello, <name> from Flask!` |

### Frontend Service (`http://localhost:3000`)

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `GET` | `/` | Serves client web form (`index.html`) |
| `POST` | `/submit` | Accepts form input, queries backend via Docker network, renders response |

---

## 🔍 DevOps & Container Engineering Highlights

### 1. Zero-Configuration Service Discovery
Instead of hardcoding brittle IP addresses, services communicate using service names defined in `docker-compose.yml`:
```javascript
// frontend/app.js
const response = await axios.post('http://backend:5000/process', req.body);
```
Docker's embedded DNS automatically resolves `backend` to the private IP assigned to the backend container within `frontend-network`.

### 2. Isolated Custom Bridge Network
Default bridge networks share access across arbitrary containers. This project defines a dedicated bridge network (`frontend-network`), ensuring network-level isolation and strict security boundaries.

### 3. Minimal Footprint via Alpine Linux
Both images use official **Alpine Linux** distributions:
* **Backend**: `python:3.11-alpine` (~50MB compressed base vs. ~300MB Debian base)
* **Frontend**: `node:20-alpine` (~40MB compressed base vs. ~350MB standard base)

This minimizes deployment latency, reduces disk usage, and hardens the container attack surface by stripping non-essential binaries.

### 4. Deterministic Dependency Caching
Both Dockerfiles copy package dependency manifests (`requirements.txt`, `package*.json`) before source code. This leverages Docker's layer caching mechanism: code edits do not trigger unnecessary dependency re-installations.

---

## 📋 Common Operations & Useful Commands

| Task | Command |
| :--- | :--- |
| **Stream Live Logs** | `docker compose logs -f` |
| **Stream Backend Logs Only** | `docker compose logs -f backend` |
| **Stop Containers** | `docker compose stop` |
| **Tear Down Stack & Networks** | `docker compose down` |
| **Rebuild Without Cache** | `docker compose build --no-cache` |
| **Execute Shell in Container** | `docker compose exec backend sh` |

---

## 🔮 Roadmap & Production Enhancements

- [ ] **Reverse Proxy**: Add Nginx container as an ingress controller for SSL/TLS termination and load balancing.
- [ ] **Data Persistence**: Integrate a managed database service (e.g., PostgreSQL or Redis) with named Docker volumes.
- [ ] **Healthchecks**: Implement native Docker `HEALTHCHECK` instructions for self-healing container lifecycles.
- [ ] **CI/CD Pipeline**: GitHub Actions workflow for automated image scanning (Trivy), linting, and Docker Hub registry push.
- [ ] **Environment Configuration**: Decouple environment variables using `.env` files for 12-factor compliance.

---

## 👨‍💻 Author

* **Developer**: *Your Name*
* **GitHub**: [@yourusername](https://github.com/yourusername)
* **LinkedIn**: [linkedin.com/in/yourprofile](https://linkedin.com/in/yourprofile)
* **Portfolio**: [yourportfolio.com](https://yourportfolio.com)

---

## 📄 License

This project is licensed under the [ISC License](LICENSE) - feel free to use, modify, and distribute for personal and professional projects.
#   2 - t i e r - d o c k e r - c o m p o s e - a p p  
 