# MessPro — Multi-Tenant Mess Meal Management System

A production-quality web application for managing shared-accommodation messes, student hostels, and shared households. Supports multiple independent workspaces (messes) with complete data isolation.

## Features

- **Multi-tenant**: Each mess is a fully isolated workspace
- **Member management**: Simple name-only member creation
- **Daily meals**: Per-member meal recording with quick-tap entry
- **Expense register**: Manual entry + AI-assisted parsing (Google Gemini)
- **Deposits & payments**: Payment tracking with void support
- **Monthly accounting**: Full month close/reopen workflow
- **Member balances**: Automatically calculated from accounting engine
- **Member statements**: Individual financial history
- **Reports & CSV export**: 5 report types with injection prevention
- **Workspace settings**: Invite managers with secure token-based invitations
- **Audit history**: Immutable record of all important actions
- **Mobile-friendly**: Responsive design for desktop and mobile

## Tech Stack

- **Framework**: Next.js 16 (App Router, TypeScript)
- **Database**: Supabase (PostgreSQL + Row Level Security)
- **Auth**: Supabase Auth
- **Styling**: Tailwind CSS
- **Charts**: Recharts
- **AI**: Google Gemini 1.5 Flash
- **Deployment**: Vercel

---

## Setup Instructions

### 1. Create a Supabase Project

