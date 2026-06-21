# DASH by Datadog 2026 — Conference Notes
**Date:** June 10, 2026 · **Location:** Javits Center, NYC (single-day attendance)

---

## Session 1 — Datadog: Building & Operating Production AI Agents
Main-stage Datadog talk (no title slide captured); covers their `Dispatch` agent platform, eval/observability methodology, and forward-looking trends. Likely paired with their **State of AI/ML Engineering** report.

### `Dispatch` — agent deployment platform
- Dual documentation views demoed: **Human view** (rendered docs site) vs **Agent view** (raw markdown/terminal-friendly) — same content, agent-consumable format served separately.
- Persistent storage config via `dispatch.yaml`:
  ```yaml
  agent_name: my-agent
  namespace: production
  entrypoint: agent.py
  volumes:
    - name: data
      mountPath: /data
      mode: read_write_many
  ```
  Storage survives restarts/redeployments — for caching, state files, downloaded assets.

### Design principles
| Proactive over Reactive | Operationalized Agents |
|---|---|
| Background — long running | Durable execution |
| Chat is only one modality | Scalable |
| Event driven | Persistent storage + filesystem |

| Sandboxed | Guardrails |
|---|---|
| Isolated containers | MCP/Tools Proxy |
| Safe topology | AI Gateway |
| — | Network access controls |

### Eval + Observability: closing the loop
| Offline | Online | Living |
|---|---|---|
| Representative, measurable, rerunnable | Instrumentation, human-influenced | Ongoing sampling/annotation |
| Task vs. E2E | Replayable | Drift detection, release process, learning |

**Automate the loop:** dedicated `Eval & Analysis Agent(s)` automatically sample production traces, generate test cases, judge outputs against your rubric, and surface regressions before they reach users.

**LLM Observability demo:** Datadog's `LLM Observability` product shown tracing a multi-agent "Budget Guru" financial assistant — span tree included `Triage Agent` → `PII Detection Agent` → `Budgeting Agent (LLM)` → `Summarizer Agent`, with a built-in `Sensitive Data Scanner` and prompt-injection detection facet filters alongside pass/fail evaluations per span.

