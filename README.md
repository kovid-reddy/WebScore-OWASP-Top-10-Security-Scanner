<!--Banner -->
<p align="center">
  <img src="demo-assets/banner.png" alt="WebScore Banner" width="100%">
</p>

<!-- Badges -->
<p align="center">

<img src="https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB"/>

<img src="https://img.shields.io/badge/Fastify-000000?style=for-the-badge&logo=fastify&logoColor=white"/>

<img src="https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white"/>

<img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white"/>

<img src="https://img.shields.io/badge/Prisma-2D3748?style=for-the-badge&logo=prisma&logoColor=white"/>

<img src="https://img.shields.io/badge/OWASP%20Top%2010-Security-critical?style=for-the-badge&color=8A2BE2"/>

<!-- intro -->
</p>
<p align="center">
**Analyze • Detect • Score • Secure**
</p>

## 📖 Overview

**WebScore** is a full-stack web application vulnerability scanner inspired by the **OWASP Top 10**.

It automatically analyzes websites, identifies security vulnerabilities, calculates an overall **security score**, and provides detailed remediation guidance to help developers build more secure applications.

Built with a scalable **Turborepo monorepo architecture**, WebScore separates the frontend, API, and scanning engine into independent services for improved maintainability and scalability.

## 📸 Dashboard Preview

<p align="center">
<img src="demo-assets/completed_scan_results_1783526772579.png" width="95%" alt="WebScore Dashboard">
</p>

<p align="center">
<i>WebScore dashboard showing scan configuration, real-time progress, vulnerability findings and overall security score.</i>
</p>

## 🚀 Why WebScore?

WebScore isn't just another vulnerability scanner. It combines automated vulnerability detection, security scoring, and remediation guidance into a single platform to help developers identify and prioritize security risks efficiently.

### Highlights

- 🛡️ OWASP Top 10 inspired vulnerability detection
- 📊 Dynamic security scoring system
- ⚡ Background asynchronous scanning
- 🌐 Automatic endpoint crawling
- 📄 Actionable remediation recommendations
- 🏗️ Modular Turborepo architecture

<!-- ol md

# 🛡️ WebScore: OWASP Top 10 Security Scanner

WebScore is a monorepo-based web vulnerability scanner and assessment platform. It executes passive and active security checks against target websites and APIs, computes a risk-adjusted security score (from 0 to 100), and provides actionable remediation steps for detected issues.
-->
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
## 🔍 Security Checks Registry

### 🛡 Authentication

| Check ID | Level | Detection Strategy |
| :--- | :---: | :--- |
| `cookies` | 🌐 Site | Inspects cookies for missing **Secure**, **HttpOnly**, **SameSite** flags and validates security headers such as **HSTS**, **CSP**, and **X-Content-Type-Options**. |
| `jwt` | 🌐 Site | Tests JWT implementations for weak secrets, **none** algorithm abuse, and signature verification bypasses. |
| `csrf` | 🌐 Site | Detects missing CSRF tokens and insufficient protection for state-changing requests. |
| `ssrf` | 🌐 Site | Checks whether server-side requests can be redirected to internal networks or cloud metadata endpoints. |

---

### 🔓 Broken Access Control

| Check ID | Level | Detection Strategy |
| :--- | :---: | :--- |
| `cors` | 🌐 Site | Detects wildcard origins, origin reflection, and credentialed cross-origin requests. |
| `forced-browsing` | 🌐 Site | Discovers hidden files, backups, Git directories, admin panels, and exposed resources. |
| `http-method-abuse` | 🌐 Site | Tests support for unexpected HTTP methods such as **PUT**, **DELETE**, and **TRACE**. |
| `idor` | 🎯 Endpoint | Manipulates predictable identifiers to detect Insecure Direct Object Reference vulnerabilities. |

---

### 💉 Injection

| Check ID | Level | Detection Strategy |
| :--- | :---: | :--- |
| `sqli` | 🎯 Endpoint | Error-based • Boolean-blind • Time-based SQL Injection detection. |
| `xss` | 🎯 Endpoint | Reflected • Stored • DOM-based Cross-Site Scripting detection. |
| `ssti` | 🎯 Endpoint | Tests common template engines using standard SSTI payloads. |
| `os-command` | 🎯 Endpoint | Attempts OS command execution using Linux and Windows payloads. |
| `file-upload` | 🎯 Endpoint | Detects unrestricted file upload vulnerabilities and MIME bypasses. |
| `xxe` | 🎯 Endpoint | Tests XML parsers for XXE, SSRF, and local file disclosure. |
<!--
<table>
<thead>
<tr>
<th>Group</th>
<th>Check ID</th>
<th>Category</th>
<th>Level</th>
<th>Detection Strategy</th>
</tr>
</thead>

<tbody>

<tr>
<td><b>Auth</b></td>
<td><code>cookies</code></td>
<td>Authentication</td>
<td>Site</td>
<td>
Inspects HTTP response headers for missing or insecure cookie flags
(<code>Secure</code>, <code>HttpOnly</code>, <code>SameSite</code>)
and validates security headers like
<code>HSTS</code>,
<code>CSP</code>,
<code>X-Content-Type-Options</code>.
</td>
</tr>

