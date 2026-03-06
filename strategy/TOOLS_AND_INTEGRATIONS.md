# Tools & Integrations — Venture Studio

## Core Principle
The studio is an **orchestrator** of free tools, not a builder of things free tools already do.
If a free tool does it, we call it. We don't replicate it.

---

## Full Zero-Cost Stack

```
Scout:       Reddit RSS + HN API + Google Trends + Jina Reader + Firecrawl
Evaluator:   SerpAPI (100/mo free) + rule-based scoring
Planner:     GitHub Models gpt-4o-mini (free with Copilot)
Builder:     v0.dev + Vercel + Supabase
Distributor: Buffer + Beehiiv + Typefully
Analytics:   PostHog + Vercel Analytics
─────────────────────────────────────────────────────────────
Total cost:  $0/month until generating revenue
```

---

## Scout Agent Tools

| Tool | Purpose | Cost | Notes |
|------|---------|------|-------|
| Reddit RSS | Read-only demand signals | Free | `reddit.com/r/{sub}/new/.rss` — READ ONLY, never post |
| HN Algolia API | Tech community signals | Free | `hn.algolia.com/api/v1` |
| Google Trends | Rising interest | Free | `trends.google.com/trends/api` |
| Product Hunt API | Launch gaps | Free | `api.producthunt.com/v2` |
| Jina AI Reader | Clean scrape any URL | Free, no key | `r.jina.ai/{url}` |
| Firecrawl | Deep scrape + markdown | Free tier | `api.firecrawl.dev` |
| Exploding Topics | Pre-viral trends | Free tier | Manual check initially |
| GitHub Trending | Dev tool gaps | Free | `github.com/trending` RSS |

### Jina AI — Zero Setup Scraping (No API Key)
```typescript
// studio/agents/scout/sources/jina.ts
export async function readUrl(url: string): Promise<string> {
  const res = await fetch(`https://r.jina.ai/${url}`);
  return res.text(); // Returns clean markdown of any webpage
}
```

### Firecrawl — Competitor Deep Scraping
```typescript
// studio/agents/scout/sources/firecrawl.ts
export async function scrapeCompetitor(url: string) {
  const res = await fetch('https://api.firecrawl.dev/v1/scrape', {
    method: 'POST',
    headers: { 'Authorization': `Bearer ${process.env.FIRECRAWL_API_KEY}` },
    body: JSON.stringify({ url, formats: ['markdown'] }),
  });
  return res.json(); // Clean markdown — feed directly to Evaluator LLM
}
```

---

## Evaluator Agent Tools

| Tool | Purpose | Cost |
|------|---------|------|
| SerpAPI | Google SERP competition check | 100 searches/month free |
| Ahrefs Webmaster Tools | SEO difficulty (your own domains) | Free |
| Google Search Console | Real impression data | Free |
| Similarweb | Competitor traffic estimates | Free tier |

---

## Builder Agent Tools

| Tool | Purpose | Cost |
|------|---------|------|
| v0.dev | Generate full UI from prompt | Free tier |
| Shadcn/ui | Component library | Free |
| Vercel | Deploy Next.js | Free |
| Supabase | Postgres + Auth + Storage | Free tier |
| Cloudflare Pages | Static site hosting | Free |
| Railway | Backend hosting | $5 credit/month free |
| Resend | Transactional email | 3,000 emails/month free |

---

## Distributor Agent Tools

| Tool | Purpose | Cost |
|------|---------|------|
| Buffer | Schedule social posts | Free tier |
| Typefully | X/Twitter thread scheduling | Free tier |
| Beehiiv | Newsletter platform | Free to 2,500 subscribers |
| ConvertKit | Email list + automations | Free to 1,000 subscribers |
| Indie Hackers | Builder community posts | Free |
| Product Hunt | Launch platform | Free |
| DEV.to / Hashnode | Developer audience | Free |

---

## Analytics Agent Tools

| Tool | Purpose | Cost |
|------|---------|------|
| PostHog | Full product analytics | Free to 1M events/month |
| Vercel Analytics | Web traffic | Free with Vercel |
| Plausible | Privacy-friendly analytics | Free self-hosted |
| Umami | Simple web analytics | Free self-hosted |

### PostHog Setup (Add to Every Project at Launch)
```typescript
// projects/active/<slug>/mvp/lib/analytics.ts
import PostHog from 'posthog-node';

const client = new PostHog(process.env.POSTHOG_KEY!);

export function trackEvent(projectSlug: string, event: string, props?: object) {
  client.capture({ distinctId: projectSlug, event, properties: props });
}
```

---

## Environment Variables Reference

```bash
# .env.example

# AI — GitHub Models (free with Copilot)
LLM_PROVIDER=github
GITHUB_TOKEN=

# Scraping
FIRECRAWL_API_KEY=
SERP_API_KEY=

# Infrastructure
SUPABASE_URL=
SUPABASE_ANON_KEY=
DATABASE_URL=

# Email
RESEND_API_KEY=

# Analytics
POSTHOG_KEY=

# Distribution
BUFFER_ACCESS_TOKEN=
PRODUCT_HUNT_API_TOKEN=
BEEHIIV_API_KEY=
```

---

## Integration Priority (Build in This Order)

| Priority | Tool | Why First |
|----------|------|-----------|
| 1 | **Jina AI Reader** | Zero setup, immediate Scout value |
| 2 | **Supabase** | Database — needed for everything |
| 3 | **PostHog** | Analytics on every project from day one |
| 4 | **Vercel** | Deployment for all projects |
| 5 | **Beehiiv** | Email list on every project at launch |
| 6 | **Firecrawl** | Competitor research for Evaluator |
| 7 | **SerpAPI** | Competition scoring |
| 8 | **Buffer** | Social scheduling when Distributor is ready |
