# 🏗️ DevLucid — Technical Architecture

> **Public technical overview.**  
> This document describes the architecture and technology choices of **DevLucid** for employers, contributors, and technical users.  
> It does not expose business logic, proprietary AI prompts, or implementation details.

---

## 📋 Project Overview

**DevLucid** is a **full coding practice platform** with:

- **Coding Arena** (LeetCode‑style tasks in 8 languages)
- **Lucid AI** – an in‑app mentor/chat for explanations and help
- **AI Code Review** – automated code review with refactors and complexity analysis
- **Gamification** – daily quest, streaks, streak freezes, and achievement emails
- **Production‑grade platform** – subscriptions, emails, legal docs, rate limits, and cost‑controls

The product is monetized via subscription tiers with per‑plan usage quotas.  
The app fully supports **Polish** and **English** with cookie‑ and route‑based locale handling (and Geo‑IP used in some flows).

---

## 🛠️ Tech Stack

We use a modern, type‑safe stack optimized for UX, observability, and cost control.

| Layer | Technology | Rationale |
| :--- | :--- | :--- |
| **Framework** | **Next.js 16** (App Router) | File‑based routing, Server Components, API Routes, and good Vercel integration. |
| **Language** | **TypeScript** | End‑to‑end type safety across frontend, API routes, and server actions. |
| **Database** | **PostgreSQL** (Supabase) | Managed Postgres for users, submissions, problems, paths, favorites, and quotas. |
| **ORM** | **Prisma** | Type‑safe schema, migrations, and good DX for complex queries and relations. |
| **Styling** | **Tailwind CSS** | Utility‑first CSS for fast iteration and consistent design system. |
| **State** | **React + Context/Hooks** | Local state for editor/UI; simple context for language, auth user, and feature flags. |
| **Auth** | **Supabase Auth** | Email/password and OAuth with secure JWT cookies and server‑side `getUser()` checks. |
| **Payments** | **Stripe** | Subscriptions, upgrades/downgrades, webhooks for billing state and quotas. |
| **AI** | **OpenAI** | Used for Lucid AI (mentor chat) and AI Code Review. Responses streamed and rendered incrementally. |
| **Code Execution** | **Judge0 CE** (RapidAPI) | Sandboxed execution for Coding Arena in 8 languages, with strict limits and drivers. |
| **Email** | **Resend** | Transactional emails: subscription events, streak milestones, drip campaigns, weekly report. |
| **Deployment** | **Vercel** | First‑class hosting for Next.js, Cron Jobs, and regional deployment. |

**Key Libraries & Utilities**

- `react-markdown`, `remark-gfm`, `remark-math`, `rehype-katex` – rich markdown + math rendering (AI responses, code review).
- `react-syntax-highlighter` (Prism) – syntax highlighting for code blocks.
- `Zod` – runtime validation for API payloads.
- Custom **rate limiter** – `checkRateLimit` with central `RateLimitConfigs`.
- Custom **streak** & **calendar** utilities – timezone‑aware computation (`getDateKeyInTimezone`, etc.).

---

## 🔄 High-Level Architecture

The platform is split into three main product surfaces:

1. **Coding Arena** – tasks, submissions, history, progress, streak.
2. **Lucid AI (Mentor)** – chat‑style assistance, explanations, and guided help.
3. **AI Code Review** – paste code → get structured review.

### 1. Coding Arena Flow

**Core entities**

- `User` – Supabase + Prisma record.
- `Problem` – algorithmic task (title, description, per‑language starter code, difficulty).
- `TestCase` – input/expectedOutput pairs for Judge0; some hidden, some visible.
- `Submission` – user submissions with status (`ACCEPTED`, `WRONG_ANSWER`, `ERROR`), language, and timestamps.
- `LearningPath` – “career paths” grouping problems across multiple languages.
- `UserFavorite` – many‑to‑many relation between users and problems.
- `CalendarStreak` (computed) – based on `Submission` history and freeze uses.

**Execution Path (run code / submit solution)**

1. **Client** (Arena page)
   - User selects language and writes code in the Monaco‑style editor.
   - Calls Server Action `runCode` or `submitSolution`.

