# Project Report
## FinTrack — Personal Finance Management System

---

### Submitted By
INDRAJEET KUMAR 
ARKA JAIN UNIVERSITY, JAMSHEDPUR, JHARKHAND 
Internship Project — IT DEVELOPER 
Date: May 2026

---

## Table of Contents
1. Project Overview
2. Objectives
3. Technologies Used
4. System Architecture
5. Features Implemented
6. Database Design
7. API Design
8. Frontend Design
9. Screenshots / Module Descriptions
10. Challenges Faced
11. Conclusion
12. Future Enhancements

---

## 1. Project Overview

FinTrack is a full-stack Personal Finance Management System developed as part of an internship project. It is a web-based application that allows users to track their income and expenses, monitor savings goals, manage recurring financial entries, generate visual analytics, and download financial reports as PDF documents.

The application supports multiple users securely through Clerk-based authentication, ensuring that all financial data is private and scoped to each individual user.

---

## 2. Objectives

- Build a production-ready full-stack web application from scratch
- Implement secure user authentication and authorization
- Design a relational database to manage financial data
- Develop a RESTful API with full CRUD operations
- Create an interactive, responsive frontend with data visualizations
- Enable PDF report generation for financial summaries
- Implement recurring transaction automation
- Follow industry-standard practices: contract-first API design, type safety, and modular architecture

---

## 3. Technologies Used

| Layer | Technology |
|---|---|
| **Frontend** | React 19, Vite, TypeScript, Tailwind CSS v4, shadcn/ui |
| **Charts** | Recharts |
| **Routing** | Wouter |
| **State & Data Fetching** | TanStack React Query |
| **Authentication** | Clerk (JWT-based, multi-user) |
| **Backend** | Node.js 24, Express 5, TypeScript |
| **Database** | PostgreSQL |
| **ORM** | Drizzle ORM |
| **Validation** | Zod v4 |
| **API Contract** | OpenAPI 3.1 (contract-first) |
| **Code Generation** | Orval (generates React Query hooks + Zod schemas) |
| **PDF Generation** | jsPDF + html2canvas |
| **Monorepo** | pnpm workspaces |
| **Build Tool** | esbuild |

---

## 4. System Architecture

FinTrack is built as a **pnpm monorepo** with clearly separated packages:

```
fintrack/
├── artifacts/
│   ├── finance-app/        ← React + Vite frontend (port 18748)
│   └── api-server/         ← Express 5 REST API (port 8080)
├── lib/
│   ├── api-spec/           ← OpenAPI spec (source of truth)
│   ├── api-client-react/   ← Auto-generated React Query hooks
│   ├── api-zod/            ← Auto-generated Zod validation schemas
│   └── db/                 ← Drizzle ORM schema + migrations
```

### Architecture Pattern: Contract-First API Development
The OpenAPI specification (`openapi.yaml`) is the single source of truth. All backend Zod validators and frontend React Query hooks are auto-generated from it using Orval. This eliminates type drift between frontend and backend.

### Request Flow
```
User → React Frontend → REST API (Express) → PostgreSQL (via Drizzle ORM)
                ↑
         Clerk JWT Auth
```

---

## 5. Features Implemented

### 5.1 User Authentication
- Secure sign-up and sign-in via Clerk
- Google OAuth support
- JWT-based session management
- All data is scoped per user (multi-user isolation)

### 5.2 Dashboard
- Monthly income and expense summary cards
- Net savings calculation
- Savings goal progress bar
- Pie chart: expense breakdown by category
- Bar chart: monthly income vs expense trend (last 6 months)
- Recent transactions feed

### 5.3 Transaction Management
- Add, edit, and delete income/expense transactions
- Filter by type (income/expense), category, and date range
- Pagination support
- Category and payment method tagging

### 5.4 Recurring Transactions
- Define recurring income/expense entries (daily, weekly, monthly, yearly)
- Set start date, optional end date, and active/paused toggle
- "Apply Due" button creates actual transaction records for all overdue entries
- Next date is automatically advanced after each application

### 5.5 Category Management
- 12 default categories auto-seeded on first login
- Create, edit, and delete custom categories
- Color coding for visual identification
- Categories typed as income, expense, or both

### 5.6 Savings Goals
- Set monthly savings targets
- Track progress against actual net savings
- Add notes to goals

### 5.7 Financial Reports & PDF Export
- Select a custom date range
- View income, expense, and net savings summary
- Category breakdown table
- Full transaction list for the period
- One-click PDF export using jsPDF + html2canvas

### 5.8 Profile Settings
- Update display name, email preferences
- Set preferred currency
- Configure monthly savings target

---

## 6. Database Design

The PostgreSQL database contains five tables:

### `users`
Stores user profile information synced from Clerk.
| Column | Type | Description |
|---|---|---|
| id | serial PK | Internal user ID |
| clerk_user_id | text UNIQUE | Clerk identity reference |
| display_name | text | User's display name |
| currency | text | Preferred currency (default: USD) |
| monthly_savings_target | double | Optional monthly savings goal |

