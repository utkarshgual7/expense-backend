# Expense Tracker — Backend

Hono API over Supabase Postgres. Deploys as a Render Web Service.

## Setup

```bash
npm install
cp .env.example .env     # set DATABASE_URL
npm run dev              # :4000
```

Run `supabase/schema.sql` once in the Supabase SQL editor before first use.

## Deploy (Render → Web Service)

| | |
| --- | --- |
| Build | `npm ci && npm run build` |
| Start | `npm start` |
| Root dir | *(blank)* |

Env vars:
- `DATABASE_URL` — Supabase transaction pooler, port **6543**, keep `?pgbouncer=true`
- `CORS_ORIGINS` — the deployed frontend origin, e.g. `https://expense-frontend.onrender.com`

Do not set `PORT`; Render injects it.

## API

| Endpoint | Notes |
| --- | --- |
| `POST /api/expenses` | `title` ≤100, `amount` > 0 with ≤2dp, `category ∈ food\|travel\|bills\|other`, `date?` YYYY-MM-DD and not future |
| `GET /api/expenses` | `?category=&from=&to=&page=1&limit=10` → `{ items, total_count, total_amount }` |
| `DELETE /api/expenses/:id` | 200 `{ ok: true }`, or 404 |
| `GET /api/expenses/summary?month=YYYY-MM` | per-category sums + total |
| `GET /health` | liveness only — says nothing about the database |

Errors are always `{ "error": string }`: 400 invalid input, 404 missing record, 500 otherwise.

`total_count` and `total_amount` are aggregated in Postgres across the entire filtered
set, concurrently with the page query — never summed from the returned page.

`src/shared/expense.ts` is the Zod contract, duplicated in the frontend repo. Keep the
two copies in sync when validation rules change.