2. **Server Action** `app/problems/[slug]/actions.ts`
   - Validates:
     - Code length (`MAX_CODE_CHARS`, hard‑capped to avoid Denial‑of‑Wallet).
     - Per‑user **rate limits** (`RUN_CODE`, `SUBMIT_SOLUTION`).
   - For `submitSolution`:
     - Fetches all `TestCase` rows for the problem.
     - For each test case:
       - Combines user code + hidden **driver / wrapper** code per language:
         - Python: function call + `print(...)`.
         - JavaScript / TypeScript: `console.log(...)`.
         - Java / C / C# / Go: full `main` with argument parsing and normalized output printing.
       - Sends combined code to **Judge0**.
       - Compares normalized output with expected result (`normalizeOutput`).
     - Persists `Submission` rows with status and metadata.
     - Recomputes streak and checks for **7‑day / 30‑day** milestones; triggers emails if needed.
   - For `runCode`:
     - Sends the current code once to Judge0 and returns raw stdout/stderr merged.

3. **Judge0 Integration**
   - Endpoint: `https://judge0-ce.p.rapidapi.com/submissions?base64_encoded=true&wait=true`
   - Request body:
     - `source_code` – **base64** encoded combined code.
     - `stdin` – **base64** encoded (currently empty for arena; design is ready for future stdin).
     - `language_id` – mapped per language (`JUDGE0_LANGUAGE_IDS`).
     - **Limits**: `cpu_time_limit: 2.0`, `wall_time_limit: 2.0` seconds.
   - Response:
     - `stdout`, `stderr`, `compile_output`, `message` are base64‑decoded by our backend.
     - `status.id` is normalized to `{ code: 0/1, error: boolean }`.
   - **Wrappers & drivers:**
     - **Go** wrapper imports are now **conditional**:
       - Scalar results: `import "fmt"`.
       - Array results: `import ("fmt"\n"strconv"\n"strings")` plus helper `intSliceToStrings`.
       - This prevents „imported and not used” compiler errors.
     - Similar drivers for Java, C, and C# parse arrays, booleans, and strings, and print canonical string representations (e.g. `[0,1]`, `true`/`false`) to match `expectedOutput`.

4. **Streak & Calendar**
   - `GET /api/calendar-streak` computes:
     - `currentStreak`, `previousStreak`, `solvedDates`, `freezeUsedDates`.
     - `currentLocalHour`, `todayKey` based on user timezone (or sensible fallback).
   - Frontend shows:
     - Calendar with solved/freezed days.
     - Flame widget with “dying flame” state after 20:00 local time.
     - Streak freeze modal (using Shadcn `AlertDialog`).

5. **Daily Quest / Career Paths**
   - `GET /problems/daily` – daily problem with countdown to **local midnight**, using helper `getEndOfDayLocalMs(timezone)`.
   - `GET /problems/paths` – lists multi‑language learning paths (frontend, backend, fullstack, algorithms‑interview, system‑low‑level).
   - `GET /api/learning-paths/[slug]/languages` – returns languages present in a path, used to filter tasks & favorites per language.

### 2. Lucid AI (Mentor) Flow

- **Endpoint:** `POST /api/explain`  
  Request contains user question + optional code and language.
- **Backend:**
  - Validates payload with Zod.
  - Checks user’s plan and remaining **Lucid AI usage**.
  - Assembles prompt, sends to **OpenAI** (GPT model).
  - Streams the result back to the client using **streaming responses**.
- **Frontend:**
  - Shows a chat‑like UI.
  - Renders markdown + code blocks + math (via `react-markdown` + `remark-*` + `rehype-katex`).

### 3. AI Code Review Flow

- **Endpoints:**
  - `POST /api/code-review` – main analysis.
  - `GET /api/code-review/history` – paginated history per user.
  - `GET /api/code-review/limits` – shows remaining quota.

- **Processing:**
  - Validates request with Zod (hard‑limits `code` length).
  - Checks rate limit and subscription quota.
  - Asks OpenAI for:
    - Bug analysis, improvements, complexity, refactor suggestions.
    - „Fix” and „Fix & optimize” modes.
  - Stores result in DB for history.

