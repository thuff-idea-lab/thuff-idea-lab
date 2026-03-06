# AI Internet Venture Studio — Repo Architecture (Copilot-Friendly)

This document is the **single source of truth** for building the Venture Studio in VS Code with GitHub Copilot.

## Goal
A portfolio system of AI agents that continuously:
1) discover opportunities
2) evaluate/prioritize
3) generate project plans
4) build MVPs
5) test traction
6) scale winners / kill losers

Supports **any internet business asset** (websites, directories, SaaS, YouTube channels, trading bots, newsletters, data products, etc.).

---

## 0) High-level decisions
- **Monorepo** for the venture studio “control plane” + per-project folders.
- Agents communicate via **typed JSON contracts** stored in the database and as files in `/studio/out/`.
- Start with a **Minimum Viable Studio** (3 agents): Scout → Evaluate → Plan. Add Board/Build/Test/Scale after.
- The full pipeline is: Scout → Evaluate → **Board** → Plan → Build → Test → Scale/Kill.
- Keep everything deterministic + testable: agents should be small scripts with clear inputs/outputs.

---

## 1) Folder structure

```
venture-studio/
  README.md
  .env.example
  package.json
  tsconfig.json

  docs/
    VENTURE_STUDIO_CANVAS.md
    AGENT_CONTRACTS.md
    RUNBOOK.md
    AI_STRATEGY.md           # LLM strategy, GitHub Models, lib/llm.ts
    TOOLS_AND_INTEGRATIONS.md # Every free tool + when to use it
    VALIDATION_STRATEGY.md   # How to validate ideas without Reddit risk

  studio/
    config/
      niches.json              # optional: constraints/tags (not required)
      sources.json             # where Scout pulls from (reddit feeds, rss, etc.)
      scoring_weights.json     # evaluator scoring weights

    contracts/
      idea.schema.json
      evaluation.schema.json
      board_decision.schema.json
      project.schema.json
      mvp.schema.json
      experiment.schema.json
      observatory_report.schema.json

    agents/
      scout/
        index.ts
        prompts.ts
        sources/
          reddit.ts
          trends.ts
          forums.ts

      evaluator/
        index.ts
        scoring.ts

      planner/
        index.ts
        project_types.ts

      builder/                 # phase 2
        index.ts
        templates/
          nextjs-tool/
          directory-shell/
          landing-page/
          newsletter-kit/
          youtube-kit/
          trading-bot-kit/

      distributor/             # phase 2
        index.ts
        channels/
          reddit.ts
          x.ts
          email.ts

      analytics/               # phase 2
        index.ts
        metrics.ts

      expander/                # phase 3
        index.ts
        strategies/

      board/                   # phase 2 — deliberation gate between Evaluator and Planner
        index.ts               # orchestrates the board session
        ceo.ts                 # strategic fit, portfolio balance
        cmo.ts                 # distribution, channels, audience
        coo.ts                 # build complexity, timelines, risks
        cfo.ts                 # unit economics, CAC, LTV, margins
        deliberation.ts        # synthesis + vote tallying → GO | NO_GO | WATCH

      observatory/             # phase 3 — self-improving quality loop
        index.ts
        quality_scorer.ts      # scores agent outputs over time
        prompt_versioner.ts    # tracks prompt versions + outcome correlation
        drift_detector.ts      # alerts when agents degrade or repeat
        improvement_report.ts  # surfaces suggested prompt changes for human review

    lib/
      llm.ts                   # GitHub Models/local routing (see docs/AI_STRATEGY.md)
      db.ts                    # Prisma client
      logger.ts
      validate.ts              # JSON schema validation
      urls.ts
      time.ts

    out/
      ideas.ndjson
      evaluations.ndjson
      board_decisions.ndjson
      projects.ndjson
      mvp.ndjson
      experiments.ndjson
      observatory_reports.ndjson

  apps/
    dashboard/
      (Next.js admin UI)

  projects/
    _templates/
      web_tool_nextjs/
      directory_nextjs/
      youtube_channel/
      newsletter/
      trading_bot/
      micro_saas/

    active/
      <project-slug>/
        project.json
        notes.md
        mvp/
        build/

    archived/
      <project-slug>/

  prisma/
    schema.prisma
    migrations/

  scripts/
    run-agent.ts               # run one agent
    run-pipeline.ts            # run Scout→Evaluate→Board→Plan
    run-board.ts               # run Board deliberation only
    run-observatory.ts         # run Observatory quality report
    promote-project.ts         # move from lab to “active”
    archive-project.ts

  .github/
    workflows/
      scout.yml
      evaluate.yml
      board.yml
      plan.yml
      nightly-pipeline.yml
      observatory.yml
```