<tr>
<td></td>
<td><code>jwt</code></td>
<td>Authentication</td>
<td>Site</td>
<td>
Tests JWT implementations for weak secrets,
<code>none</code> algorithm abuse,
signature verification bypasses,
and insecure token validation.
</td>
</tr>

<tr>
<td></td>
<td><code>csrf</code></td>
<td>Authentication</td>
<td>Site</td>
<td>
Detects missing CSRF tokens,
Origin/Referer validation,
and state-changing endpoints without CSRF protection.
</td>
</tr>

<tr>
<td></td>
<td><code>ssrf</code></td>
<td>Authentication</td>
<td>Site</td>
<td>
Attempts SSRF payloads to determine whether internal resources,
localhost,
or cloud metadata endpoints are reachable.
</td>
</tr>

<tr>
<td><b>Access Control</b></td>
<td><code>cors</code></td>
<td>Broken Access Control</td>
<td>Site</td>
<td>
Tests for wildcard origins,
origin reflection,
and credentialed cross-origin requests.
</td>
</tr>

<tr>
<td></td>
<td><code>forced-browsing</code></td>
<td>Broken Access Control</td>
<td>Site</td>
<td>
Discovers hidden files,
backups,
Git repositories,
admin panels,
configuration files,
and exposed resources using optimized wordlists.
</td>
</tr>

<tr>
<td></td>
<td><code>http-method-abuse</code></td>
<td>Broken Access Control</td>
<td>Site</td>
<td>
Checks support for unexpected HTTP methods
(<code>PUT</code>, <code>DELETE</code>, <code>TRACE</code>, etc.)
that may bypass authorization.
</td>
</tr>

<tr>
<td></td>
<td><code>idor</code></td>
<td>Broken Access Control</td>
<td>Endpoint</td>
<td>
Manipulates numeric and predictable identifiers to detect
Insecure Direct Object Reference vulnerabilities.
</td>
</tr>

<tr>
<td><b>Injection</b></td>
<td><code>sqli</code></td>
<td>Injection</td>
<td>Endpoint</td>
<td>

• Error-based SQL Injection<br>
• Boolean-blind SQL Injection<br>
• Time-based SQL Injection

</td>
</tr>

<tr>
<td></td>
<td><code>xss</code></td>
<td>Injection</td>
<td>Endpoint</td>
<td>

• Reflected XSS<br>
• Stored XSS<br>
• DOM-based XSS

</td>
</tr>

<tr>
<td></td>
<td><code>ssti</code></td>
<td>Injection</td>
<td>Endpoint</td>
<td>
Tests common template engines using payloads like
<code>{{7*7}}</code>,
<code>${7*7}</code>,
and equivalent engine-specific payloads.
</td>
</tr>

<tr>
<td></td>
<td><code>os-command</code></td>
<td>Injection</td>
<td>Endpoint</td>
<td>
Attempts command injection using payloads targeting both
Linux and Windows environments and validates reflected output.
</td>
</tr>

<tr>
<td></td>
<td><code>file-upload</code></td>
<td>Injection</td>
<td>Endpoint</td>
<td>
Tests upload functionality for unrestricted file extensions,
MIME validation bypasses,
and executable upload vulnerabilities.
</td>
</tr>

<tr>
<td></td>
<td><code>xxe</code></td>
<td>Injection</td>
<td>Endpoint</td>
<td>
Injects XML External Entity payloads to detect
internal file disclosure,
SSRF,
and parser misconfigurations.
</td>
</tr>

</tbody>
</table>
--
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
-->
---

## 🎥 WebScore in Action

<p align="center">
  <img src="./demo-assets/run_demo_scan_1783526726800.webp" width="95%">
</p>

<p align="center">
  <em>Scanning a target website and generating a complete security report.</em>
</p>

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

# ⚙️ Tech Stack

<div align="center">

| Layer | Technologies |
|:------|:-------------|
| 🎨 **Frontend** | React • TypeScript • Tailwind CSS |
| ⚙️ **Backend API** | Fastify • Node.js |
| 🔍 **Scanner Engine** | Node.js • Axios • Puppeteer |
| 🗄️ **Database** | PostgreSQL • Prisma ORM |
| 📬 **Queue System** | BullMQ • Redis |
| ☁️ **Deployment** | Docker • Render |
| 🏗️ **Architecture** | Turborepo Monorepo |

</div>

<p align="center">
<img src="https://skillicons.dev/icons?i=react,ts,nodejs,fastify,postgres,docker,redis,git"/>
</p>

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
> **WebScore** is intended solely for authorized security testing, research, and educational purposes.
>
> Only scan systems, applications, or APIs that you own or have explicit written permission to assess. Unauthorized security testing may violate applicable laws and regulations.
>
> The authors and contributors of WebScore are **not responsible** for any misuse, unauthorized activity, or damages resulting from the use of this software.

> 💙 Please use WebScore responsibly and help make the web a safer place.

## 📜 License

This project is currently **not licensed for public reuse or redistribution**.

All rights reserved © Kovid Reddy Kontham.
