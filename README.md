# DKHCBC UMS — University Management System

**Debre Kidan Holy Cross Bible College · ደብረ ቅዳን ቅዱስ መስቀል መጽሐፍ ቅዱስ ኮሌጅ**

Enterprise-grade University Management System with full Ethiopian localization — bilingual (English + Amharic), Ethiopian Calendar support, and ETB currency.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [System Modules](#system-modules)
3. [Tech Stack](#tech-stack)
4. [Quick Start (Docker)](#quick-start-docker)
5. [Local Development](#local-development)
6. [Database Migrations](#database-migrations)
7. [Default Credentials](#default-credentials)
8. [API Documentation](#api-documentation)
9. [Kubernetes Deployment](#kubernetes-deployment)
10. [CI/CD Pipeline](#cicd-pipeline)
11. [Testing](#testing)
12. [Ethiopian Localization](#ethiopian-localization)
13. [Security](#security)
14. [Monitoring](#monitoring)
15. [Project Structure](#project-structure)

---

## Architecture Overview

```
┌─────────────┐     ┌───────────────────────────────────────────────┐
│   Browser   │────▶│  Nginx (Reverse Proxy + Rate Limiting)        │
└─────────────┘     └──────────────┬────────────────────────────────┘
                                   │
                    ┌──────────────▼──────────────┐
                    │   Next.js 14 Frontend        │
                    │   TypeScript + TailwindCSS   │
                    └──────────────┬───────────────┘
                                   │ REST API
                    ┌──────────────▼───────────────┐
                    │   FastAPI Backend (Python)    │
                    │   10 Service Routers          │
                    │   JWT Auth + RBAC             │
                    │   Audit Middleware            │
                    └────┬──────────────────┬───────┘
                         │                  │
              ┌──────────▼──────┐  ┌────────▼────────┐
              │  PostgreSQL 16  │  │   Redis 7        │
              │  (Primary DB)   │  │   (Cache/Rate)   │
              └─────────────────┘  └─────────────────┘
```

---

## System Modules

| Module | Endpoint Prefix | Description |
|--------|----------------|-------------|
| Authentication | `/api/v1/auth` | JWT login, refresh, RBAC, password management |
| Students | `/api/v1/students` | Registration, profiles, enrollment history |
| Faculty | `/api/v1/faculty` | Staff profiles, department assignment, schedules |
| Academic | `/api/v1/academic` | Courses, programs, sections, grade submission |
| Registrar | `/api/v1/registrar` | Enrollment approval, transcript generation (PDF) |
| Finance | `/api/v1/finance` | Fee structures (ETB), payments, scholarships |
| Library | `/api/v1/library` | Book catalog, loans, returns, overdue fines |
| Notifications | `/api/v1/notifications` | Multi-channel alerts, broadcast messages |
| Analytics | `/api/v1/analytics` | KPI dashboards, enrollment trends, GPA distribution |
| Admin | `/api/v1/admin` | User management, audit logs, system configuration |

### User Roles

| Role | Access Level |
|------|-------------|
| `admin` | Full system access |
| `registrar` | Students, enrollments, transcripts |
| `finance` | Fee records, payments, scholarships |
| `department_head` | Department courses and faculty |
| `professor` | Grade submission for assigned sections |
| `librarian` | Book catalog, loans management |
| `student` | Own profile, grades, notifications |

---

## Tech Stack

### Backend
- **Python 3.11** + **FastAPI 0.111** (async)
- **SQLAlchemy 2.0** (async ORM) + **asyncpg**
- **Alembic** (database migrations)
- **PostgreSQL 16** (primary database)
- **Redis 7** (caching + rate limiting)
- **python-jose** (JWT authentication)
- **passlib + bcrypt** (password hashing)
- **ReportLab** (bilingual PDF transcript generation)
- **Prometheus** (metrics via `prometheus-fastapi-instrumentator`)
- **structlog** (structured logging)

### Frontend
- **Next.js 14** (App Router) + **TypeScript**
- **TailwindCSS 3** (utility-first styling)
- **Recharts** (analytics charts)
- **Axios** (API client with JWT interceptor)
- **Sonner** (toast notifications)
- **React Hook Form + Zod** (form validation)

### Infrastructure
- **Docker** + **Docker Compose**
- **Kubernetes** (deployment manifests included)
- **Nginx** (reverse proxy + rate limiting)
- **Prometheus + Grafana** (monitoring)
- **GitHub Actions** (CI/CD pipeline)

---

## Quick Start (Docker)

### Prerequisites
- Docker Desktop (or Docker Engine + Compose plugin)
- 4 GB RAM minimum

### 1. Clone and configure

```bash
git clone https://github.com/your-org/dkhcbc-ums.git
cd dkhcbc-ums/eth-ums
cp .env.example .env
```

Edit `.env` — at minimum set a strong `JWT_SECRET_KEY`:

```bash
# Generate a secure key:
python -c "import secrets; print(secrets.token_hex(64))"
```

### 2. Start all services

```bash
docker compose up -d
```

This starts: PostgreSQL, Redis, Backend API, Frontend, Nginx, Prometheus, Grafana.

### 3. Run migrations and seed

```bash
# Run Alembic migrations
docker compose exec backend alembic upgrade head

# Seed initial data (admin, faculty, student accounts)
docker compose exec backend python seed.py
```

### 4. Access the application

| Service | URL |
|---------|-----|
| Frontend Dashboard | http://localhost |
| API Documentation | http://localhost/api/docs |
| Grafana Monitoring | http://localhost:3001 |
| Prometheus | http://localhost:9090 |
| Backend Direct | http://localhost:8000 |

---

## Local Development

### Backend

```bash
cd eth-ums/backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Start PostgreSQL and Redis (using Docker for infra only)
docker compose up -d db redis

# Copy and configure environment
cp .env.example .env
# Edit DATABASE_URL and REDIS_URL to point to localhost

# Run migrations
alembic upgrade head

# Seed initial data
python seed.py

# Start development server
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

### Frontend

```bash
cd eth-ums/frontend

# Install dependencies
npm install

# Configure API URL
echo "NEXT_PUBLIC_API_URL=http://localhost:8000/api/v1" > .env.local

# Start development server
npm run dev
```

Open http://localhost:3000

---

## Database Migrations

```bash
# Apply all pending migrations
alembic upgrade head

# Create a new migration
alembic revision --autogenerate -m "add_table_name"

# Rollback one step
alembic downgrade -1

# View migration history
alembic history

# View current revision
alembic current
```

---

## Default Credentials

After running `python seed.py`:

| Role | Email | Password |
|------|-------|----------|
| Super Admin | admin@dkhcbc.edu.et | Admin@1234 |
| Registrar | registrar@dkhcbc.edu.et | Reg@1234 |
| Finance Officer | finance@dkhcbc.edu.et | Fin@1234 |
| Professor | prof.bekele@dkhcbc.edu.et | Prof@1234 |
| Student | abebe.kebede@dkhcbc.edu.et | Stu@1234 |

> **⚠️ Change all passwords immediately in production.**

---

## API Documentation

Interactive API documentation is available at:
- **Swagger UI**: http://localhost:8000/api/docs
- **ReDoc**: http://localhost:8000/api/redoc
- **OpenAPI JSON**: http://localhost:8000/api/openapi.json

### Authentication

All endpoints (except `/health` and `/api/v1/auth/login`) require a Bearer token:

```bash
# Login
curl -X POST http://localhost:8000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "admin@dkhcbc.edu.et", "password": "Admin@1234"}'

# Use token
curl http://localhost:8000/api/v1/analytics/dashboard \
  -H "Authorization: Bearer <access_token>"
```

### Key Endpoints

```
POST   /api/v1/auth/login                          Login
GET    /api/v1/auth/me                             Current user
POST   /api/v1/auth/refresh                        Refresh token

GET    /api/v1/students                            List students
POST   /api/v1/students                            Register student
GET    /api/v1/students/{id}                       Get student
GET    /api/v1/students/{id}/enrollments           Student enrollments

GET    /api/v1/academic/courses                    Course catalog
POST   /api/v1/academic/grades                     Submit grade
PATCH  /api/v1/academic/grades/{id}/finalize       Finalize grade
GET    /api/v1/academic/students/{id}/gpa          Student GPA

GET    /api/v1/registrar/enrollment-queue          Pending enrollments
PATCH  /api/v1/registrar/enrollment-queue/{id}/approve
POST   /api/v1/registrar/transcripts/generate      Generate PDF transcript
GET    /api/v1/registrar/transcripts/{no}/download Download transcript

POST   /api/v1/finance/payments                    Record payment
GET    /api/v1/finance/summary                     Finance KPIs

GET    /api/v1/analytics/dashboard                 Admin dashboard
GET    /api/v1/analytics/enrollment-trends         Monthly trends
GET    /api/v1/analytics/gpa-distribution          Grade breakdown
```

---

## Kubernetes Deployment

```bash
# Apply all manifests
kubectl apply -f eth-ums/infra/k8s/

# Check rollout status
kubectl rollout status deployment/dkhcbc-backend -n dkhcbc-ums
kubectl rollout status deployment/dkhcbc-frontend -n dkhcbc-ums

# Run migrations in cluster
kubectl exec -n dkhcbc-ums deployment/dkhcbc-backend -- alembic upgrade head

# View logs
kubectl logs -n dkhcbc-ums deployment/dkhcbc-backend -f

# Scale backend
kubectl scale deployment/dkhcbc-backend --replicas=4 -n dkhcbc-ums
```

### Required Secrets

Before deploying to Kubernetes, update `infra/k8s/00-namespace-config.yaml`:
- `DATABASE_URL` — production PostgreSQL connection string
- `REDIS_URL` — production Redis connection string
- `JWT_SECRET_KEY` — 64-character random hex string
- `POSTGRES_PASSWORD` — database password

---

## CI/CD Pipeline

The GitHub Actions workflow (`.github/workflows/ci-cd.yml`) runs:

1. **test-backend** — pytest suite against PostgreSQL + Redis services
2. **test-frontend** — TypeScript type-check, ESLint, Next.js build
3. **build-push** — Docker images pushed to GitHub Container Registry (on `main`)
4. **deploy** — kubectl rolling update to Kubernetes cluster (on `main`)

### Required GitHub Secrets

| Secret | Description |
|--------|-------------|
| `KUBECONFIG` | Base64-encoded kubeconfig for production cluster |

---

## Testing

### Backend Tests

```bash
cd eth-ums/backend

# Run all tests
pytest

# With coverage report
pytest --cov=. --cov-report=html
open htmlcov/index.html

# Run specific test file
pytest tests/test_auth.py -v

# Run specific test
pytest tests/test_academic_finance.py::test_compute_cgpa_perfect -v
```

### Test Coverage

| Module | Tests |
|--------|-------|
| Authentication | Login, token refresh, /me, change password |
| GPA Calculator | score_to_grade, compute_cgpa (weighted), gpa_standing |
| Academic API | Course listing, program listing, grade submission |
| Finance API | Summary endpoint, fee structure listing |
| Analytics API | Dashboard KPIs |

---

## Ethiopian Localization

### Ethiopian Calendar (E.C.)

The system stores Ethiopian Calendar dates in dedicated fields:
- `students.dob_ec` — Date of birth in E.C.
- `academic_years.label_ec` — Academic year label (e.g. "2016/17 E.C.")
- `academic_years.label_gc` — Corresponding Gregorian label

### Amharic Language

All major entities include bilingual fields:
- `name_en` / `name_am` — English and Amharic names
- `title` / `title_am` — Bilingual notification titles
- `message` / `message_am` — Bilingual notification messages

PDF transcripts are generated with both English and Amharic content using ReportLab with Noto Sans Ethiopic font support.

### Currency

All financial amounts use **Ethiopian Birr (ETB)**:
- `tuition_etb`, `amount_etb`, `fine_etb`, `total_due_etb`, `paid_etb`

### Grading System

Ethiopian academic grading (CA 30% / Mid 30% / Final 40%):

| Grade | Score | Points |
|-------|-------|--------|
| A+ | 90–100 | 4.0 |
| A  | 85–89  | 4.0 |
| A- | 80–84  | 3.7 |
| B+ | 75–79  | 3.3 |
| B  | 70–74  | 3.0 |
| B- | 65–69  | 2.7 |
| C+ | 60–64  | 2.3 |
| C  | 55–59  | 2.0 |
| F  | < 55   | 0.0 |

---

## Security

- **Password hashing**: bcrypt via passlib
- **JWT tokens**: HS256, configurable expiry
- **RBAC**: Role-based access enforced on every endpoint via `require_roles()` dependency
- **Rate limiting**: Sliding-window per IP via Redis (configurable via env vars)
- **Audit logging**: All POST/PUT/PATCH/DELETE requests logged with user ID, IP, user agent
- **SQL injection**: Prevented by SQLAlchemy ORM parameterized queries
- **CORS**: Configurable origin whitelist
- **Input validation**: Pydantic v2 models on all request bodies
- **Nginx security headers**: X-Content-Type-Options, X-Frame-Options, X-XSS-Protection

---

## Monitoring

### Prometheus Metrics

Available at `/metrics` (restricted by Nginx in production):
- Request count, latency histograms by endpoint
- Active database connections
- Redis hit/miss rates

### Grafana Dashboards

Default admin: `admin` / `grafana_admin`

Access at http://localhost:3001. Prometheus is pre-configured as the default datasource.

### Health Check

```bash
curl http://localhost:8000/health
# {"status":"ok","version":"2.0.0","service":"DKHCBC UMS"}
```

---

## Project Structure

```
eth-ums/
├── .env.example                    # Environment variable template
├── .github/
│   └── workflows/
│       └── ci-cd.yml               # GitHub Actions CI/CD pipeline
├── docker-compose.yml              # Full stack Docker Compose
│
├── backend/
│   ├── main.py                     # FastAPI app entry point
│   ├── requirements.txt            # Python dependencies
│   ├── Dockerfile
│   ├── alembic.ini
│   ├── seed.py                     # Dev/test seed data
│   ├── pytest.ini
│   ├── alembic/
│   │   ├── env.py
│   │   └── versions/
│   │       └── 0001_initial.py     # Full schema migration
│   ├── services/
│   │   ├── auth/router.py          # JWT + RBAC
│   │   ├── student/router.py
│   │   ├── faculty/router.py
│   │   ├── academic/router.py      # Courses, grades, GPA
│   │   ├── registrar/router.py     # Enrollments, transcripts
│   │   ├── finance/router.py       # Fees (ETB), payments
│   │   ├── library/router.py       # Books, loans
│   │   ├── notification/router.py
│   │   ├── analytics/router.py     # KPI dashboards
│   │   └── admin/router.py         # Users, audit logs
│   ├── shared/
│   │   ├── config/
│   │   │   ├── database.py         # Async SQLAlchemy engine
│   │   │   ├── redis_client.py
│   │   │   └── settings.py         # Pydantic settings
│   │   ├── middleware/
│   │   │   ├── audit.py            # Audit log middleware
│   │   │   └── rate_limiter.py     # Redis rate limiting
│   │   ├── models/
│   │   │   └── orm_models.py       # All 20+ SQLAlchemy models
│   │   └── utils/
│   │       ├── gpa_calculator.py   # Ethiopian grading logic
│   │       └── pdf_generator.py    # Bilingual PDF transcripts
│   └── tests/
│       ├── conftest.py             # Pytest fixtures
│       ├── test_auth.py
│       └── test_academic_finance.py
│
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── next.config.js
│   ├── tailwind.config.js
│   ├── tsconfig.json
│   └── src/
│       ├── app/
│       │   ├── layout.tsx          # Root layout with fonts
│       │   ├── page.tsx            # Redirect to /dashboard
│       │   ├── globals.css
│       │   ├── login/page.tsx      # Login page
│       │   └── dashboard/
│       │       ├── layout.tsx      # Sidebar + top bar
│       │       ├── page.tsx        # Analytics dashboard
│       │       ├── students/page.tsx
│       │       ├── faculty/page.tsx
│       │       ├── academic/page.tsx
│       │       ├── registrar/page.tsx
│       │       ├── finance/page.tsx
│       │       ├── library/page.tsx
│       │       ├── notifications/page.tsx
│       │       └── audit/page.tsx
│       ├── hooks/
│       │   └── useAuth.tsx         # Auth context + hook
│       ├── lib/
│       │   └── api.ts              # Axios client + all service APIs
│       └── types/
│           └── index.ts            # TypeScript interfaces
│
└── infra/
    ├── nginx/
    │   └── nginx.conf
    ├── monitoring/
    │   ├── prometheus.yml
    │   └── grafana-datasources.yml
    └── k8s/
        ├── 00-namespace-config.yaml
        ├── 01-backend.yaml
        └── 02-services.yaml
```

---

## License

Copyright © 2025 Debre Kidan Holy Cross Bible College. All rights reserved.
