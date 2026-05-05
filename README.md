# Competitor Research Agent

A Claude Code harness for **intensive competitor research**. One user-invocable skill
orchestrates four specialist subagents in parallel to produce a tight, falsifiable
competitive landscape.

## How it works

```
User: "Run competitor research on Notion"
        │
        ▼
Claude reads available skills → matches "competitor-research"
        │
        ▼
┌─────────────────────────────────────────────┐
│  SKILL: competitor-research                 │
│  (this is just instructions Claude follows) │
│                                             │
│  Step 1: Ask scoping question               │
│  Step 2: WebSearch to pick 3–5 competitors  │
│  Step 3: For EACH competitor, spawn 4       │
│          subagents in parallel ──────────┐  │
│  Step 4: Collect their reports           │  │
│  Step 5: Synthesize 6-bullet cards       │  │
│  Step 6: Write Gap Analysis              │  │
│  Step 7: Ask the closing question        │  │
└──────────────────────────────────────────┼──┘
                                           │
            ┌──────────────────────────────┘
            ▼ (all 4 fire at once, per competitor)
   ┌────────────────┬────────────────┬────────────────┬─────────────────────┐
   │ research-agent │ pricing-analyst│ review-miner   │ positioning-mapper  │
   │ (subagent)     │ (subagent)     │ (subagent)     │ (subagent)          │
   └────────────────┴────────────────┴────────────────┴─────────────────────┘
        │                │                │                │
        ▼                ▼                ▼                ▼
   Each runs in its own context window, does WebSearch/WebFetch,
   returns a structured report back to the skill orchestrator.
```

## Skill vs. subagent — the mental model

- **Skill** = a *playbook* the main Claude reads and follows. It's not a separate
  process; it's instructions loaded into the current conversation.
- **Subagent** = an *actual separate Claude instance* with its own context window,
  tool access, and remit. The skill spawns them via the `Agent` tool.

The user only ever talks to the **skill**. The subagents are invisible — they just
return structured reports the skill synthesizes.

## Does the skill ask questions?

Yes, but bounded:

| Scenario                          | Behavior                                              |
| --------------------------------- | ----------------------------------------------------- |
| User gives no context             | Asks one scoping question: product + audience         |
| User already provided both        | Skips the question, confirms in one line              |
| Scope still ambiguous             | Up to **2** clarifying questions (hard cap)           |
| End of report                     | Always asks one closing question about the gap to own |

Never an interview. The cap is intentional.

## Usage

In a Claude Code session, trigger the skill by either:

- Typing `/competitor-research`, or
- Asking naturally: *"Who are our competitors for X?"* / *"Run a competitive
  analysis on Y"* / *"What's the market landscape for Z?"*

The skill will scope, fan out subagents in parallel, and return:

1. One **6-bullet card** per competitor (positioning, target customer, pricing,
   differentiators, weaknesses, source date)
2. A **Gap Analysis** section with three falsifiable gaps
3. One sharp closing question

## File layout

```
.
├── CLAUDE.md                                  # project-level instructions for Claude
├── README.md                                  # this file
└── .claude/
    ├── skills/
    │   └── competitor-research/
    │       └── SKILL.md                       # user-invocable orchestrator
    └── agents/
        ├── research-agent.md                  # positioning, target customer, context
        ├── pricing-analyst.md                 # tiers, price points, billing model
        ├── review-miner.md                    # recurring strengths + weaknesses
        └── positioning-mapper.md              # hero copy, taglines, differentiators
```

All four subagents share the same wiring:

- **tools**: `WebSearch`, `WebFetch`, `Read` (read-only — no write/edit access)
- **model**: `claude-opus-4-6`

## Design conventions (enforced by the skill + subagents)

- **Recency bias** — prioritize sources from the last 12 months; flag older
- **Cross-referencing** — every non-trivial claim needs ≥2 independent sources
- **Hard 6-bullet cap** per competitor, with `—` placeholders rather than padding
- **Falsifiable gaps** — *"no per-seat pricing under $10/mo for teams <5"* beats
  *"better UX"*
- **Patterns, not anecdotes** — one G2 complaint is noise; 5+ across platforms is
  signal
- **Unknown is valid** — *"Pricing not public"* beats an invented number
