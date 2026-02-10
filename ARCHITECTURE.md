# 🏗️ DevLucid — Technical Architecture

> **Public technical overview.** > This document describes the architecture and technology choices of **DevLucid** for employers, contributors, and technical users.
> It does not expose business logic, proprietary AI prompts, or implementation details.

---

## 📋 Project Overview

**DevLucid** is an **AI-powered programming tutor SaaS**. Unlike generic chat bots, it is engineered to act as a mentor: users ask coding questions in a chosen language (e.g., Python, JavaScript), 
and the system returns step-by-step explanations, logic breakdowns, and runnable code.

The product is monetized via subscription tiers (Free, Basic, Standard, Pro) with daily usage quotas. It fully supports **Polish** and **English** with automated Geo-IP–based locale detection.

---

## 🛠️ Tech Stack

We chose a modern, type-safe stack optimized for performance and developer experience.

| Layer | Technology | Rationale |
| :--- | :--- | :--- |
| **Framework** | **Next.js 16** (App Router) | SSR/SSG for SEO and fast first load; API routes and middleware for secure auth and edge logic. |
| **Language** | **TypeScript** | End-to-end type safety across frontend and API; ensures better refactors and fewer runtime errors. |
| **Database** | **PostgreSQL** (Supabase) | Managed Postgres with connection pooling; fits the relational model for users, questions, and subscriptions perfectly. |
| **ORM** | **Prisma** | Type-safe schema, automated migrations, and connection pooling (e.g., `pgbouncer`) for serverless environments. |
| **Styling** | **Tailwind CSS 4** | Utility-first CSS and design tokens; allows for fast UI iteration and consistent theming across the app. |
| **State** | **React Context + Hooks** | Profile, usage, and i18n live in context; no complex global store (Redux/Zustand) needed for current scope. |
| **Auth** | **Supabase Auth** | JWT-based auth, email verification, and session refresh. We use `getUser()` in middleware for strict server-side validation. |
| **Payments** | **Stripe** | Checkout, subscriptions, webhooks. Plan quotas and billing lifecycle are handled asynchronously via webhooks. |
| **AI** | **OpenAI** (GPT-4o Mini) | Streaming completions for low perceived latency and incremental UX. |
| **Deployment** | **Vercel** | Next.js–native hosting, Edge Functions, and Geo headers for instant locale detection. |

**Additional Libraries:**
* `react-markdown` + `remark-gfm` / `rehype-katex` for rich text rendering.
* `react-syntax-highlighter` (Prism) for beautiful code blocks.
* `Zod` for strict runtime request validation.
* `Resend` for reliable transactional emails.

---

## 🔄 High-Level Architecture

### 1. Request Flow

1.  **Middleware (Edge)** Every request (except static assets and auth callback paths) passes through Next.js middleware. It enforces the **canonical host** (e.g., `www.devlucid.com`), resolves **locale** (Cookie override → Geo-IP → Default), and refreshes the **Supabase session**.  
    *Security Note:* We use `getUser()` (not just `getSession()`) to ensure "zombie" sessions (deleted users) are instantly invalidated.

2.  **API Routes (Serverless)** Authenticated AI requests hit `POST /api/explain`. The handler:
    * Validates the body using **Zod**.
    * Checks **Rate Limits** and **Daily Quota** (syncing Prisma with Supabase ID).
    * Calls the AI provider with **streaming enabled**.
    * Returns a `ReadableStream` sent as **Server-Sent Events (SSE)**.
    * *Stripe Webhooks:* Handled by a dedicated route with signature verification to manage subscription states securely.

3.  **Client (Frontend)** The "Ask Mentor" interface is a Client Component. It consumes the SSE stream using `response.body.getReader()`. As chunks arrive, the state is updated, and markdown is rendered incrementally.
    * *Performance:* Code highlighting (`react-syntax-highlighter`) is loaded dynamically with `ssr: false` to keep the initial bundle size small.

### 2. Data Flow Summary

* **Auth:** Supabase Auth (JWTs/Cookies) → Middleware validation → API routes resolution.
* **Billing:** Stripe Webhooks → Prisma Database update → User UI reflects new quota.
* **Streaming:** OpenAI Stream → Backend `ReadableStream` → Client SSE → Incremental React Render.

---

## 🎯 Key Features & Engineering Challenges

### ⚡ 1. Real-time Streaming UX
* **Challenge:** Waiting for a full AI explanation (30+ seconds) creates a poor user experience.
* **Solution:** We implemented a custom streaming architecture. The backend exposes the AI response as `Content-Type: text/event-stream`. The client parses JSON lines (`data: {...}`) in real-time. This reduces **Time To First Byte (TTFB)** to milliseconds, making the app feel instant.

### 🎨 2. Performance-First Syntax Highlighting
* **Challenge:** Syntax highlighters are heavy JavaScript libraries. Running them on the server slows down page loads.
* **Solution:** We use **Dynamic Imports** for the `CodeBlock` component. Highlighting logic runs *only* on the client and *only* when a code block is detected. This keeps the core bundle lightweight.

### 🛡️ 3. Secure Session Handling
* **Challenge:** Preventing access to premium API routes by users with stale cookies or expired subscriptions.
* **Solution:** A robust Middleware layer checks `getUser()` against Supabase on every protected navigation. If a user is deleted or banned, their cookies are instantly cleared, and they are redirected to login. API routes perform a secondary check against the Prisma database to verify active subscription quotas.

---

## 🔒 Security & Scalability

### Security Measures
* **Rate Limiting:** Critical endpoints (login, register, AI generation) are protected by in-memory rate limiters (configurable windows).
* **Input Validation:** All incoming data is sanitized and validated with **Zod** schema to prevent injection attacks or malformed AI prompts.
* **Environment Safety:** All API keys (OpenAI, Stripe Secrets) are stored as Server-Side Environment Variables and are never exposed to the client bundle.

### Scalability Strategy
* **Database Pooling:** Prisma is configured to use connection pooling (via Supabase `pgbouncer`) to handle thousands of concurrent serverless function invocations without exhausting database connections.
* **Edge Caching:** Static assets and marketing pages utilize Vercel's Edge Network for global caching.
* **Serverless:** The backend scales automatically from 0 to N instances based on traffic demand, ensuring cost-efficiency during idle times and stability during spikes.

---

## 📁 Configuration Types (Example)

*Note: This is an illustrative TypeScript interface for our subscription model. Actual business logic is private.*

```typescript
// Example: How we type-check plan quotas
type PlanSlug = 'free' | 'basic' | 'standard' | 'pro';

interface PlanConfig {
  slug: PlanSlug;
  quota: number;        // e.g., 5 questions/day
  features: string[];   // e.g., ["GPT-4o", "Priority Support"]
  priceIds: Partial<Record<'PLN' | 'USD' | 'EUR', string>>;
}
