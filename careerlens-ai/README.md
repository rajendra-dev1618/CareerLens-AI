# CareerLens AI — Resume & Job Matching Platform

A single-file, client-side working build of CareerLens AI: upload a resume
(PDF/DOCX/TXT), save a job description, and get a weighted compatibility
score, skill-gap analysis and recommendations — all computed in the browser.

This is the frontend prototype. It mirrors the intended Spring Boot + MySQL
architecture one-to-one (see the in-app **API & schema** page), so the
browser-side Store/Auth/Parser/Engine modules can be swapped for real API
calls without changing the UI.

## Run it locally

No build step, no dependencies to install. It's one HTML file.

**Option A — just open it**
```
open index.html          # macOS
xdg-open index.html      # Linux
start index.html         # Windows
```

**Option B — serve it (recommended, avoids browser file:// restrictions)**
```bash
# Python (built in on most systems)
python3 -m http.server 8000
# then open http://localhost:8000

# or Node
npx serve .
```

Data (accounts, resumes, jobs, analyses) is stored in your browser's
localStorage — nothing is uploaded anywhere.

## Deploy to Vercel

```bash
npm i -g vercel
cd careerlens-ai
vercel
```

Vercel auto-detects this as a static site — no `vercel.json` needed.

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

Then on vercel.com, "Import Project" from that GitHub repo for automatic
deploys on every push.

## Stack

- Vanilla JS, no framework, no build tooling
- `pdf.js` (PDF text extraction) and `mammoth.js` (DOCX extraction) via CDN
- Skill taxonomy + weighted matching engine (required 50% / preferred 15% /
  experience 15% / education 10% / projects 10%)

## Next step: real backend

The in-app **API & schema** page documents the REST endpoints, MySQL schema
and `/api/match` response shape this was designed against, for when you build
the Spring Boot service.
