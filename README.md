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