---

## 2) The “Minimum Viable Venture Studio” (start here)

### Phase 1 (Weekend build)
- ✅ Scout Agent (discover)
- ✅ Evaluator Agent (score)
- ✅ Planner Agent (choose project type + draft build plan)
- ✅ Dashboard (simple table view)

### Phase 2 (Next)
- Builder Agent (generate MVP scaffold in `/projects/active/<slug>`)
- Distributor Agent (draft posts + optional ad kit)
- Analytics Agent (score traction)
- **Board of Agents** — CEO, CMO, COO, CFO deliberation gate between Evaluator and Planner
  - Only `WATCH`/`BUILD` ideas go to the Board for a GO / NO_GO / WATCH vote
  - Each persona has a constrained system prompt (deliberately blind to other domains until synthesis)
  - Grounded in hard data from DB — not opinion, not creative writing

### Phase 3 (Scale)
- Expander Agent (clone/expand winners)
- Spinout Agent (split repo/domain)
- **Observatory Agent** — monitors all agent output quality over time
  - Scores output signal/noise per agent per run
  - Detects drift (e.g., Scout recycling same ideas for weeks)
  - Versions prompts and correlates versions with outcome quality
  - Surfaces improvement suggestions for human review before any prompt is changed
  - Does **not** self-modify autonomously — human approval required

---

## 3) Data model (Prisma)

### Tables
- `Idea`
  - `id`, `title`, `summary`, `evidence`, `sources[]`, `keywords[]`, `tags[]`, `assetTypeHint`, `createdAt`
- `Evaluation`
  - `id`, `ideaId`, `scoreTotal`, `scoreBreakdown(json)`, `recommendation` (DROP | WATCH | BUILD), `notes`, `createdAt`
- `Project`
  - `id`, `ideaId`, `slug`, `projectType`, `status` (PLANNED | BUILDING | LIVE | PAUSED | ARCHIVED), `plan(json)`, `createdAt`
- `MVP`
  - `id`, `projectId`, `spec(json)`, `repoPath`, `deployTarget`, `createdAt`
- `Experiment`
  - `id`, `projectId`, `channel` (REDDIT | X | ADS | EMAIL | SEO), `setup(json)`, `metrics(json)`, `result` (KILL | ITERATE | SCALE)
- `BoardDecision`
  - `id`, `evaluationId`, `ceoPov(json)`, `cmoPov(json)`, `cooPov(json)`, `cfoPov(json)`, `verdict` (GO | NO_GO | WATCH), `rationale`, `createdAt`
- `ObservatoryReport`
  - `id`, `runAt`, `agentScores(json)`, `driftAlerts(json)`, `promptVersions(json)`, `suggestedImprovements(json)`, `approvedAt`

### Key principle
Everything an agent outputs must be stored as structured JSON in DB **and** optionally in `/studio/out/*.ndjson` for debugging.

---

## 4) Agent contracts (JSON)

### 4.1 Idea contract (`IdeaRecord`)
```json
{
  "title": "string",
  "summary": "string",
  "evidence": [{"type":"link|metric|quote","value":"string"}],
  "sources": [{"platform":"string","url":"string","context":"string"}],
  "keywords": ["string"],
  "tags": ["string"],
  "assetTypeHint": "website|directory|saas|youtube|newsletter|bot|data|unknown"
}
```

