# CLAUDE.md — Kitchen Management System

This document describes the structure, conventions, and development workflows for the Kitchen Management System. It is intended for AI assistants (Claude and others) to understand how this repository is organized and how to work effectively within it.

---

## Repository Overview

This repository contains a **standalone Kitchen Management System** for professional restaurant/food-service operations. The application covers:

- Ingredients (raw materials, suppliers, pricing)
- Semi-finished recipes (intermediate preparations, recursive cost calculation)
- Final recipes (finished dishes with automated food cost)
- Food Matrix (unified cost view across all products)
- Weekly production planning
- HACCP compliance and traceability
- Order management and shopping lists
- Multi-store support
- Document archive and invoices

The UI is primarily in **Italian** (professional restaurant context). Comments in source code may also be in Italian.

---

## Repository Structure

```
/home/user/kitchen-app/             ← Git repository root
├── CLAUDE.md                       ← This file
├── kitchen-management-standalone.tar.gz  ← Full application source code
└── railway (1).json                ← Railway deployment configuration
```

### Extracting the Application Source

The application source lives inside the tarball. To work with the code:

```bash
tar -xzf kitchen-management-standalone.tar.gz
cd kitchen-management-app-modified/
```

### Application Source Structure

```
kitchen-management-app-modified/
├── client/                  ← React 19 frontend (TypeScript + Tailwind CSS 4)
│   ├── index.html
│   ├── public/
│   └── src/
│       ├── App.tsx          ← Root component with routing
│       ├── main.tsx         ← React entry point
│       ├── index.css        ← Global styles
│       ├── const.ts         ← Frontend constants
│       ├── _core/
│       │   └── hooks/
│       │       └── useAuth.ts       ← Auth hook
│       ├── components/      ← Reusable UI components
│       │   ├── DashboardLayout.tsx
│       │   ├── RecipeForm.tsx       ← Shared recipe form
│       │   ├── StoreSelector.tsx
│       │   ├── ErrorBoundary.tsx
│       │   └── ui/          ← shadcn/ui components (Radix UI based)
│       ├── contexts/
│       │   └── ThemeContext.tsx
│       ├── hooks/           ← Shared React hooks
│       │   ├── useComposition.ts
│       │   ├── useKeyboardShortcuts.ts
│       │   ├── useMobile.tsx
│       │   └── usePersistFn.ts
│       ├── lib/             ← Utility functions
│       └── pages/           ← One file per route
│           ├── Home.tsx
│           ├── Login.tsx
│           ├── Dashboard.tsx
│           ├── Ingredients.tsx
│           ├── Recipes.tsx          ← Semi-finished recipes
│           ├── FinalRecipes.tsx
│           ├── FoodMatrix.tsx
│           ├── Production.tsx / ProductionNew.tsx
│           ├── ShoppingList.tsx
│           ├── OrdersNew.tsx / OrderHistory.tsx
│           ├── HACCPLanding.tsx / HACCPProductions.tsx / HACCPNonCompliance.tsx
│           ├── Suppliers.tsx
│           ├── Invoices.tsx
│           ├── Fridges.tsx
│           ├── DocumentArchive.tsx
│           ├── Users.tsx            ← Admin: user management
│           ├── SuperAdminDashboard.tsx
│           ├── MultiStoreEditor.tsx
│           ├── CostAnalysisDashboard.tsx
│           ├── Menu.tsx
│           ├── Waste.tsx
│           ├── Storage.tsx
│           ├── Assistant.tsx        ← AI assistant integration
│           └── NotFound.tsx
│
├── server/                  ← Express 4 + tRPC 11 backend (Node.js 22+)
│   ├── _core/               ← Framework infrastructure
│   │   ├── index.ts         ← Server entry point (Express setup, port binding)
│   │   ├── trpc.ts          ← tRPC init; publicProcedure, protectedProcedure, adminProcedure
│   │   ├── context.ts       ← tRPC request context (user, storeId, logout)
│   │   ├── auth.ts          ← Local auth: PBKDF2 hashing, JWT via jose, cookie management
│   │   ├── env.ts           ← Environment variable declarations
│   │   ├── vite.ts          ← Dev mode: Vite middleware setup
│   │   ├── oauth.ts         ← OAuth routes (legacy/alternative auth)
│   │   ├── llm.ts           ← LLM integration utilities
│   │   ├── notification.ts
│   │   ├── sdk.ts
│   │   └── types/
│   ├── db.ts                ← Database access layer (all DB queries)
│   ├── routers.ts           ← Main tRPC router (assembles all sub-routers)
│   ├── calculations.ts      ← Recursive recipe cost calculation
│   ├── allergens.ts         ← Allergen logic
│   ├── storeMiddleware.ts   ← Multi-store context middleware
│   ├── exportExcel.ts       ← Excel export utilities
│   ├── storage.ts           ← File storage (AWS S3)
│   ├── generateOrderPDF.ts  ← PDF generation for orders
│   ├── generateUserOrderPDF.ts
│   ├── importSalaInventory.ts
│   ├── auditLogDb.ts / auditLogRouter.ts / auditLogHelper.ts
│   ├── documentsDb.ts / documentsRouter.ts
│   ├── fridgesRouter.ts
│   ├── haccpDb.ts / haccpRouter.ts
│   ├── invoicesDb.ts / invoicesRouter.ts
│   ├── multiStoreEditorDb.ts / multiStoreEditorRouter.ts
│   ├── nonConformitiesDb.ts / nonConformitiesRouter.ts
│   ├── orderSessionsDb.ts / orderSessionsRouter.ts
│   ├── storesDb.ts / storesRouter.ts
│   ├── scripts/
│   │   ├── export_excel.py
│   │   └── import_excel.py
│   └── *.test.ts            ← Vitest test files
│
├── shared/                  ← Code shared between client and server
│   ├── types.ts             ← Re-exports all types from drizzle/schema + errors
│   ├── const.ts             ← Shared constants (error messages, etc.)
│   ├── allergens.ts         ← Allergen definitions
│   ├── recipeValidation.ts  ← Recipe validation (used both client & server)
│   └── _core/
│       └── errors.ts        ← Error constructors (ForbiddenError, etc.)
│
├── drizzle/                 ← Database schema and migrations
│   ├── schema.ts            ← Drizzle ORM table definitions + TypeScript types
│   ├── relations.ts         ← Drizzle ORM relation definitions
│   ├── 0000_*.sql → 0030_*.sql  ← Migration files (auto-generated)
│   └── meta/                ← Drizzle migration metadata snapshots
│
├── package.json             ← Dependencies and scripts
├── pnpm-lock.yaml           ← Lockfile (pnpm)
├── tsconfig.json            ← TypeScript configuration
├── vite.config.ts           ← Vite build configuration
├── vitest.config.ts         ← Vitest test configuration
├── drizzle.config.ts        ← Drizzle Kit configuration
├── components.json          ← shadcn/ui configuration
└── patches/
    └── wouter@3.7.1.patch   ← Patched wouter dependency
```

