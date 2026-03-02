# 🗺️ DevLucid Roadmap & Vision

> **Status:** Active Development  
> **Focus:** From “AI tutor” → full **coding practice platform** (Arena + AI Mentor + AI Code Review).

This document outlines where DevLucid is **today** and where it is heading.  
The original “chat‑first” idea has evolved into a **code‑first** platform with an Arena, Daily Quest, Lucid AI and AI Code Review.

---

## ✅ Phase 1: Foundation (Completed)

*Build the core AI & SaaS infrastructure.*

- [x] **AI Logic Engine** – core logic for explaining code and algorithms in natural language.
- [x] **Streaming Architecture** – real‑time AI responses streamed to the browser for low perceived latency.
- [x] **Multi‑Language AI Support** – questions and answers in **Polish** and **English**.
- [x] **SaaS Infrastructure**
  - Stripe subscriptions (plans, upgrades, downgrades)
  - Supabase Auth (email/password + OAuth)
  - Per‑plan daily quotas & usage tracking
- [x] **Community** – Discord server, basic support/ticketing flow.

---

## ✅ Phase 2: Interactive Coding Platform (Current Core – Most Shipped)

*From “chat‑only” to a real coding product: LeetCode‑style Arena + AI help.*

### 💻 1. Coding Arena & Execution

- [x] **In‑Browser Editor & Arena**
  - Task list (Blind‑75‑style algorithms + language‑specific tasks)
  - 8 languages: Python, JavaScript, TypeScript, Java, C++, C, C#, Go
- [x] **Sandboxed Execution (Judge0)**
  - Server‑side execution environment with strict time and memory limits
  - Per‑language drivers / wrappers for consistent input/output formatting
  - Base64‑encoded payloads to avoid UTF‑8 issues
- [x] **Test Case Validation**
  - Visible + hidden test cases per problem
  - Adversarial cases to detect naive O(n²) solutions and wrong greedy approaches
- [x] **Daily Quest**
  - One daily problem with timezone‑aware countdown
  - Clear “time left today” indicator in user’s local timezone

### 🎓 2. Structured Practice & Gamification

- [x] **Career Paths (Learning Paths)**
  - Multi‑language paths (Frontend, Backend, Fullstack, Algorithms, System/Low‑Level)
  - Problems grouped across languages instead of being limited to one stack
- [x] **Problem & Progress Tracking**
  - Accepted submissions history
  - Favorites, streak calendar, and per‑path progress indicator
- [x] **Gamification**
  - Streaks, **streak freeze**, “dying flame” state after 20:00 local time
  - Daily quest integration with streak system
- [x] **Email & Automation**
  - 7‑day and 30‑day streak milestone emails
  - Drip emails (onboarding around day 3 and 7)
  - Weekly activity report
  - Vercel Cron Jobs powering drip, streak reminder, and weekly report

### 🧠 3. Lucid AI & AI Code Review

- [x] **Lucid AI – Context‑Aware Mentor**
  - Explains code and algorithms step‑by‑step
  - Works together with the Arena (not a separate static chat)
  - Input length limits + rate limits to keep costs predictable

- [x] **AI Code Review**
  - Paste code, choose mode:
    - “Fix errors”
    - “Fix & optimize (Code Review)”
  - Structured feedback: bugs, improvements, complexity, refactors
  - Markdown + math (LaTeX) rendering
  - History UI (past reviews per user)

---

## 🔮 Phase 3: Community & Ecosystem (Planned / In Progress)

*From “solo practice” → collaborative & ecosystem features.*

- [ ] **Richer Learning Experience**
  - Interactive quizzes and mini‑challenges attached to problems
  - More tailored “paths” (e.g. specific job roles / exams)
- [ ] **Peer & Team Features**
  - Peer review between users
  - Private workspaces for small teams, classrooms or bootcamps
- [ ] **Mock Interviews**
  - AI‑driven interview sessions based on Arena problems
  - Timed rounds, feedback on both code and communication
- [ ] **Certificates & Progress Summaries**
  - Certificates upon completing certain paths or milestones
  - Sharable progress dashboards (e.g., for LinkedIn / portfolio)
- [ ] **Editor / IDE Integrations**
  - VS Code (or other editor) extension that brings Lucid AI & Arena tasks into the local dev environment

---

## 📝 How to Use This Roadmap

This roadmap is **deliberately high‑level** and focuses on:

- What is **already in production**.
- What is **being iterated on** for better UX, quality, and security.
- What is **planned** but not yet committed to specific dates.

If you have ideas, questions, or feature requests, feel free to open an Issue in this repository or reach out via [devlucid.com](https://devlucid.com).
