# Validation Strategy — Venture Studio

## Core Rule
**Reddit is a data source. It is never a distribution channel.**

Shadow bans happen fast. Promotional accounts are flagged within the first post.
We read Reddit via RSS only. We never post to Reddit from the studio.

---

## Validation Hierarchy

### Tier 1 — Passive Signals (Zero Risk, Zero Account Needed)
These run automatically inside the Scout agent:

| Signal | What it proves | Source |
|--------|---------------|--------|
| Google search volume | Active demand exists | SerpAPI / Google Trends |
| Reddit post engagement | Community cares (read-only) | RSS feed |
| HN Ask/Show engagement | Tech audience interest | Algolia API |
| Amazon book sales rank | People pay for this topic | Scrape |
| Udemy enrollment count | People pay to learn this | Scrape |
| App Store review complaints | Gap in existing solutions | Scrape |
| YouTube search suggestions | Content demand | Google Trends |

**Strongest signal: people already paying for adjacent solutions.**

---

### Tier 2 — Low Risk Posting (Human-Reviewed Before Anything Goes Live)
Distributor agent *drafts* these. You *approve* before posting.

| Channel | Risk | Why Safe |
|---------|------|---------|
| Indie Hackers | Very Low | Built for builders, rewards genuine work |
| Hacker News Show HN | Low | Rewards genuine builders |
| Product Hunt | Very Low | Expected promotional behavior |
| LinkedIn | Low | Professional identity protects you |
| DEV.to / Hashnode | Very Low | Builder community |
| Niche Discord servers | Low | Smaller, community-driven |
| Niche Facebook Groups | Medium | Slower to ban, more forgiving |

---

### Tier 3 — Strongest Validation (Before Building Full MVP)

#### Waitlist Landing Page (Default First Step)
- Single page, email capture, deployed to Vercel in 30 minutes
- Builder agent scaffolds this **before** the full MVP
- **Gate:** 50 signups = build it. Under 50 = kill or rethink positioning.

#### Google Ads Micro-Test ($20 max budget)
- Write 3 headline variants targeting core keyword
- Point to waitlist landing page
- Run for 48 hours
- **Gate:** CTR > 3% AND cost-per-signup < $5 = build it

---

## Revised Pipeline with Validation Gate

```
Scout finds signal (passive, automated)
        ↓
Evaluator scores demand confidence (automated)
        ↓
Planner generates plan (automated)
        ↓
You approve in dashboard                     ← 2 min of your time
        ↓
Builder scaffolds LANDING PAGE ONLY (not full MVP)
        ↓
Distributor drafts post for Indie Hackers / HN
        ↓
You review + post                            ← 5 min of your time
        ↓
Analytics watches email signups for 7 days
        ↓
≥50 signups → Builder scaffolds full MVP
<50 signups → Archive, move on
```

---

## Validation Gates by Asset Type

| Asset Type | Validation Gate | Kill Threshold |
|-----------|----------------|----------------|
| Directory / content site | 100 organic visitors in 7 days | < 25 visitors |
| SaaS tool | 50 waitlist signups | < 15 signups |
| Newsletter | 100 subscribers before issue 3 | < 30 subscribers |
| Affiliate site | 1 affiliate click in first 7 days | Zero clicks |
| Trading bot | Profitable on paper trading for 30 days | Any live money before paper proof |

---

## Distribution Channels by Phase

### Phase 1 — Pre-Revenue (Use These)
- Indie Hackers
- Hacker News Show HN
- Product Hunt
- DEV.to / Hashnode
- LinkedIn (personal)
- Niche Discord servers

### Phase 2 — Post-Traction (Earn These)
- Newsletter (build via Beehiiv — start day one, monetize later)
- SEO (start technical setup day one, results in 3–6 months)
- X/Twitter (build audience over time)

---

## The Key Mindset

> Don't use Reddit to **create** demand signals.
> Use Reddit (read-only) to **find** demand signals that already exist.

The upvotes are already there. You don't need to be the one posting.
You just need to be the one reading.

**Reddit as a data source = powerful.**  
**Reddit as a distribution channel = fragile.**
