# DocuIntel | Enterprise AI Document Intelligence Platform

DocuIntel is a production-grade, secure, on-premise Document Intelligence workspace. It performs OCR parsing, document classification, RAG chat, structured data extraction, risk audits, and side-by-side agreement comparisons entirely inside your organization's infrastructure using local LLMs (Ollama/vLLM) and ChromaDB.

---

## Workspace Directory Structure

```
docu-intel/
├── backend/
│   ├── app/
│   │   ├── config.py          # Unified system configuration
│   │   ├── database.py        # SQLAlchemy connections
│   │   ├── models/            # SQLAlchemy DB tables & Pydantic schemas
│   │   ├── services/          # Document chunking & RAG vector client
│   │   ├── agents/            # Specialized agents (Analysis, Contract, Search, Extraction)
│   │   └── api/               # API route endpoints
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── app/               # Next.js 15 pages
│   │   └── globals.css        # Tailwind style variables & glassmorphism
│   ├── package.json
│   └── Dockerfile
├── docker-compose.yml
├── k8s/                       # Kubernetes manifests (db, backend, frontend)
└── README.md
```

---

## Configuration & Microsoft Entra ID Integration

To run in **Production Mode**, configure the environment variables in `backend/app/config.py` or pass them through Docker/Kubernetes config maps.

### Microsoft Entra ID Setup
1. Register an application in the **Microsoft Entra admin center**.
2. Set the redirect URI to your frontend port: `http://localhost:3000` (or your Ingress domain).
3. Under **Token configuration**, add the `preferred_username`, `email`, and custom security roles.
4. Set the following environment variables on the backend:
   ```bash
   ENTRA_CLIENT_ID="your-azure-ad-client-id"
   ENTRA_TENANT_ID="your-azure-ad-tenant-id"
   ```

---

## Running the Application

### 1. Local Development Run
The code imports itself as `backend.app`, so the backend runs from the repository root:
```bash
pip install -r backend/requirements.txt
python -m uvicorn backend.app.main:app --reload --port 8081
```

**Boot the frontend** (Next.js dev server):
```bash
cd frontend
npm ci
npm run dev
```
Open `http://localhost:3000` in your web browser.

### 2. Multi-Container Orchestration (Docker-Compose)
PostgreSQL, Redis, ChromaDB, the FastAPI backend and the frontend. `SECRET_KEY` must be set (in `.env`):
```bash
docker compose up --build
```
The frontend is the static export (`npm run build` writes `frontend/out/`) served by nginx on
`http://localhost:3001`; the API is on `http://localhost:8081/api`. nginx reads `ai_config.json` from
`frontend/public/`, where the backend writes it when an admin saves settings.

### Tests
```bash
pip install -r backend/requirements-dev.txt
python -m pytest backend/app/tests -q          # backend, from the repository root

cd frontend
npm run typecheck && npm run lint && npm test  # TypeScript, ESLint, Vitest unit tests
npm run test:e2e                               # Playwright against the running stack (frontend/tests/e2e/README.md)
```
GitHub Actions (`.github/workflows/ci.yml`) runs all of these on every pull request and push to `main`;
the end-to-end job starts the Docker stack on a fresh database with throwaway secrets.

### 3. Deploying to Kubernetes (Private Cloud)
```bash
kubectl apply -f k8s/database.yaml
kubectl apply -f k8s/backend.yaml
kubectl apply -f k8s/frontend.yaml
```
Traffic is exposed on host `http://docu-intel.local`.
"# test" 
"# test" 