### 4.2 Evaluation contract (`EvaluationRecord`)
```json
{
  "ideaTitle": "string",
  "scoreTotal": 0,
  "scoreBreakdown": {
    "demand": 0,
    "monetization": 0,
    "competition": 0,
    "automationPotential": 0,
    "buildComplexity": 0
  },
  "recommendation": "DROP|WATCH|BUILD",
  "notes": "string"
}
```

### 4.3 Board decision contract (`BoardDecisionRecord`)
```json
{
  "evaluationId": "string",
  "ideaTitle": "string",
  "ceoPov": {
    "strategicFit": "string",
    "portfolioBalance": "string",
    "vote": "GO|NO_GO|WATCH"
  },
  "cmoPov": {
    "targetAudience": "string",
    "distributionChannels": ["string"],
    "marketingRisk": "string",
    "vote": "GO|NO_GO|WATCH"
  },
  "cooPov": {
    "buildComplexity": "LOW|MEDIUM|HIGH",
    "estimatedHours": 0,
    "topRisks": ["string"],
    "vote": "GO|NO_GO|WATCH"
  },
  "cfoPov": {
    "estimatedCAC": "string",
    "revenueModel": "string",
    "breakEvenTimeline": "string",
    "vote": "GO|NO_GO|WATCH"
  },
  "verdict": "GO|NO_GO|WATCH",
  "rationale": "string"
}
```

### 4.4 Project contract (`ProjectRecord`)
```json
{
  "slug": "string",
  "projectType": "website|directory|saas|youtube|newsletter|bot|data",
  "oneSentencePitch": "string",
  "targetUser": "string",
  "primaryMonetization": ["ads|affiliate|leadgen|subscription|product|data"],
  "mvpDefinition": {
    "deliverables": ["string"],
    "timeboxHours": 6,
    "successMetric": "string"
  },
  "buildPlan": {
    "steps": ["string"],
    "tech": ["string"],
    "risks": ["string"],
    "constraints": ["string"]
  },
  "testPlan": {
    "channels": ["REDDIT","X","ADS","EMAIL","SEO"],
    "budget": 0,
    "tracking": ["string"],
    "goNoGo": ["string"]
  }
}
```

---

## 5) How Copilot should be used (practical)

### Rule
**Never** paste long chat threads into Copilot.

### Use this doc as the anchor
Prompt Copilot like:
- “Read `/docs/VENTURE_STUDIO_CANVAS.md` and implement the Prisma schema.”
- “Using the `IdeaRecord` schema in `/studio/contracts/idea.schema.json`, implement `studio/agents/scout/index.ts`.”
- “Implement `scripts/run-pipeline.ts` to run Scout→Evaluator→Planner and write outputs to DB + `/studio/out/*.ndjson`.”

### Work file-by-file
Copilot performs best when each task is a single file with a clear contract.

---

## 6) Execution: scripts you can run

### Run a single agent
```
node scripts/run-agent.ts scout
node scripts/run-agent.ts evaluator
node scripts/run-agent.ts board
node scripts/run-agent.ts planner
node scripts/run-agent.ts observatory
```

### Run the pipeline
```
node scripts/run-pipeline.ts
```

### Run the board deliberation only
```
node scripts/run-board.ts
```

### Run the observatory report
```
node scripts/run-observatory.ts
```

---

## 7) GitHub Actions schedules

- `scout.yml` — daily
- `evaluate.yml` — daily
- `board.yml` — daily (runs after evaluate; gates Planner)
- `plan.yml` — daily (only processes `GO` verdicts)
- `nightly-pipeline.yml` — nightly end-to-end
- `observatory.yml` — weekly (quality report + drift detection)

Each workflow:
- installs deps
- runs the script
- stores logs/artifacts

