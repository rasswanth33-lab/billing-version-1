# Eliyo Billing System — Architecture & Build Plan

## Where the repo stands

The current repository is a **bare Next.js scaffold** — config files only, no
application code, schema, or UI. The enhancement prompt assumes an existing
system to extend; in reality this is a greenfield build on a pre-chosen stack.

**Stack (from `package.json` / configs):** Next.js 14 App Router · Prisma ORM ·
PostgreSQL (recommended) · TypeScript · Tailwind (CSS-variable design tokens) ·
React Hook Form + Zod · Zustand · TanStack Table · lucide-react.

## Folder convention

```
src/
  app/                # routes (App Router) + route handlers under app/api/*
  modules/<feature>/  # each module: components, server actions, queries
  lib/                # db client, gst engine, validations, shared utils
prisma/
  schema.prisma       # single source of truth for the data model
```

`src/modules/` is already wired into the Tailwind content globs, so each business
module (sales, purchase, inventory…) lives in its own folder and is mounted into
the App Router via thin route files.

## What's in this foundation drop

| File | Purpose |
|------|---------|
| `prisma/schema.prisma` | 52 models / 15 enums covering org, RBAC, parties, products, inventory, sales, purchase, payments, expenses, cash/bank, double-entry ledger, manufacturing, documents, notifications. Structurally validated. |
| `src/lib/db.ts` | Prisma client singleton (hot-reload safe). |
| `src/lib/gst.ts` | GST engine: line discounts, inclusive/exclusive pricing, CGST/SGST vs IGST split, round-off. Unit-tested. |
| `src/lib/validations/invoice.ts` | Zod schema — the validation pattern every module follows. |
| `src/app/globals.css` | Design tokens the Tailwind config references. |
| `.env.example` | Required environment variables. |

## Reality check on scope

The prompt lists ~150 features across 25 modules — sales, purchase, inventory,
multi-branch, GST returns, P&L / balance sheet / cash flow, RBAC, manufacturing,
AI, integrations. That is a multi-month effort for a team, not a single
generation. Building it all at once would produce code that compiles but doesn't
actually work — the opposite of the "production-ready" requirement.

The workable path is **vertical slices**: ship one module fully (DB → API →
validation → UI → PDF/print) before starting the next, so every step is real and
testable.

## Progress

- [x] **Foundation** — schema, db client, GST engine, validations, design tokens.
- [x] **Step 1** — seed (org/branch/warehouse/admin/tax/units), dashboard shell + nav, Products master (list, create, edit, soft-delete).
- [ ] Customers + Suppliers masters
- [ ] Sales: GST invoice (create/PDF/payments)
- [ ] Inventory wiring, Purchase, Cash/Bank, Reports…

## Suggested phase order

1. **Setup & auth** — migrations, seed (org/branch/admin), login, RBAC middleware.
2. **Masters** — products, categories, units, tax/HSN, customers, suppliers.
3. **Sales core** — GST invoice create/list/view, PDF, print, status, payments.
4. **Inventory** — stock movements wired to invoices/purchases, levels, low-stock.
5. **Purchase** — orders, bills, returns, supplier payments.
6. **Cash & bank + accounting** — accounts, journal postings, party ledgers.
7. **Reports** — sales, purchase, GST summary, P&L, balance sheet (date/branch filters, export).
8. **Remaining doc types** — quotation, proforma, challan, credit/debit note, recurring.
9. **Multi-branch, expenses, notifications.**
10. **Optional** — manufacturing/BOM, then AI feature hooks, integrations.

## Getting it running

```bash
npm install
cp .env.example .env        # set DATABASE_URL to your Postgres
npx prisma migrate dev --name init
npm run dev
```

## Notes

- `provider` is `postgresql`. For throwaway local dev on SQLite you'd remove the
  enums (SQLite has no native enum support) — Postgres is the right call for a
  real ERP, so it's the default.
- The double-entry `ChartOfAccount` / `JournalEntry` models exist because
  Balance Sheet, P&L and Cash Flow reports can't be done correctly without a
  proper ledger; party `LedgerEntry` rows drive customer/supplier statements.
