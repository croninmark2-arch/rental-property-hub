# RentTrack

A personal landlord tool for tracking rental income, expenses, mileage, and generating P&L reports across a portfolio of rental properties.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 5000)
- `pnpm --filter @workspace/scripts run seed` — seed database with initial property data
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite (artifacts/renttrack), wouter routing, shadcn/ui, TanStack Query
- API: Express 5 (artifacts/api-server) at `/api`
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)

## Where things live

- `lib/api-spec/openapi.yaml` — single source of truth for all API contracts
- `lib/db/src/schema/` — DB schema (properties, units, payments, expenses)
- `lib/api-client-react/src/generated/` — generated React Query hooks
- `lib/api-zod/src/generated/` — generated Zod validation schemas
- `artifacts/api-server/src/routes/` — Express route handlers
- `artifacts/renttrack/src/pages/` — Dashboard, Properties, Expenses, Reports
- `scripts/src/seed.ts` — seeds the 3 real properties + 4 units

## Architecture decisions

- OpenAPI-first: all endpoints defined in the spec before writing any code; Orval codegen produces both client hooks and server-side Zod validators from the same spec
- `/reports/property` uses a query param `propertyId` (not a path param) to avoid Orval TS2308 naming collision between Zod schema exports and TypeScript interface exports
- Monetary columns use `doublePrecision` in Drizzle (not `numeric`) so values arrive as JS numbers — avoids `Number()` casting throughout route handlers
- Payment tracking is purely manual (no PayPal/CashApp API integration) — landlord selects method from dropdown when marking paid
- Expense `amount` for mileage entries stores the dollar deduction (miles × $0.67 IRS rate), with the raw `miles` stored separately for reference

## Product

- **Dashboard**: monthly rent collection at a glance — 4 unit cards each showing Paid/Unpaid status, one-click "Mark Paid" flow with payment method picker (PayPal, Cash App, Check, Cash)
- **Properties**: view/add/edit properties and their units, manage tenant info, toggle vacancy
- **Expenses**: log expenses by category (Tax, Insurance, Repair, Mileage, Other) with file upload UI for bill photos, mileage auto-calculates IRS deduction at $0.67/mile
- **Reports**: portfolio P&L and per-property P&L with month/year picker — Gross Rent, Vacancy Loss, Net Rental Income, expense breakdown, Net Operating Income

## Properties (seeded)

1. 146 W. Fourth St, Corning NY 14830 — single family, $1,700/mo
2. 114 Orchard St, Horseheads NY 14845 — single family, $1,405/mo
3. 220 Elmwood Ave, Elmira Heights NY 14903 — duplex (Unit A $1,400/mo, Unit B $1,100/mo)

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- After changing `lib/api-spec/openapi.yaml`, always re-run `pnpm --filter @workspace/api-spec run codegen` before starting the API server
- Orval TS2308 collision: any operation with BOTH path params AND query params generates a `<OperationIdPascal>Params` in both `api.ts` and `types/`. Fix: use query params only for such endpoints (no `{id}` path segment)
- The seed script is idempotent (skips if properties exist) — safe to run multiple times

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
