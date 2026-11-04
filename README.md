# VulnGuard AI

VulnGuard is a single-organization vulnerability-management MVP for authorized assessment data. It provides inventory, findings, transparent local prioritization, remediation workflow, CSV reports, user roles, and audit history. It does not actively scan systems.

## Current capabilities

- Local SQLite development or PostgreSQL via `DATABASE_URL`.
- CSV preview and explicit import commit; records are saved and reimported by stable asset/CVE/software identity.
- Overview, assets, findings/evidence, intelligence source status, remediation, reports, administration, and ML readiness screens.
- Administrator, analyst, remediation owner, and manager roles. Remediation owners can update their assigned tasks; analysts/admins verify closure.
- Argon2 password hashes, signed HttpOnly sessions, eight-hour expiry, CSRF checks, login throttling, one-time bootstrap, and admin account controls.
- Container deployment package with PostgreSQL, Caddy HTTPS reverse proxy, and a PostgreSQL backup script.

Public NVD/KEV enrichment, scanner APIs, multi-tenant hosting, production monitoring, and ML predictions are not enabled. The local priority is CVSS base × 10; it is not an industry-standard score or probability of compromise.

## Local development

Requirements: Python 3.11+ and Node.js 20+.

### Backend

From the repository root, in PowerShell:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r backend\requirements.txt
$bytes = New-Object byte[] 32
$rng = [System.Security.Cryptography.RandomNumberGenerator]::Create()
$rng.GetBytes($bytes)
$env:VULNGUARD_BOOTSTRAP_TOKEN = [System.BitConverter]::ToString($bytes).Replace('-', '')
python -m uvicorn backend.app.main:app --reload --host 127.0.0.1 --port 8000
```

Keep that terminal open. The bootstrap token is in `$env:VULNGUARD_BOOTSTRAP_TOKEN`; enter it once in VulnGuard to create the first administrator. If you omit the environment variable, the API prints a generated development token when it starts with no users.

### Frontend

In a second PowerShell window:

```powershell
cd frontend
npm install
npm run dev -- --host 127.0.0.1
```

Open the Vite URL, usually `http://127.0.0.1:5173`. The Vite development proxy forwards `/api` requests to the local backend. API documentation is at `http://127.0.0.1:8000/docs`.

The local database is `vulnguard.db` in the repository root; set `VULNGUARD_DB` to store it elsewhere. Local service binds to loopback. Do not change that for team access; use the prepared deployment stack after reviewing [team deployment](docs/team-deployment.md).

## Team deployment package

Review [docs/team-deployment.md](docs/team-deployment.md). It covers environment setup, Docker Compose, public-domain or internal-CA HTTPS options, first administrator setup, roles, and backup/restore operations. The shared stack is not running from this workstation; Docker/PostgreSQL hosting and company DNS/certificate choices must be supplied by the system owner.

## Import format

The supported CSV schema is documented in [docs/import-format.md](docs/import-format.md). The included sample is fictional.