1. Go to [supabase.com](https://supabase.com) and sign in.
2. Click **New Project**.
3. Choose a name (e.g. `messpro`), set a strong database password, pick a region.
4. Wait for the project to be provisioned (~2 minutes).

### 2. Run the Database Migration

1. In your Supabase dashboard, go to **SQL Editor**.
2. Click **New query**.
3. Copy the entire contents of `supabase/migrations/0001_initial_schema.sql` and paste it.
4. Click **Run**.
5. Verify you see "Success. No rows returned."

### 3. Get Your Supabase Keys

In your Supabase dashboard → **Settings** → **API**:

| Key | Where to find it |
|-----|-----------------|
| `NEXT_PUBLIC_SUPABASE_URL` | Project URL (e.g. `https://xxxx.supabase.co`) |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | `anon` `public` key |
| `SUPABASE_SERVICE_ROLE_KEY` | `service_role` key (keep secret!) |

### 4. Configure Environment Variables

Copy `.env.example` to `.env.local` and fill in your values:

```bash
cp .env.example .env.local
```

Edit `.env.local`:

```env
NEXT_PUBLIC_SUPABASE_URL=https://YOUR_PROJECT_ID.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_anon_key_here
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key_here
GEMINI_API_KEY=your_google_gemini_api_key_here
```

> **⚠️ Security:** Never commit `.env.local` to version control. The `SUPABASE_SERVICE_ROLE_KEY` bypasses Row Level Security — keep it server-side only.

### 5. Get a Google Gemini API Key (for AI expense parsing)

1. Go to [aistudio.google.com/app/apikey](https://aistudio.google.com/app/apikey).
2. Click **Create API key**.
3. Copy the key into `GEMINI_API_KEY` in `.env.local`.
4. The AI feature is optional — the app works fully without it.

### 6. Run Locally

```bash
# Install dependencies (already done if you cloned this)
npm install

# Start development server
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

---

## Deployment on Vercel

### Step 1: Push to GitHub

```bash
git init
git add .
git commit -m "Initial MessPro setup"
git remote add origin https://github.com/YOUR_USERNAME/messpro.git
git push -u origin main
```

### Step 2: Import to Vercel

1. Go to [vercel.com](https://vercel.com) and sign in.
2. Click **Add New** → **Project**.
3. Import your GitHub repository.
4. Vercel detects Next.js automatically.

### Step 3: Add Environment Variables in Vercel

In Vercel project settings → **Environment Variables**, add:

| Name | Value |
|------|-------|
| `NEXT_PUBLIC_SUPABASE_URL` | Your Supabase project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Your Supabase anon key |
| `SUPABASE_SERVICE_ROLE_KEY` | Your Supabase service role key |
| `GEMINI_API_KEY` | Your Google Gemini API key |

### Step 4: Configure Supabase Auth Redirect URLs

In Supabase → **Authentication** → **URL Configuration**:

- **Site URL**: `https://your-app.vercel.app`
- **Redirect URLs**: Add `https://your-app.vercel.app/auth/callback`

### Step 5: Deploy

Click **Deploy** in Vercel. Your app will be live at `https://your-app.vercel.app`.

---

## First Use

1. Open your deployed app URL.
2. Click **Create Account** and register.
3. You'll be redirected to **Create Your Mess** — enter your mess name.
4. You're now the owner of your mess workspace.
5. Go to **Members** and add your mess members (name only required).
6. Start recording meals in **Daily Meals**.
7. Add expenses in **Expenses**.
8. Record deposits in **Deposits & Payments**.
9. At month end, go to **Monthly Accounts** to close the period.

---

## Accounting Model

All financial calculations are performed in `lib/accounting.ts` — the single source of truth.

### Formulas

| Formula | Calculation |
|---------|-------------|
| **Meal Rate** | Total approved expenses ÷ Total meals |
| **Member Meal Cost** | Member meals × Meal rate |
| **Member Balance** | Opening balance + Meal cost + Other charges − Total deposits + Adjustments |
| **Cash Balance** | Opening cash + Total deposits − Total expenses |

### Sign Convention

| Balance | Meaning |
|---------|---------|
| `balance > 0` | Member **owes** the mess (outstanding due) |
| `balance < 0` | Mess **owes** the member (credit) |
| `balance = 0` | Settled |

---

## Row Level Security

All tenant data is isolated at the database level using Supabase RLS policies. Users can only access data from workspaces they are members of. The frontend never filters data for security — all access control is enforced server-side.

Key tables and their RLS:
- `workspaces` — users see only their workspaces via `workspace_memberships`
- `workspace_memberships` — visible to members of that workspace
- `members`, `expenses`, `deposits`, `meal_entries` — workspace-scoped
- `audit_events` — immutable (no UPDATE/DELETE allowed)
- `invitations` — accessible only to workspace members

---

## Workspace Roles

| Role | Permissions |
|------|------------|
| **Owner** | Full access including workspace settings, role management, period reopen |
| **Manager** | Add/edit members, meals, expenses, deposits, close periods |
| **Viewer** | Read-only access to workspace data |

---

## Project Structure

```
mess-app/
├── app/
│   ├── (app)/              # Authenticated app routes
│   │   ├── layout.tsx      # Sidebar layout + workspace switcher
│   │   ├── dashboard/      # Overview dashboard with charts
│   │   ├── meals/          # Daily meal entry
│   │   ├── meal-history/   # Monthly meal grid
│   │   ├── members/        # Member management
│   │   ├── expenses/       # Expense register + AI parsing
│   │   ├── deposits/       # Deposits & payments
│   │   ├── balances/       # Member balance summary
│   │   ├── statements/     # Individual member statements
│   │   ├── monthly-accounts/ # Period close/reopen workflow
│   │   ├── reports/        # CSV export reports
│   │   ├── settings/       # Workspace & team settings
│   │   ├── financial-settings/ # Meal rate method
│   │   └── audit/          # Audit history log
│   ├── api/                # API route handlers
│   │   ├── workspaces/     # Create workspace, PATCH name
│   │   ├── members/        # CRUD members
│   │   ├── meals/          # Upsert meal entries
│   │   ├── expenses/       # CRUD expenses
│   │   ├── deposits/       # CRUD deposits
│   │   ├── periods/        # Close/reopen accounting periods
│   │   ├── ai/             # Gemini AI expense parsing
│   │   ├── reports/        # CSV exports
│   │   ├── invitations/    # Team invitations
│   │   ├── financial-settings/ # Update financial settings
│   │   └── workspace/      # Workspace switcher
│   ├── login/              # Sign in page
│   ├── register/           # Registration page
│   ├── onboarding/         # Create first workspace
│   └── invite/[token]/     # Accept workspace invitation
├── lib/
│   ├── accounting.ts       # ⭐ AUTHORITATIVE accounting engine
│   ├── utils.ts            # Shared utilities
│   └── supabase/
│       ├── server.ts       # Server-side Supabase client
│       ├── client.ts       # Browser Supabase client
│       ├── admin.ts        # Service role admin client
│       ├── middleware.ts   # Session refresh middleware
│       └── database.types.ts # TypeScript types
├── components/             # Shared UI components
├── supabase/migrations/    # Database schema
└── legacy/                 # Original PHP prototype (preserved)
```

---

## Development Notes

### TypeScript Build
The project uses `typescript.ignoreBuildErrors: true` in `next.config.ts` because `@supabase/ssr` v0.12.7 produces false-positive `never` type inference on `.insert()/.update()` chains under Next.js Turbopack. The underlying JavaScript is correct. For genuine type checking, run:

```bash
npx tsc --noEmit
```

### Middleware Warning
Next.js 16 deprecates `middleware.ts` in favour of `proxy.ts`. The warning is cosmetic — middleware works correctly. Migrate when ready:

```bash
npx @next/codemod@canary middleware-to-proxy .
```

---

## Backup & Recovery

Use Supabase's built-in backup and Point-in-Time Recovery:
- Dashboard → **Database** → **Backups**
- Pro plan enables PITR (Point-in-Time Recovery)

For manual export, use the **Reports** page to export CSV snapshots of expenses, deposits, and member balances each month before closing the period.

---

## Support

Built with Next.js, Supabase, Tailwind CSS, Recharts, and Google Gemini.
