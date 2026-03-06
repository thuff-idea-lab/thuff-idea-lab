# AI Strategy — Venture Studio

## Core Principle
GitHub Copilot subscription already provides access to frontier models at $0 incremental cost.
We do NOT need a separate OpenAI API key.

---

## Model Access — GitHub Models API

All LLM calls route through the GitHub Models inference endpoint.
This is included with GitHub Copilot.

**Endpoint:** `https://models.inference.ai.azure.com/chat/completions`  
**Auth:** `GITHUB_TOKEN` (Personal Access Token with models scope)  
**Available Models:** gpt-4o-mini, gpt-4o, gpt-4.1, gpt-5-mini

### Rate Limits (Free Tier)
- ~15 requests/minute
- ~150 requests/day per model
- Sufficient for nightly pipeline on 20–30 ideas

---

## The Abstraction Layer — `studio/lib/llm.ts`

ALL agent LLM calls go through this single file.
No agent should ever call an AI API directly.

### Why
- Swap providers with one env var change
- Add fallback logic in one place
- Test agents without real API calls (mock this file)
- Future-proof if GitHub Models limits become a bottleneck

### Provider Priority
1. `github` — Default, free with Copilot (use this always first)
2. `ollama` — Local fallback, zero cost, lower quality
3. `anthropic` — Premium fallback, only when generating revenue

### Environment Variable
```bash
# .env
LLM_PROVIDER=github          # github | anthropic | ollama
GITHUB_TOKEN=your_pat_here
```

### Implementation Pattern
```typescript
// studio/lib/llm.ts
type LLMProvider = 'github' | 'anthropic' | 'ollama';

interface LLMOptions {
  model?: string;
  temperature?: number;
  provider?: LLMProvider;
}

export async function callLLM(
  prompt: string,
  options: LLMOptions = {}
): Promise<string> {
  const provider = options.provider
    ?? (process.env.LLM_PROVIDER as LLMProvider)
    ?? 'github';

  switch (provider) {
    case 'github':   return callGitHubModels(prompt, options);
    case 'anthropic': return callClaude(prompt, options);
    case 'ollama':   return callOllama(prompt, options);
    default:         throw new Error(`Unknown provider: ${provider}`);
  }
}

async function callGitHubModels(prompt: string, options: LLMOptions): Promise<string> {
  const res = await fetch('https://models.inference.ai.azure.com/chat/completions', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${process.env.GITHUB_TOKEN}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      model: options.model ?? 'gpt-4o-mini',
      messages: [{ role: 'user', content: prompt }],
      temperature: options.temperature ?? 0.3,
    }),
  });
  const data = await res.json();
  return data.choices[0].message.content;
}

async function callClaude(prompt: string, options: LLMOptions): Promise<string> {
  // Swap in later when needed
  throw new Error('Anthropic provider not configured yet');
}

async function callOllama(prompt: string, options: LLMOptions): Promise<string> {
  const res = await fetch('http://localhost:11434/api/generate', {
    method: 'POST',
    body: JSON.stringify({ model: options.model ?? 'llama3.2', prompt }),
  });
  const data = await res.json();
  return data.response;
}
```

---

## Which Agents Actually Need LLM

| Agent | Needs LLM? | Why |
|-------|-----------|-----|
| Scout | ❌ No | Pure scraping + RSS |
| Evaluator | ⚠️ Optional | Rule-based covers 80% |
| Planner | ✅ Yes | Free-form plan generation |
| Builder | ✅ Yes | Scaffold + content generation |
| Distributor | ✅ Yes | Post/copy drafting |
| Analytics | ❌ No | Math + thresholds |

---

## VS Code Copilot Skills (Phase 2)

Build a custom `@studio` chat participant in VS Code.
This lets you interact with your studio without leaving the editor.

### Planned Skills
- `@studio run scout` — trigger scout agent
- `@studio status` — show all active projects
- `@studio evaluate [idea]` — score an idea on demand
- `@studio what should I build next?` — query top scored ideas

### Location
`venture-studio/.vscode/studio-extension/`

---

## When to Add Claude (Anthropic)

Only revisit when:
- GitHub Models rate limits are a real daily bottleneck
- A project is generating enough revenue to justify the cost
- Structured JSON output quality becomes a measurable problem

**Target model:** Claude 3.5 Haiku (~$1–3/month at studio scale)  
**Swap effort:** Change `LLM_PROVIDER=anthropic` in `.env` + implement `callClaude()` in `lib/llm.ts`