---

## Technology Stack

| Layer | Technology |
|-------|-----------|
| **Frontend framework** | React 19 |
| **Language** | TypeScript (strict mode) |
| **Styling** | Tailwind CSS v4 (via `@tailwindcss/vite`) |
| **UI components** | shadcn/ui (Radix UI primitives) |
| **Routing** | Wouter v3 |
| **Server state** | TanStack Query v5 + tRPC v11 client |
| **Forms** | React Hook Form + Zod v4 |
| **Animations** | Framer Motion |
| **Backend framework** | Express 4 |
| **API layer** | tRPC v11 (with superjson transformer) |
| **Database** | MySQL 8+ |
| **ORM** | Drizzle ORM |
| **Auth** | Local email/password; PBKDF2 (Web Crypto API); JWT via `jose`; httpOnly cookies |
| **PDF generation** | PDFKit + jsPDF |
| **Excel** | ExcelJS |
| **File storage** | AWS S3 (`@aws-sdk/client-s3`) |
| **Build tool** | Vite 7 (client), esbuild (server bundle) |
| **Test runner** | Vitest 2 |
| **Package manager** | pnpm 10 |
| **Runtime** | Node.js 22+ |
| **Deployment** | Railway (primary), VPS/Docker also supported |

---

## Development Setup

