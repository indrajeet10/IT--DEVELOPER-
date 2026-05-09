# FinTrack — Personal Finance Management System

A full-stack personal finance web application built as an internship project. Track income and expenses, monitor savings goals, automate recurring entries, visualize your finances with charts, and export PDF reports.

---

## Features

- **Authentication** — Secure sign-up / sign-in via Clerk (email + Google OAuth)
- **Dashboard** — Summary cards, savings progress, pie chart (by category), monthly bar chart
- **Transactions** — Add, edit, delete income/expense entries with category and date filters
- **Recurring Entries** — Define daily/weekly/monthly/yearly templates; apply them in one click
- **Categories** — Custom income/expense categories with color coding (12 defaults pre-seeded)
- **Savings Goals** — Set monthly targets and track progress
- **Reports** — Date-range financial summary with one-click PDF export
- **Profile** — Update name, currency preference, and monthly savings target

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 19, Vite, TypeScript, Tailwind CSS v4, shadcn/ui |
| Charts | Recharts |
| Auth | Clerk |
| Backend | Node.js 24, Express 5, TypeScript |
| Database | PostgreSQL + Drizzle ORM |
| Validation | Zod v4 |
| API Contract | OpenAPI 3.1 (contract-first, codegen via Orval) |
| PDF Export | jsPDF + html2canvas |
| Monorepo | pnpm workspaces |

---

## Project Structure

```
fintrack/
├── artifacts/
│   ├── finance-app/        # React + Vite frontend
│   └── api-server/         # Express 5 REST API
├── lib/
│   ├── api-spec/           # OpenAPI spec (source of truth)
│   ├── api-client-react/   # Auto-generated React Query hooks
│   ├── api-zod/            # Auto-generated Zod schemas
│   └── db/                 # Drizzle ORM schema & migrations
├── PROJECT_REPORT.md       # Full internship project report
└── README.md
```

---

## Getting Started

### Prerequisites

- Node.js 20+
- pnpm (`npm install -g pnpm`)
- PostgreSQL database

### 1. Clone the repository

```bash
git clone https://github.com/YOUR-USERNAME/fintrack.git
cd fintrack
```

### 2. Install dependencies

```bash
pnpm install
```

### 3. Set up environment variables

Copy the example env file and fill in your values:

```bash
cp .env.example .env
```

Then edit `.env` with your actual credentials (see `.env.example` for what's needed).

### 4. Push the database schema

```bash
pnpm --filter @workspace/db run push
```

### 5. Start the development servers

In two separate terminals:

```bash
# Terminal 1 — API server (port 8080)
pnpm --filter @workspace/api-server run dev

# Terminal 2 — Frontend (port 5173)
pnpm --filter @workspace/finance-app run dev
```

Then open [http://localhost:5173](http://localhost:5173) in your browser.

---

## Environment Variables

See `.env.example` for the full list. Key variables:

| Variable | Description |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `CLERK_SECRET_KEY` | Clerk backend secret key |
| `CLERK_PUBLISHABLE_KEY` | Clerk publishable key |
| `VITE_CLERK_PUBLISHABLE_KEY` | Clerk key for the frontend |

---

## Available Scripts

| Command | Description |
|---|---|
| `pnpm --filter @workspace/api-server run dev` | Start API server in dev mode |
| `pnpm --filter @workspace/finance-app run dev` | Start frontend in dev mode |
| `pnpm run typecheck` | Full TypeScript check across all packages |
| `pnpm run build` | Build all packages |
| `pnpm --filter @workspace/api-spec run codegen` | Regenerate API hooks & Zod schemas |
| `pnpm --filter @workspace/db run push` | Push DB schema changes |

---

## API Overview

All endpoints are prefixed with `/api` and require a valid Clerk JWT (except `/api/healthz`).

- `GET /api/healthz` — Health check
- `GET/PUT /api/users/profile` — User profile
- `GET/POST/PATCH/DELETE /api/categories` — Category management
- `GET/POST/PATCH/DELETE /api/transactions` — Transaction management
- `GET/POST/PATCH/DELETE /api/savings-goals` — Savings goals
- `GET/POST/PATCH/DELETE /api/recurring-transactions` — Recurring entries
- `POST /api/recurring-transactions/apply` — Apply all due recurring entries
- `GET /api/dashboard/summary` — Dashboard stats
- `GET /api/dashboard/category-breakdown` — Expense by category
- `GET /api/dashboard/monthly-trend` — Monthly income vs expense trend
- `GET /api/dashboard/recent-transactions` — Latest transactions

---

## License

MIT