### `categories`
Stores income/expense categories per user.
| Column | Type | Description |
|---|---|---|
| id | serial PK | Category ID |
| clerk_user_id | text | Owner reference |
| name | text | Category label |
| color | text | Hex color code |
| type | text | income / expense / both |

### `transactions`
Stores all income and expense records.
| Column | Type | Description |
|---|---|---|
| id | serial PK | Transaction ID |
| clerk_user_id | text | Owner reference |
| type | text | income / expense |
| amount | double | Transaction value |
| category_id | integer FK | Linked category |
| source | text | Source / merchant |
| payment_method | text | e.g. Cash, Card |
| date | text | ISO date YYYY-MM-DD |
| notes | text | Optional notes |

### `savings_goals`
Stores monthly savings targets.
| Column | Type | Description |
|---|---|---|
| id | serial PK | Goal ID |
| clerk_user_id | text | Owner reference |
| name | text | Goal label |
| target_amount | double | Target value |
| month | integer | Month (1–12) |
| year | integer | Full year e.g. 2026 |

### `recurring_transactions`
Stores recurring entry templates for automation.
| Column | Type | Description |
|---|---|---|
| id | serial PK | Recurring entry ID |
| clerk_user_id | text | Owner reference |
| type | text | income / expense |
| amount | double | Amount per occurrence |
| frequency | text | daily/weekly/monthly/yearly |
| start_date | text | First occurrence date |
| end_date | text | Optional end date |
| next_date | text | Date of next pending application |
| is_active | boolean | Active or paused |

---

## 7. API Design

The REST API is documented in OpenAPI 3.1 format. All routes (except health check) require a valid Clerk JWT bearer token.

### Endpoints Summary

| Method | Endpoint | Description |
|---|---|---|
| GET | /api/healthz | Health check |
| GET | /api/users/profile | Get user profile |
| PUT | /api/users/profile | Update user profile |
| GET | /api/categories | List categories |
| POST | /api/categories | Create category |
| PATCH | /api/categories/:id | Update category |
| DELETE | /api/categories/:id | Delete category |
| GET | /api/transactions | List transactions (with filters) |
| POST | /api/transactions | Create transaction |
| GET | /api/transactions/:id | Get single transaction |
| PATCH | /api/transactions/:id | Update transaction |
| DELETE | /api/transactions/:id | Delete transaction |
| GET | /api/savings-goals | List savings goals |
| POST | /api/savings-goals | Create savings goal |
| PATCH | /api/savings-goals/:id | Update savings goal |
| DELETE | /api/savings-goals/:id | Delete savings goal |
| GET | /api/dashboard/summary | Dashboard summary stats |
| GET | /api/dashboard/category-breakdown | Expense by category |
| GET | /api/dashboard/monthly-trend | Monthly income/expense trend |
| GET | /api/dashboard/recent-transactions | Recent transactions feed |
| GET | /api/recurring-transactions | List recurring entries |
| POST | /api/recurring-transactions | Create recurring entry |
| POST | /api/recurring-transactions/apply | Apply all due entries |
| PATCH | /api/recurring-transactions/:id | Update recurring entry |
| DELETE | /api/recurring-transactions/:id | Delete recurring entry |

---

## 8. Frontend Design

The frontend is a Single Page Application (SPA) built with React and Vite. Key design decisions:

- **Tailwind CSS v4** for utility-first styling with a consistent teal/green brand color (`#0f766e`)
- **shadcn/ui** for accessible, pre-built UI components (dialogs, forms, badges, toasts)
- **Wouter** for lightweight client-side routing
- **React Query** for server state management, caching, and background refetching
- **Clerk's shadcn theme** for a seamlessly branded sign-in/sign-up experience
- Fully **responsive layout** — sidebar collapses on mobile

---

## 9. Challenges Faced

| Challenge | Solution |
|---|---|
| Avoiding type drift between frontend and backend | Adopted contract-first development: OpenAPI spec drives all codegen |
| Clerk auth integration with Express | Used `@clerk/express` middleware with proxy URL support |
| PDF generation of dynamic React content | Used `html2canvas` to capture DOM nodes and embed in `jsPDF` |
| Orval generating duplicate exports | Changed orval output mode to `"single"` and manually managed barrel exports |
| Recurring transaction date advancement across months | Implemented UTC-based `Date` arithmetic to correctly handle month/year rollovers |

---

## 10. Conclusion

FinTrack is a complete, production-ready personal finance management system. It demonstrates the full lifecycle of modern web application development — from database design and API development to frontend engineering, authentication, and deployment readiness. The project follows industry-standard practices including contract-first API design, type-safe code generation, multi-user data isolation, and modular monorepo structure.

---

## 11. Future Enhancements

- Budget limits per category with overspend alerts
- Bank statement CSV import
- Email/push notifications for savings milestones
- Mobile app (React Native / Expo)
- Multi-currency conversion using live exchange rates
- AI-powered spending insights and recommendations
____