---

## 8) “Anything goes” without SEO chaos

This studio is **not** one public website.
It’s a portfolio generator.

- Some outputs become websites.
- Some become directories.
- Some become channels.
- Some become bots.

The **studio dashboard** is the control center; each project lives in `/projects/active/<slug>`.

---

## 9) Next build steps (do in order)

### Step 1 — Repo + DB
1) Create the folders above
2) Add Prisma schema + migrations
3) Add `studio/lib/db.ts`

### Step 2 — Scout Agent (MVP)
- Start with 1–2 sources (e.g., Reddit RSS + Google Trends)
- Output `IdeaRecord` entries

### Step 3 — Evaluator Agent
- Read new Ideas from DB
- Produce Evaluation records

### Step 4 — Planner Agent
- For `GO` board verdicts, generate `ProjectRecord`
- Create `/projects/active/<slug>/project.json`

### Step 5 — Dashboard
- Show ideas, scores, board decisions, projects
- Add "Approve" / "Archive" buttons

### Step 6 — Board of Agents (Phase 2)
- Implement `studio/agents/board/` with CEO, CMO, COO, CFO personas
- Each persona has a **constrained system prompt** — it only reasons within its domain
- `deliberation.ts` synthesizes the 4 views and outputs a `BoardDecisionRecord`
- Insert Board as a gate: Evaluator → Board → Planner
- Pipeline only continues to Planner for `GO` verdicts
- Wire `run-board.ts` script + `board.yml` GitHub Action

### Step 7 — Observatory Agent (Phase 3)
- Implement `studio/agents/observatory/`
- Score each agent's output quality weekly (signal/noise, uniqueness, outcome correlation)
- Version all prompts and track which versions produce better GO→LIVE conversion
- Generate `ObservatoryReport` with drift alerts and improvement suggestions
- **Human approval required** before any prompt change is applied
- Wire `run-observatory.ts` script + `observatory.yml` GitHub Action

---

## 10) Quality guardrails
- Every agent output must validate against its JSON schema.
- Prefer deterministic templates over “free-form” generation.
- Log minimally; store raw outputs for debugging.- **Board agents must be grounded in DB data** — no persona reasoning without real numbers (actual scores, benchmarks, prior outcomes). Without this, the board is just expensive creative writing.
- **Observatory never self-modifies** — it surfaces suggestions only. A human must approve and apply any prompt changes. Autonomous self-modification is explicitly out of scope.
- **Board personas must be constrained** — each persona's system prompt is scoped to its domain only. If all four agents can see all dimensions, they converge to agreement (LLM confirmation bias × 4).
---

## 11) Definition of done (Phase 1)
You are done with Phase 1 when:
- The nightly pipeline runs on schedule
- New ideas appear daily
- They get scored
- A few become planned projects with a clear MVP + test plan
- You can review everything in the dashboard

---

## 12) Strategy docs (read before building)

| Doc | What it covers |
|-----|---------------|
| [`strategy/AI_STRATEGY.md`](./strategy/AI_STRATEGY.md) | LLM approach, GitHub Models, `lib/llm.ts`, zero API cost |
| [`strategy/TOOLS_AND_INTEGRATIONS.md`](./strategy/TOOLS_AND_INTEGRATIONS.md) | Every free tool, when to use it, env vars |
| [`strategy/VALIDATION_STRATEGY.md`](./strategy/VALIDATION_STRATEGY.md) | How to validate ideas without Reddit risk |
| [`strategy/SPRINT_0_CHECKLIST.md`](./strategy/SPRINT_0_CHECKLIST.md) | Accounts + admin steps before writing any code |
| [`strategy/AGENT_CONTRACTS.md`](./strategy/AGENT_CONTRACTS.md) | JSON schemas for all agent I/O |
| [`strategy/RUNBOOK.md`](./strategy/RUNBOOK.md) | How to operate the studio day-to-day |