### Prerequisites
- Node.js 22+
- pnpm (`npm install -g pnpm`)
- MySQL 8+ (local or cloud)

### Environment Variables

Create a `.env` file in the project root:

```env
# Required
DATABASE_URL="mysql://user:password@localhost:3306/kitchen_db"
JWT_SECRET="minimum-64-character-random-string"

# Optional
PORT=3000
VITE_APP_TITLE="Kitchen Management"
```

Generate a secure JWT secret:
```bash
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

### Install and Run

```bash
pnpm install         # Install dependencies
pnpm db:push         # Generate + apply DB schema migrations
pnpm dev             # Start dev server (port 3000, with Vite HMR)
pnpm build           # Production build
pnpm start           # Run production build
pnpm test            # Run Vitest tests
pnpm check           # TypeScript type-check (no emit)
pnpm format          # Prettier formatting
```

### First Login
- Navigate to `http://localhost:3000`
- Click "Registrati" to register
- **The first registered account automatically becomes Admin**
- Subsequent users can be promoted from the `/users` page (admin only)

---

## Key Development Conventions

### TypeScript Path Aliases

Configured in both `tsconfig.json` and `vite.config.ts` / `vitest.config.ts`:

```typescript
import { something } from "@/components/MyComponent"; // → client/src/components/MyComponent
import { SomeType } from "@shared/types";               // → shared/types
```

### tRPC Procedure Types

Defined in `server/_core/trpc.ts`:

```typescript
publicProcedure    // No auth required
protectedProcedure // Requires authenticated user (any role)
adminProcedure     // Requires role === 'admin'
```

For role checks beyond admin (e.g., manager), use `protectedProcedure` and check `ctx.user.role` inside the handler:
```typescript
if (ctx.user?.role !== "admin" && ctx.user?.role !== "manager") {
  throw new Error("Unauthorized");
}
```

### User Roles

```typescript
type Role = "user" | "admin" | "manager" | "cook"
```

- `admin` — full access, user management, all CRUD
- `manager` — can create/update ingredients, recipes; cannot manage users
- `cook` — read-only access to recipes and production
- `user` — basic read access

### Database Patterns

All DB access goes through `server/db.ts`. Never import Drizzle directly in routers — call the exported functions.

```typescript
// ✅ Correct
import * as db from "./db";
const items = await db.getIngredients(ctx.currentStoreId);

// ❌ Wrong — don't use drizzle directly in routers
```

**ID conventions:**
- User table: `int` auto-increment primary key
- All other entities: `varchar(36)` UUID (generated by client with `nanoid` or `uuid`)

**Soft delete:** Ingredients use `isActive: false` for deletion. Final recipes have an `isActive` flag. Most other entities use hard deletes.

**Multi-store:** Most data is scoped to a `storeId`. The current store is provided via `ctx.currentStoreId` (derived from `user.preferredStoreId`).

**Numeric fields in MySQL:** Prices, quantities, percentages are stored as `decimal`/`varchar` in the DB and need explicit parsing (`parseFloat`, `.toString()`) when converting between JS numbers and DB values.

### Schema Types

Types are inferred from Drizzle table definitions:

```typescript
// In drizzle/schema.ts:
export type Ingredient = typeof ingredients.$inferSelect;
export type InsertIngredient = typeof ingredients.$inferInsert;

// Import from shared/types.ts (re-exports all schema types):
import type { Ingredient } from "@shared/types";
```

