# Sprint 0 Checklist — Before You Write Code

## Quick Reference
```
[x] GitHub PAT with models scope → saved locally (add to .env next)
[x] Supabase project created → 3 env vars copied
[x] venture-studio repo created + cloned locally
[x] commit.gpgsign false set on new repo
[x] node --version confirms v18+ (running v22)
[ ] Create .env file with all saved values
[ ] Run schema.sql in Supabase SQL editor
[ ] Add SUPABASE_URL + SUPABASE_SERVICE_ROLE_KEY as GitHub Actions secrets
[ ] npm run pipeline → confirm first run works
```

---

## ✅ Step 1 — GitHub Personal Access Token (DONE)
Token generated with `models: read` scope. Saved locally — add to `.env` in next step.

---

## ✅ Step 2 — Supabase Account (DONE)
Project `venture-studio` created. Three values saved locally:
- `SUPABASE_URL`
- `SUPABASE_ANON_KEY`
- `SUPABASE_SERVICE_ROLE_KEY`

---

## ✅ Step 3 — Create the Code Repo (DONE)
Repo `venture-studio` created on GitHub, cloned locally, `commit.gpgsign false` set.

---

## ✅ Step 4 — Verify Node.js (DONE)
Running Node v22.14.0 — well above v18 requirement.

---

## ⬜ Step 5 — Create .env File
Create the real `.env` file in `venture-studio/` with all saved values:
```bash
cd "/Users/tylerhuffman/Documents/DEV PROJECTS/thuff-idea-lab/venture-studio"
cp .env.example .env
# then fill in real values
```

Values to paste in:
- `GITHUB_TOKEN` — your GitHub PAT
- `SUPABASE_URL` — your project URL
- `SUPABASE_ANON_KEY` — your publishable key
- `SUPABASE_SERVICE_ROLE_KEY` — your secret key

---

## ⬜ Step 6 — Run Database Schema in Supabase
1. Go to **supabase.com → your venture-studio project → SQL Editor**
2. Paste the contents of `venture-studio/prisma/schema.sql`
3. Click **Run**
4. Verify tables appear: `ideas`, `evaluations`, `projects`

---

## ⬜ Step 7 — Add GitHub Actions Secrets
So the nightly pipeline can connect to Supabase when it runs in the cloud:
1. Go to **github.com/thuff-idea-lab/venture-studio → Settings → Secrets and variables → Actions**
2. Add two secrets:

| Secret Name | Value |
|-------------|-------|
| `SUPABASE_URL` | your project URL |
| `SUPABASE_SERVICE_ROLE_KEY` | your secret key |

(`GITHUB_TOKEN` is automatic — no setup needed)

---

## ⬜ Step 8 — First Pipeline Run
```bash
cd "/Users/tylerhuffman/Documents/DEV PROJECTS/thuff-idea-lab/venture-studio"
npm run pipeline
```
✅ Done when: ideas appear in your Supabase `ideas` table.

---

## Accounts to Set Up Later (Don't Do These Yet)

| Tool | When you need it | Sign up at |
|------|-----------------|-----------|
| PostHog | When first project goes live | posthog.com |
| Beehiiv | When first project launches | beehiiv.com |
| Resend | When email capture is needed | resend.com |
| Firecrawl | When Scout agent is running | firecrawl.dev |
| SerpAPI | When Evaluator needs competition scoring | serpapi.com |
| Buffer | When Distributor agent is built | buffer.com |
| Product Hunt | When first project launches | producthunt.com |
| Indie Hackers | When first project launches | indiehackers.com |

---

## Already Done
- [x] GitHub account (personal)
- [x] Vercel account
