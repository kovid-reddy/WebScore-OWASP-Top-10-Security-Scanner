# 🛡️ WebScore: OWASP Top 10 Security Scanner

WebScore is a monorepo-based web vulnerability scanner and assessment platform. It executes passive and active security checks against target websites and APIs, computes a risk-adjusted security score (from 0 to 100), and provides actionable remediation steps for detected issues.

---

## 🏗️ Architecture Overview

The project is structured as a TypeScript monorepo managed with **pnpm Workspaces** and **Turborepo**:

```mermaid
graph TD
    Web[apps/web<br>Next.js Frontend] -->|REST API calls| API[apps/api<br>Fastify Backend]
    API -->|Saves Scan Results| DB[(PostgreSQL Database)]
    API -->|Pushes Jobs| Queue[BullMQ / Redis]
    Queue -->|Pulls Jobs| Worker[apps/api<br>Background Worker]
    Worker -->|Dynamic Import| Scanner[apps/scanner<br>Security Engine]
    Scanner -->|Probes / Crawls| Target[Target Site / API]
```

- **`apps/web`**: Next.js (React 19, Tailwind CSS v4, Recharts) frontend client. Allows starting scans, viewing progress, displaying results, showing score gauges, and exporting scan results as JSON.
- **`apps/api`**: Fastify REST API and background task processor using **Prisma** (PostgreSQL) and **BullMQ** (Redis). Contains:
  - An API server (`src/server.ts`) for starting scans and fetching statuses.
  - A worker (`src/queue/scan.worker.ts`) to execute scans asynchronously.
- **`apps/scanner`**: The core security engine. Crawls pages, extracts forms and endpoints, and executes security checkers against target endpoints.
- **`packages/shared-types`**: Shared TypeScript types ensuring consistency between the frontend, backend, and security engine.

---

## 🔍 Security Engine Capabilities

The scanner runs in two phases:
1. **Site-wide Checks**: Executes once against the target's root URL.
2. **Endpoint-specific Checks**: Runs a localized crawl (up to 40 pages, max depth 3) extracting `a[href]` links, forms, and API endpoint references from JavaScript (`fetch`/`axios`). The scanner then runs the selected checks against all discovered endpoints in batches of 5.

### Check Registry

| Group | Check ID | Category | Check Level | Detection Strategy / Description |
| :--- | :--- | :--- | :--- | :--- |
| **Auth** | `cookies` | Auth | Site | Inspects HTTP response headers for missing or insecure cookie flags (`Secure`, `HttpOnly`, `SameSite`) and missing security headers (`HSTS`, `CSP`, `X-Content-Type-Options`). |
| | `jwt` | Auth | Site | Tests JWT endpoints for common signature bypasses (`none` algorithm, weak HS256 secrets, lack of signature validation). |
| | `csrf` | Auth | Site | Checks if state-changing requests or form inputs lack CSRF tokens or header protection. |
| | `ssrf` | Auth | Site | Evaluates if server-side redirects or requests can be manipulated to hit internal IP ranges. |
| **Access Control** | `cors` | Broken Access Control | Site | Sends requests with an attacker-controlled origin header to detect wildcard `*` configs and origin reflection combined with credentials. |
| | `forced-browsing` | Broken Access Control | Site | Scans for sensitive configuration, administrative panels, logs, backups, and Git directories using an optimized dictionary. |
| | `http-method-abuse` | Broken Access Control | Site | Probes endpoints with arbitrary HTTP methods (like `DELETE`, `PUT`, `TRACE`) to check for lax access configurations. |
| | `idor` | Broken Access Control | Endpoint | Probs endpoint parameters (like `/users/123`) to determine if resources belonging to other accounts can be enumerated. |
| **Injection** | `sqli` | Injection | Endpoint | Checks for: <ul><li>**Error-based SQLi** (triggers SQL syntax errors in GET/POST fields).</li><li>**Boolean-blind SQLi** (analyzes content length/status differences on true/false conditions).</li><li>**Time-based SQLi** (injects `SLEEP` functions and monitors response delays).</li></ul> |
| | `xss` | Injection | Endpoint | Detects reflected and DOM-based Cross-Site Scripting by verifying if injected scripts reflect in the parsed HTML without escaping. |
| | `ssti` | Injection | Endpoint | Checks for Server-Side Template Injection using standard template payloads (`{{7*7}}`, `${7*7}`). |
| | `os-command` | Injection | Endpoint | Injects OS commands (`whoami`, `cat /etc/passwd`, `ipconfig`) and checks for command output reflections in the response. |
| | `file-upload` | Injection | Endpoint | Tests upload endpoints for missing file extension filtering or execution capabilities. |
| | `xxe` | Injection | Endpoint | Probes XML endpoints with External Entity payloads to test for internal file disclosures or SSRF. |