### Shared Validation

Recipe validation is shared between client and server in `shared/recipeValidation.ts`:

```typescript
import { validateRecipe, isDuplicateComponent } from "../shared/recipeValidation";
```

### tRPC Router Pattern

Each domain has its own router file (`server/*Router.ts`) that is composed into the main `appRouter` in `server/routers.ts`:

```typescript
// server/routers.ts
export const appRouter = router({
  auth: authRouter,
  ingredients: ingredientsRouter,
  recipes: recipesRouter,
  // ...
});
```

### Auth Cookie

Session cookie name: `kitchen_session` (httpOnly, path `/`).
JWT claims: `{ userId: string, name: string }`.
Expiry: 1 year.

---

## Testing

Tests are co-located with server code in `server/**/*.test.ts`.

### Running Tests

```bash
pnpm test           # Run all tests
pnpm test --watch   # Watch mode
```

### Test Pattern

Tests use `appRouter.createCaller(ctx)` with a manually constructed mock context — no HTTP layer or real database required:

```typescript
import { describe, expect, it } from "vitest";
import { appRouter } from "./routers";
import type { TrpcContext } from "./_core/context";

function createAdminContext(): TrpcContext {
  return {
    user: {
      id: 1,
      openId: "admin-user",
      role: "admin",
      // ... other required User fields
    },
    req: { protocol: "https", headers: {} } as TrpcContext["req"],
    res: { clearCookie: () => {} } as TrpcContext["res"],
    currentStoreId: null,
    logout: () => {},
  };
}

describe("my feature", () => {
  it("admin can do X", async () => {
    const caller = appRouter.createCaller(createAdminContext());
    const result = await caller.myRouter.myProcedure({ ... });
    expect(result).toBeDefined();
  });
});
```

Tests that hit the database will return empty arrays/null when `DATABASE_URL` is not set (the DB layer handles missing DB gracefully). This allows auth and validation logic to be tested without a real database.

### Existing Test Files

| File | What it tests |
|------|--------------|
| `server/ingredients.test.ts` | Ingredients CRUD and role-based access |
| `server/recipeForm.validation.test.ts` | Recipe form validation |
| `server/finalRecipes.serverValidation.test.ts` | Final recipe server-side validation |
| `server/production.test.ts` | Production planning |
| `server/production_flow.test.ts` | Full production flow |
| `server/shopping_list.test.ts` | Shopping list logic |
| `server/shoppingListSession.test.ts` | Shopping list sessions |
| `server/auth.logout.test.ts` | Auth logout |
| `server/bugfixes.test.ts` | Regression tests |

---

## Build & Deployment

### Production Build

```bash
pnpm build
```

This runs two build steps:
1. **Vite** builds the React client → `dist/public/`
2. **esbuild** bundles the Express server → `dist/index.js`

### Starting Production

```bash
node dist/index.js
# or
NODE_ENV=production pnpm start
```

### Railway Deployment (recommended)

Config is in `railway (1).json`:
```json
{
  "build": { "builder": "NIXPACKS", "buildCommand": "corepack enable && pnpm install && pnpm build" },
  "deploy": { "startCommand": "node dist/index.js", "healthcheckPath": "/", "restartPolicyType": "ON_FAILURE" }
}
```

Set environment variables `DATABASE_URL` and `JWT_SECRET` in the Railway dashboard.

### Database Migrations

```bash
pnpm db:push    # Generates SQL migrations + applies them immediately
```

Migration files are stored in `drizzle/` (numbered `0000_*.sql` through `0030_*.sql`). The schema source of truth is `drizzle/schema.ts`. Do not edit migration files manually; use `drizzle-kit generate` then `drizzle-kit migrate`.

---

## Application Domain Model

### Recipe Hierarchy (3 levels)

