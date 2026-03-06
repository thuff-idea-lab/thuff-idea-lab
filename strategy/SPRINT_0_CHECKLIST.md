# Sprint 0 Checklist — Before You Write Code

## Quick Reference
```
[ ] GitHub PAT with models scope → saved in .env
[ ] Supabase project created → 3 env vars copied
[ ] venture-studio repo created + cloned locally
[ ] commit.gpgsign false set on new repo
[ ] node --version confirms v18+
```
**Estimated time: ~15 minutes total**

---

## Step 1 — GitHub Personal Access Token (30 seconds)
Powers free AI calls via GitHub Models. Not a new account — just a token from your existing GitHub.

1. Go to **github.com → Settings → Developer Settings → Personal Access Tokens → Fine-grained tokens**
2. Click **Generate new token**
3. Set expiration: 1 year
4. Under **Permissions** → add `models: read`
5. Copy it → this becomes `GITHUB_TOKEN` in your `.env`

---

## Step 2 — Supabase Account (5 min — free)
Your database. Nothing in Phase 1 works without it.

1. Sign up at **supabase.com** with your GitHub account (one click)
2. Create a new project called `venture-studio`
3. After it spins up, grab three values from **Settings → Database**:
   - **Project URL** → `SUPABASE_URL`
   - **Anon public key** → `SUPABASE_ANON_KEY`
   - **Connection string (URI mode)** → `DATABASE_URL`

---

## Step 3 — Create the Code Repo (2 min)
The `documentation` repo exists. You need a separate repo for the actual studio code.

1. Create a new GitHub repo called `venture-studio`
2. Clone it locally:
   ```bash
   cd "/Users/tylerhuffman/Documents/DEV PROJECTS/thuff-idea-lab"
   git clone https://github.com/<your-username>/venture-studio.git
   ```
3. Immediately disable 1Password signing on the new repo:
   ```bash
   cd venture-studio
   git config --local commit.gpgsign false
   ```

---

## Step 4 — Verify Node.js (1 min)
```bash
node --version   # need v18 or higher
npm --version
```
If missing: download LTS from **nodejs.org**

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
