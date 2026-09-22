# CareerLens AI — AI-Powered Resume & Job Matching Platform

CareerLens AI helps job seekers understand *exactly* how closely their resume
aligns with a specific job description — not with a vague AI-generated
percentage, but with a transparent, explainable, weighted score that breaks
down into skills matched, skills missing, experience relevance, education
fit, and project relevance.

This repository is a fully working **frontend prototype**: a single
self-contained web app that parses real resumes (PDF/DOCX), reads real job
descriptions, and runs an actual matching engine — all in the browser, with
no backend required to try it.

**Live demo flow:** open the app → "Open the demo workspace" → a sample
resume and three job descriptions are pre-loaded and pre-analysed.

---

## Table of contents

- [Core features](#core-features)
- [How the matching actually works](#how-the-matching-actually-works)
- [Project structure](#project-structure)
- [Tech stack](#tech-stack)
- [Architecture insight: prototype vs. production](#architecture-insight-prototype-vs-production)
- [Run it locally](#run-it-locally)
- [Deploy to Vercel](#deploy-to-vercel)
- [Push to GitHub](#push-to-github)
- [Design decisions & known limitations](#design-decisions--known-limitations)
- [Roadmap](#roadmap)

---

## Core features

### 1. Authentication
Register and log in with name/email/password. Sessions persist across
reloads. Every user sees only their own resumes, jobs, and analyses.

### 2. Resume management
- Upload **PDF**, **DOCX**, **TXT**, or **Markdown** — or paste text directly.
- Drag-and-drop or file picker, with size/format validation.
- Keep multiple resume versions side by side (e.g. `Resume_Java`,
  `Resume_Backend`) to test which one scores better against a given job.
- Delete any resume; its analyses are cleaned up with it.

### 3. Resume parsing & structured extraction
Every uploaded resume is converted from unstructured text into structured
data:
- Name, email, phone (best-effort regex/heuristic extraction)
- Education level (Diploma → Bachelor's → Master's → Doctorate)
- Years of experience (from explicit "X years" mentions or date-range math)
- Section detection: Summary, Education, Experience, Projects, Skills,
  Certifications, Achievements
- Skills detected against a 60+ entry taxonomy, categorized (Language,
  Framework, Database, Cloud, DevOps, Tool, Concept, Soft Skill, etc.)

### 4. Job description management
Paste a job posting with a title and company. The parser automatically
splits it into:
- **Required** skills/requirements
- **Preferred / nice-to-have** skills (detected via heading cues like
  "Preferred", "Bonus", "Nice to have")
- Years of experience expected
- Education level expected

### 5. Resume–job matching engine
A rule-based, explainable engine — **not** an LLM asked to guess a number.
See [How the matching actually works](#how-the-matching-actually-works)
below for the full formula.

### 6. Compatibility scoring
A single 0–100 score, built from five weighted factors, always shown with
its breakdown — never a black box.

### 7. Skill gap analysis
Gaps are categorized by severity:
- **Critical** — required skill, no evidence at all
- **Partial** — required skill only implied by an adjacent skill (e.g. you
  list "HTML" and the job wants "React" — related, but not proof)
- **Preferred** — a nice-to-have you don't currently show

### 8. AI-style personalized recommendations
Generated *only* from what's actually present in your resume and the job
text — the engine is explicitly constrained never to suggest claiming a
skill, project, or certification you don't have. Recommendations cover:
missing required skills, underused adjacent skills, weak experience
framing, thin project sections, and preferred skills worth picking up.

### 9. Results dashboard
Per-analysis view with:
- Score ring + five-factor bar breakdown
- Matched / partially-matched / missing skill chips
- Full skill-gap analysis
- Numbered, explained recommendations
- A plain-English written explanation of the score
- Print/Save-as-PDF support

### 10. Analysis history & version comparison
Every analysis is saved. The history page lets you:
- Browse all past analyses in a sortable table
- Compare multiple resume versions run against the *same* job, side by side
  on a bar chart
- Delete individual analyses

### 11. Main dashboard
An overview on login: resumes uploaded, jobs analysed, average match score,
best match score, a score-over-time line chart, most frequently missing
skills, and your strongest detected skills.

### 12. In-app API & schema reference
A dedicated page inside the app itself documents the intended Spring Boot
REST endpoints, the MySQL schema, and the JSON response shape for
`/api/match` — written so the browser-side modules map one-to-one onto a
real backend later.

### 13. Light/dark theme
Full theming via CSS custom properties, respecting system preference by
default with a manual override.

---

## How the matching actually works

The engine mirrors the weighting from the original spec exactly:

```
Final Score = Required Skills × 50%
            + Preferred Skills × 15%
            + Experience       × 15%
            + Education        × 10%
            + Projects         × 10%
```

**Required / Preferred skill scoring** — for each skill the job asks for:
- Found directly in the resume → full credit
- Not found directly, but a *related* skill is present → partial credit
  - 0.5 credit if the related skill is itself specific (e.g. the job wants
    "Spring Boot" and the resume shows "REST APIs")
  - 0.25 credit if the related skill is only a generic base language (e.g.
    the job wants "React" and the resume only shows "JavaScript")
- Not found at all, and nothing related → no credit (this is a **gap**)

**Experience relevance** — blends years-of-experience coverage against what
the posting asks for, with how much of the job's required tech stack
actually appears in the resume's *work experience* section specifically
(not just anywhere in the document).

**Education relevance** — full credit if the candidate's detected degree
level meets or exceeds what the posting asks for; partial credit below
that.

**Project relevance** — how much of the job's tech stack shows up in the
resume's *projects* section, plus a small bonus for having multiple
distinct projects.

A cosine-similarity **semantic score** is also computed across the full
resume and job text (bag-of-words vectors, stopwords removed) and shown as
an extra signal, though it does not feed into the weighted total.

> The compatibility score describes textual alignment with the job posting
> as written. It is not a prediction of hiring outcomes.

---

## Project structure

```
careerlens-ai/
├── index.html      ← the entire application (markup, CSS, JS)
├── README.md        ← this file
└── .gitignore
```

Everything lives in `index.html` by design — see
[Design decisions & known limitations](#design-decisions--known-limitations).
Internally, that one file is organized into clear, self-contained modules
(view them by searching the file for these section headers):

| Module | Responsibility | Mirrors (in a real backend) |
|---|---|---|
| `Store` | localStorage persistence | `*Repository` + MySQL |
| `Auth` | register / login / session | Spring Security + JWT |
| `SKILLS` / `RELATED` | skill taxonomy & adjacency rules | a `skills` reference table |
| `Parser` | text extraction, section splitting, structured field extraction | Apache PDFBox / POI + NLP extraction |
| `Engine` | weighted scoring, gap analysis, recommendation generation | `MatchingEngine` + `AIService` |
| `Chart` | inline SVG rendering (ring, line, bar charts) | Recharts / Chart.js |
| Views (`*View` functions) | all UI screens | React components |
| Router (`go`, `render`) | client-side routing | React Router |

---

## Tech stack

**This build (what's actually running):**

| Layer | Technology |
|---|---|
| UI | Vanilla JavaScript (no framework) |
| Styling | Plain CSS, custom-property design tokens, light/dark theming |
| Fonts | Inter, JetBrains Mono (Google Fonts) |
| PDF parsing | [pdf.js](https://mozilla.github.io/pdf.js/) (via CDN) |
| DOCX parsing | [mammoth.js](https://github.com/mwilliamson/mammoth.js) (via CDN) |
| Charts | Hand-built inline SVG — no charting library |
| Persistence | Browser `localStorage` |
| Auth | Client-side, simplified hashing (prototype only — see limitations) |
| Build tooling | None — static HTML, zero build step |

**Intended production stack** (from the original spec, documented in-app):

| Layer | Technology |
|---|---|
| Frontend | React.js, Axios, React Router, Recharts/Chart.js |
| Backend | Java, Spring Boot, Spring Web, Spring Security, Spring Data JPA, Hibernate |
| Auth | JWT-based, Spring Security |
| Database | MySQL |
| Document processing | Apache PDFBox (PDF), Apache POI (DOCX) |
| AI / NLP | LLM API + embeddings for deeper semantic matching |
| Dev tools | Git, GitHub, Maven, Postman, IntelliJ IDEA / VS Code |

---

## Architecture insight: prototype vs. production

This project is deliberately built in two conceptual layers that happen to
currently live in one file:

1. **A pure logic layer** (`Parser`, `Engine`, `SKILLS`/`RELATED`) that has
   zero dependency on the DOM or browser storage. This is the part that
   should port almost directly into Java — `Parser.parseResume()` becomes
   `ResumeService.parse()`, `Engine.run()` becomes `MatchingEngine.match()`,
   and the taxonomy becomes seed data for the `skills` table.
2. **A UI layer** (`Store`, views, router) that is intentionally
   framework-agnostic vanilla JS so the prototype runs with zero setup. This
   layer is what gets replaced by React + Axios calls to a real API — the
   view functions already return exactly the shape of data a React component
   would expect.

That separation is why the in-app **API & schema** page can already show you
believable REST endpoints and a MySQL schema: the logic layer was written
*as if* it were already talking to that backend.

---

## Run it locally

```bash
# 1. unzip and enter the project
unzip careerlens-ai.zip
cd careerlens-ai

# 2. serve it (avoids file:// restrictions in some browsers)
python3 -m http.server 8000
# or: npx serve .

# 3. open
http://localhost:8000
```

No install step, no `npm install` — it's static HTML.

## Deploy to Vercel

```bash
npm i -g vercel
cd careerlens-ai
vercel        # preview deploy
vercel --prod # production deploy
```

Vercel auto-detects this as a static site. No `vercel.json` needed.

## Push to GitHub

```bash
cd careerlens-ai
git init
git add .
git commit -m "Initial commit: CareerLens AI frontend prototype"
git branch -M main
git remote add origin https://github.com/<your-username>/careerlens-ai.git
git push -u origin main
```

Then import that repo at vercel.com/new for automatic deploys on every push.

---

## Design decisions & known limitations

- **Everything client-side, on purpose.** This lets the whole product be
  evaluated and demoed with zero backend setup. Nothing is uploaded
  anywhere — parsing happens entirely in your browser.
- **localStorage, not a database.** Data is per-browser and per-device. It
  will not survive clearing site data, and won't sync across devices. This
  is a stand-in for MySQL, not a replacement for it.
- **The password hash is a simplified placeholder**, not a cryptographic
  hash. It's adequate for a demo where nothing sensitive is at stake, but
  must be replaced with BCrypt (or similar) server-side before this touches
  real user credentials.
- **Scanned PDFs won't parse.** If a PDF is an image of text rather than
  real text, there's nothing for pdf.js to extract — paste the resume text
  instead.
- **The skill taxonomy is finite.** ~60 skills with aliases. A skill named
  in a way the taxonomy doesn't recognize will be missed — which, worth
  noting, mirrors how many real-world Applicant Tracking Systems behave.
- **Semantic matching is a lightweight cosine similarity**, not embeddings
  from an LLM. It's shown as a supplementary signal but doesn't feed the
  weighted score, precisely so scoring stays deterministic and explainable.

## Roadmap

- [ ] Spring Boot backend (`MatchingEngine.java`, JPA entities, JWT auth)
      with a matching REST API, using the schema already documented in-app
- [ ] MySQL persistence layer replacing localStorage
- [ ] Real password hashing (BCrypt) and refresh tokens
- [ ] LLM-backed embeddings for true semantic similarity
- [ ] Expandable/editable skill taxonomy (admin-managed, not hardcoded)
- [ ] Multi-file frontend structure (split CSS/JS/views) if/when a build
      step is introduced
