# 🧠 DevLucid – AI Coding Platform

[![Website Status](https://img.shields.io/website?url=https%3A%2F%2Fdevlucid.com&style=for-the-badge&label=DevLucid.com)](https://devlucid.com)
[![Instagram Follow](https://img.shields.io/badge/Instagram-%23E4405F.svg?style=for-the-badge&logo=Instagram&logoColor=white)](https://instagram.com/devlucidhq)
[![Discord](https://img.shields.io/discord/1470824518489473084?style=for-the-badge&label=Join%20Community&color=5865F2)](https://devlucid.com/discord)

> **Don’t just grind problems. Understand them.**  
> DevLucid is a coding platform that combines a LeetCode‑style arena with an AI mentor and AI code review – built to help you actually *learn*.

---

## 🚀 About the Project

DevLucid is **not** just a ChatGPT wrapper.

It is a full platform with:

- **Coding Arena** – algorithmic tasks in **8 languages** (Python, JavaScript, TypeScript, Java, C++, C, C#, Go) with:
  - Blind‑75‑style problems and language‑specific exercises
  - Hidden & visible test cases (including adversarial ones)
  - Judge0‑based sandboxed execution, strict time limits, and Denial‑of‑Wallet protections

- **Daily Quest & Streaks** – gamified practice:
  - Daily problem with **timezone‑aware** countdown
  - Streak calendar, “dying flame” warnings, and **streak freeze** mechanic
  - Milestone emails for 7‑ and 30‑day streaks

- **Lucid AI (Mentor)** – built‑in AI assistant that:
  - Explains code and algorithms step‑by‑step
  - Helps debug and refactor your solutions
  - Respects per‑plan limits and rate‑limits

- **AI Code Review** – paste your code, get:
  - Bug analysis, refactors, and Big‑O complexity
  - Two modes: “Fix errors” and “Fix & optimize”
  - History of previous reviews, rendered with Markdown + LaTeX

- **Production Features**
  - Auth and profiles via Supabase
  - Subscriptions and billing via Stripe
  - Transactional emails and cron‑driven campaigns via Resend + Vercel Cron
  - Security & cost controls (rate limits, payload limits, pagination, legal docs in sync with real data flows)

This repository is a **public documentation hub** describing how the platform is built and operated.

---

## 🛠 Tech Stack (Under the Hood)

We are transparent about our engineering. For a deeper dive, see [ARCHITECTURE.md](ARCHITECTURE.md).

**Core:**

- **Framework:** Next.js 16 (App Router, Server Components, API Routes)
- **Language:** TypeScript
- **Database:** PostgreSQL (Supabase)
- **ORM:** Prisma
- **Styling:** Tailwind CSS
- **Auth:** Supabase Auth
- **Payments:** Stripe
- **AI Providers:** OpenAI (Lucid AI & Code Review)
- **Code Execution:** Judge0 CE (via RapidAPI)
- **Email:** Resend
- **Hosting & Cron:** Vercel (including Cron Jobs for drip emails and weekly reports)

**UX & Rendering:**

- `react-markdown` + `remark-gfm` + `remark-math` + `rehype-katex` for Markdown + math
- `react-syntax-highlighter` for code blocks
- Custom hooks & components for streaming responses and scroll‑reveal animations

👉 **[Read the full Architecture Overview](ARCHITECTURE.md)**

---

## 🗺 Roadmap (High Level)

**Already shipped**

- ✅ Coding Arena with 8 languages
- ✅ Daily Quest, streaks, streak freeze, and achievement emails
- ✅ Lucid AI mentor integrated into the app (not a separate chat page only)
- ✅ AI Code Review with history and math‑aware rendering
- ✅ Subscription plans (Free / paid tiers), Stripe billing, and usage quotas
- ✅ Judge0 integration with:
  - Per‑language drivers
  - Base64 encoding
  - Strict CPU / wall‑time limits
  - Adversarial test cases to catch naive O(n²) implementations
- ✅ Legal & privacy docs aligned with actual data flows (OpenAI, Judge0, Stripe, Supabase, Vercel, etc.)

**Planned / in exploration**

- ⏳ VS Code / editor integrations
- ⏳ Interactive quizzes & learning paths around specific roles (Frontend, Backend, Fullstack)
- ⏳ Richer progress analytics and weekly/monthly reports
- ⏳ Team / classroom features (multi‑user access, assignments)

The public roadmap in this repo intentionally stays high‑level; specific business experiments and internal milestones remain private.

---

## 🐛 Bug Reporting & Feedback

If you’ve found an issue in the **public docs** (architecture, roadmap, wording) or have suggestions about the platform:

- Open an **Issue** in this repository, or
- Reach out via [DevLucid.com](https://devlucid.com) contact form.

> **Note:** The production application codebase is private. This repository does **not** accept feature PRs for the closed‑source app itself.

---

## 📄 License

The content of this repository (documentation, architecture overview, roadmap) is licensed under the **MIT License**.

**Important:**  
The **DevLucid source code**, branding, and core service logic are **proprietary and closed‑source**.  
This repository serves only as a **public documentation and communication hub** for the project.

[**Visit DevLucid.com →**](https://devlucid.com)