---

## 🧮 Scoring System (WebScore)

The security score (from **0 to 100**) indicates the security posture of the target web application. It starts at `100` and subtracts penalties based on the severity of unique findings discovered:

- 🛑 **CRITICAL** (e.g., SQLi, Command Injection) $\rightarrow$ **-40 points**
- 🟠 **HIGH** (e.g., CORS with credentials reflection, XXE) $\rightarrow$ **-20 points**
- 🟡 **MEDIUM** (e.g., CORS wildcard, CSRF) $\rightarrow$ **-10 points**
- 🔵 **LOW** (e.g., insecure cookies) $\rightarrow$ **-5 points**
- ⚪ **INFO** (e.g., server headers, target unreachable) $\rightarrow$ **-0 points**

**Rating Ranges:**
*   🟩 **80 - 100**: Good security posture (low/no critical vulnerabilities).
*   🟨 **50 - 79**: Moderate risks (needs attention).
*   🟥 **0 - 49**: Poor security posture (critical vulnerabilities present).

---

## 🚀 Getting Started

### Prerequisites
Make sure you have the following installed:
- [Node.js](https://nodejs.org/) (v20+ recommended)
- [pnpm](https://pnpm.io/) (v9+)
- [PostgreSQL](https://www.postgresql.org/) (database store)
- [Redis](https://redis.io/) (used by BullMQ)

### 1. Install Dependencies
In the root directory, run:
```bash
pnpm install
```

### 2. Configure Environment Variables
Create a `.env` file in `apps/api/`:
```env
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/vulnscanner"
REDIS_URL="redis://localhost:6379"
FRONTEND_URL="http://localhost:3000"
PORT=4000
```

Create a `.env.local` file in `apps/web/`:
```env
NEXT_PUBLIC_API_URL=http://localhost:4000
```

### 3. Initialize the Database
Generate the Prisma client and apply migrations:
```bash
# Run from the root directory
pnpm --filter api db:generate
pnpm --filter api db:migrate
```

### 4. Run Development Servers
Start all workspaces in development mode:
```bash
pnpm dev
```
Alternatively, you can run individual workspaces:
```bash
# Start API & background worker
pnpm dev:api

# Start Next.js frontend
pnpm dev:web
```

Open [http://localhost:3000](http://localhost:3000) to access the dashboard.

---

## 🐳 Docker Deployment

You can build and deploy the services using the provided Docker configurations:

-   **API Server**: `apps/api/Dockerfile`
-   **Queue Worker**: `apps/api/Dockerfile.worker`

Build command example:
```bash
docker build -t vuln-scanner-api -f apps/api/Dockerfile .
docker build -t vuln-scanner-worker -f apps/api/Dockerfile.worker .
```

---

## 📝 Configuration (Render)

The project includes a `render.yaml` blueprint for easy deployment on Render:
-   **Web Service**: `vuln-scanner-api`
-   **Build Command**: `pnpm install && cd apps/api && npx prisma generate`
-   **Start Command**: `cd apps/api && npx prisma migrate deploy && npx tsx src/server.ts`

---

## ⚖️ Legal Disclaimer

> [!WARNING]
> This tool is designed for security assessment and testing purposes only. You must **only** scan targets that you own or have explicit, written permission to test. Unauthorized scanning of third-party websites or APIs is illegal and a violation of computer fraud laws. The developers of WebScore assume no liability for misuse of this tool.