### "The Agent-level Bitter Lesson"
(Riff on Rich Sutton's *Bitter Lesson*)
- **Simple agent harness** — able to swap out underlying models
- **Build to rewrite** — don't over-invest in scaffolding that locks you to one model's quirks

### Memory matters
Three memory types agents need: **Episodic**, **Semantic**, **Procedural** (citing Yu et al., *"Agentic memory"* paper on short/long-term memory management for LLM agents). Pitch: a `Memory Agent` that automatically distills episodic memory into durable semantic memory, prunes stale context, and maintains a living summary of everything the agent fleet has learned.

### Looking ahead: the accelerating AI future
| Learning on the Job | Longer Running / Independent | Multimodal |
|---|---|---|
| Continual learning | Context size & compression improvements | Computer use is almost here |
| Synthetic environments | Long-horizon planning | |
| Improved memory management | Continued reasoning improvements | |

### Datadog State of AI/ML Engineering — key stats
- **Multi-model is now the norm:** orgs using 3+ LLM models grew from ~59% (Mar 2025: 19%+17%+23%) to **70%** (Mar 2026). Orgs using 6+ models alone jumped **23% → 41%** YoY.
- **LLM framework adoption doubled YoY:** organizations 9.1% → 17.9%; services 0.9% → 2.3% (Mar 2025 → Mar 2026).
- *Source: `datadoghq.com/state-of-ai-engineering/`*

---

## Session 2 — "Deterministic Until Proven Otherwise: Building AI Agents That Ship"
**Speaker:** Matthew Littlehale — `mlittlehale.co` | `matt@matt.computer` | `linkedin.com/in/matthewlittlehale` | `store.websitesoncomputers.com`

**Thesis:** *"The teams shipping real AI value aren't chasing the most advanced techniques. They're following a deliberate progression and relying on the patterns they already know."*

### Opening framing — progression, not a leap
| Anyone's Roadmap | Cursor's Roadmap (worked example) |
|---|---|
| Crawl → Walk → Run | Auto-complete → AI Coding Assistant → Coding Agent |

Quoting Abhishek Goswami (*Prompts to Production*, InfoQ 2026):
> "If my code can take a clear, unambiguous decision based on a fixed rule, there is no need for agentic reasoning."

### The 5-point framework (closing slide)
| # | Principle | Core idea |
|---|---|---|
| 01 | **Find your moat** | Identify what's actually defensible before building agentically |
| 02 | **Deterministic first** | Use fixed-rule code wherever logic is unambiguous; reserve agentic reasoning for genuine ambiguity |
| 03 | **Establish your language** | Shared vocabulary: `Agent Orchestration` (coordination logic across agents), `Agents` (autonomous, role-specific reasoners), `Tools` (APIs agents invoke) |
| 04 | **Follow your patterns** | Reuse internal conventions, existing infra, and industry-standard frameworks instead of inventing new agent paradigms per team |
| 05 | **Human-in-the-loop UX** | Keep the human as decision-maker; the agent proposes a specific, executable action — not an open-ended recommendation |

### Supporting concepts
- **Microservices parallel** for agent migration: `Identify` → `Classify` → `Build` (same decomposition discipline as a monolith-to-microservices migration, applied to capability-to-agent mapping).
- **Three foundational pieces:** Human-in-the-loop UX, Build context early, Observability from day 1.
- **Build context early** breaks into:
  - `Entity Context` — core actors (Users, Jobs, Products)
  - `Relationships` — Users↔Jobs, Users↔Products
  - `Event Logs` — action history + reasoning trace (agent-level distributed tracing)
- **UX pattern shown:** admin dashboard surfaces a deterministic, rule-based recommended action ("Send Renewal Notification — license expires in 30 days") behind an explicit `Execute Action` button. The agent never free-executes; it proposes a specific action for human confirmation.

---

## Session 3 — Agent Evals at Scale (WHOOP)
No title slide captured; case study on what broke when WHOOP scaled AI agents in production.

### When quality started to slip
- **Quality regression** — user feedback was the *only* alert signal (lagging, not leading)
- **Thin tooling** — diagnosing issues was manual and painful
- **Stopped all new feature dev** — had to pause the roadmap to fix quality

### Fix: building evals to match reality
- **Sampled ground truths** — started as one Google Sheet, ~4,000 questions. Worked initially, didn't scale; ground truths went stale.
- **End-to-end tracing** — production traces feed back into the next eval round, keeping ground truths grounded in live behavior.

**Resulting loop:** `Trace → Evaluate → Improve → Ship` — "one loop powering hundreds of reliable agents."

### Three key takeaways
1. Build for scale, *beforehand*
2. Your best eval dataset is in prod *now*
3. Define "good" the way customers will — not the way engineers assume

> Note: this is the same Offline/Online/Living eval loop Datadog described in Session 1, independently arrived at in production — strong signal it's the industry-converged pattern, not one vendor's opinion.

---

## Session 4 — Security Spotlight Theater: Indirect Prompt Injection on the Web
Live demo (exhibit hall) of an indirect prompt injection: a malicious webpage/profile embedded hidden instructions targeting an LLM agent processing the page — attempting to exfiltrate system IP, `/etc/passwd` contents, and SSH directory contents to an attacker-controlled address.

### Recap / mitigations
- Only install coding-agent extensions from trustworthy sources
- Vet and validate any repository you let a coding agent open before granting access
- Don't rely solely on built-in model protections — defense in depth required

---

## Session 5 — Main Stage Keynote: AI Maturity & Agent Sprawl (Datadog × Port.io)

### AI maturity model in engineering (4 levels)
| Level | State | Description |
|---|---|---|
| 1 | Manual | Not yet using coding assistants |
| 2 | Coding assistants | `Copilot`, `Cursor`, etc. — adopted across teams |
| 3 | Some agents across the SDLC | AI SRE agents, infra agents, security agents — **not yet at scale**, "still not fully operational" |
| 4 | Fully autonomous AI-SDLC | AI leads execution; engineering oversees and manages |

### Port.io segment — the "agent sprawl" problem
Journey arc: `Building & experimenting` → coding assistants widely adopted → **agent sprawl** (triggered "when scale begins") → `AI Native engineering` → `Autonomous`

**"Different teams build the same agents"** — duplicated effort observed across orgs:

| Team | Theme | Tools built |
|---|---|---|
| 1 | Resolution Powerhouse | SRE Agent, Ticket Resolver Agent, `PagerDuty` Workflow, Incident Skill |
| 2 | Security Guardians | Security Agent, Vuln Scan Workflow, Security Chatbot, Slack Alert App |
| 3 | Infrastructure Builders | `Terraform` Workflow, Infrastructure Agent, Cost Chatbot, Infra Skill |
| 4 | Reliability Champions | SRE Agent, Incident Skill, Slack Escalation App, On-call Workflow |

**Pitch — "Context Lake":** a unified, structured knowledge graph linking `Teams` ↔ `Services` ↔ `Domains` ↔ `Deployments` ↔ `Environments`, positioned as the fix for cross-team agent duplication (i.e., a service-catalog / metadata layer purpose-built for agent context).

---

## Cross-cutting takeaways for your track (AWS / distributed systems / MLOps)

- **The eval loop is industry-converged, not one vendor's opinion.** Datadog's `Offline/Online/Living` framework and WHOOP's `Trace → Evaluate → Improve → Ship` independently describe the same loop — structurally identical to a `SageMaker Model Monitor` + retraining pipeline. Same answer works whether the interviewer asks about LLM agents or classical ML models.
- **`Dispatch`'s Sandboxed/Guardrails model maps directly to AWS primitives:** isolated containers → `ECS/Fargate` task-level isolation; safe topology → VPC subnetting; `MCP/Tools Proxy` + `AI Gateway` → `API Gateway` in front of tool-invocation endpoints; network access controls → security groups / `PrivateLink`. Good concrete answer for "how would you sandbox an agent's tool calls" in a SWE interview.
- **"Context Lake" = service catalog problem.** Agent sprawl is a metadata-management/discovery problem, not an AI problem — directly comparable to AWS `Resource Explorer` / `Service Catalog` / an internal developer platform (IDP). Strong distributed-systems interview angle: how do you prevent N teams from re-solving the same cross-cutting concern.
- **Memory architecture → real AWS services.** Episodic memory (recent interaction state) → `DynamoDB` with TTL; semantic memory (durable knowledge) → vector store via `OpenSearch` or `Bedrock Knowledge Bases`; procedural memory (learned workflows) → versioned config/prompt store. Useful if asked to design an agent's memory layer.
- **"Agent-level Bitter Lesson" = avoid vendor lock-in at the harness level.** Architecturally this is the same argument for using `Bedrock`'s model-agnostic invocation layer over hardcoding to one model provider — directly relevant given 70% of orgs are now running 3+ models.
- **Deterministic-first + human-in-loop** maps cleanly to a hybrid orchestration architecture: `Step Functions` (deterministic, auditable control flow) wrapping `Bedrock Agents` (non-deterministic tool-use) for the genuinely ambiguous steps only.
- **Security talk → AWS SA exam relevance.** Defense-in-depth and least-privilege scoping apply directly to agentic workloads — e.g., scoping IAM for `Bedrock Agent` execution roles, restricting tool/action permissions per agent rather than granting broad access.

## Follow-ups
- [ ] Find Abhishek Goswami's "Prompts to Production" talk (InfoQ, 2026)
- [ ] Check `mlittlehale.co` for Matthew Littlehale's "Deterministic Until Proven Otherwise" deck
- [ ] Read Datadog's State of AI/ML Engineering report in full — `datadoghq.com/state-of-ai-engineering/`
- [ ] Look up the Yu et al. "Agentic memory" paper referenced in the memory-management slide
- [ ] Research Port.io's "Context Lake" architecture — compare against AWS Service Catalog / Resource Explorer
- [ ] For SWE interview prep: be ready to discuss eval/observability loop design — it's a recurring theme across three separate sessions