- **Rendering:**
  - Uses `react-markdown` with GFM + math.
  - Supports LaTeX for Big‑O and formulas.
  - Two tabs: „Fix errors” vs „Fix & optimize”.

---

## 🎯 Test Cases & Hardening

### Adversarial Test Cases

To avoid naive solutions passing:

- **Two Sum**
  - Added large arrays (e.g., 1200+ elements) to expose O(n²) implementations under Judge0 time limits.
- **Contains Duplicate**
  - Added long arrays with a single duplicate at the end to stress brute‑force scanning.
- **Longest Increasing Subsequence**
  - Includes classical LIS cases and an extra sequence that breaks greedy strategies (e.g. `[4,5,6,1,2,3]`).
- **Find Minimum in Rotated Sorted Array**
  - Test cases where min is at:
    - The beginning `[1,2,3,4,5]`
    - The end `[2,3,4,5,1]`
    - The middle of longer sequences `[6,7,8,9,1,2,3,4,5]`.

### Security & Cost Controls

- **Rate limiting:**
  - `RUN_CODE` and `SUBMIT_SOLUTION` guarded by `checkRateLimit` with per‑user windows (e.g., 15 run‑code / min, 10 submit / min).
  - Other critical endpoints (auth, AI, contact form) also guarded.

- **Payload limits:**
  - Code length (e.g., `MAX_CODE_CHARS = 15000`).
  - Selected snippet length for mentor.
  - Contact form message length (`max 2500` chars).

- **Pagination / bounded queries:**
  - History endpoints (questions, progress, calendar streak, admin lists) use `take` and/or date windows (e.g., last 365 days) to avoid unbounded queries.

---

## 📬 Emails, Cron Jobs & Automation

Emails are centralized in `lib/email.ts` and invoked from:

- **Stripe subscription flows**
  - Cancel with/without refund.
- **Streak milestones**
  - 7‑day and 30‑day streak emails on `ACCEPTED` submissions.
- **Drip campaign & reports**
  - `/api/cron/drip` – day‑3 and day‑7 onboarding emails.
  - `/api/cron/streak-reminder` – daily 18:00 local reminders if a streak is at risk.
  - `/api/cron/weekly-report` – weekly stats summary (tasks solved, Lucid AI queries, code reviews).

Configured via `vercel.json` Cron Jobs.

---

## 🔒 Security & Compliance

- **Auth enforcement:**
  - Contact form and other sensitive endpoints require authenticated Supabase session.
  - Email is always taken from the backend session, not from form input.

- **Legal documents:**
  - `/privacy` and `/terms` synchronized with:
    - Actual list of third‑party providers (Vercel, Supabase, Stripe, OpenAI, Judge0/RapidAPI, Resend, analytics, OAuth providers).
    - Real refund policy and subscription behavior.
    - Explicit „No Warranty” clause for AI‑generated code and content.
    - Administrative action rights (e.g., banning users for abuse).

- **Data routing:**
  - Clearly documented which data goes to which provider:
    - Source code + stdin → Judge0, OpenAI (depending on feature).
    - Billing and PII → Stripe, Supabase.
    - Emails → Resend.
  - This is reflected in the legal docs to maintain GDPR‑aligned transparency.

---

## 📁 Configuration Types (Example)

*Note: This is an illustrative TypeScript interface for our subscription model. Actual business logic is private.*

typescript
// Example: How we type-check plan quotas
type PlanSlug = "free" | "basic" | "standard" | "pro";
interface PlanConfig {
slug: PlanSlug;
quota: number; // e.g., problems or AI calls per day
features: string[]; // e.g., ["Lucid AI", "AI Code Review", "Arena Unlimited"]
priceIds: Partial<Record<"PLN" | "USD" | "EUR", string>>;
}
This architecture continues to evolve as DevLucid grows, but the core principles remain:- **Type‑safety**- **Transparent data flows**- **Cost‑aware execution (Judge0, OpenAI)**- **Excellent UX for learning and practicing code**