```
Level 0: Ingredients    → Raw materials with purchase price
Level 1: Semi-finished  → Intermediate preparations (can contain Ingredients)
Level 2: Final Recipes  → Finished dishes (contain Ingredients + Semi-finished)
```

Cost calculation is recursive: `calculateRecipeCost()` in `server/calculations.ts` traverses the hierarchy bottom-up.

### Key Database Tables

| Table | Purpose |
|-------|---------|
| `users` | User accounts with roles |
| `stores` | Points of sale (multi-store support) |
| `storeUsers` | User↔Store role associations |
| `suppliers` | Ingredient suppliers |
| `ingredients` | Raw materials (Level 0) |
| `semiFinishedRecipes` | Intermediate recipes (Level 1) |
| `finalRecipes` | Final dishes (Level 2) |
| `foodMatrix` | Unified product view |
| `weeklyProductions` | Production planning by week |
| `orders` / `orderItems` | Order history |
| `haccp` | HACCP compliance records |
| `productionBatches` | Production batch tracking |
| `wasteRecords` | Waste tracking |
| `menuTypes` / `menuItems` | Menu definitions |
| `auditLog` | Audit trail |
| `nonConformities` | Non-conformity tracking |
| `recipeVersions` | Recipe version history |
| `cloudStorage` | File storage references |

### Ingredient Categories (Italian)

`Additivi`, `Alcolici`, `Bevande`, `Birra`, `Caffè`, `Carni`, `Farine`, `Latticini`, `Non Food`, `Packaging`, `Spezie`, `Verdura`, `Altro`

### Unit Types

- `u` — units (pieces, e.g. eggs)
- `k` — kilograms/weight-based

---

## Code Style and Conventions

- **Strict TypeScript** — no `any` except where explicitly required by Drizzle quirks (`data as any` for DB inserts)
- **ESM modules** — `"type": "module"` in package.json; use `import/export`, not `require`
- **Prettier** — run `pnpm format` before committing
- **No barrel files** — import directly from source files
- **Italian comments** — domain-specific comments often in Italian; this is intentional
- **Error handling** — DB functions return `[]`/`null`/throw depending on context; always check `if (!db)` at the top of DB functions
- **Zod validation** — all tRPC inputs are validated with Zod schemas inline in the router

---

## Important Files for Common Tasks

| Task | File(s) to read/edit |
|------|---------------------|
| Add a new tRPC endpoint | `server/routers.ts` or create `server/myFeatureRouter.ts` |
| Add a new DB table | `drizzle/schema.ts` → run `pnpm db:push` |
| Add a new page | `client/src/pages/MyPage.tsx` + register in `client/src/App.tsx` |
| Modify auth logic | `server/_core/auth.ts`, `server/_core/context.ts` |
| Shared validation | `shared/recipeValidation.ts` |
| Cost calculation | `server/calculations.ts` |
| Environment config | `server/_core/env.ts` |
| PDF generation | `server/generateOrderPDF.ts`, `server/generateUserOrderPDF.ts` |
| Excel import/export | `server/exportExcel.ts`, `server/scripts/import_excel.py` |

---

## Known Issues and Technical Debt

See `BUG_REPORT.md` and `todo.md` inside the extracted source for known issues. Key ongoing work includes:

- Infinite nesting for semi-finished recipes (currently 2-level only)
- Automated unit conversion between `kg`, `g`, `u`
- Configurable waste percentages per production batch
- Full Excel import/export testing with real data
- Consolidated Food Matrix with full cost propagation

---

## Utility Scripts

Several one-off Python and JS scripts exist for data import/migration (not part of normal workflow):

```
import_data.py                 # Generic data import
import_all_ingredients.py      # Bulk ingredient import
import_final_recipes.py        # Final recipe import
migrate_suppliers.py / .mjs    # Supplier migration
tag_and_deduplicate.py         # Ingredient deduplication
fix_*.py / fix_*.sql           # Data correction scripts
```

These scripts were used during initial data population and are kept for reference.
