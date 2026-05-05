# Competitor Research Agent

A Claude Code harness for **competitor research**. One user-invocable skill does
the bulk of the work itself, plus one specialist subagent for the one task that
genuinely benefits from its own context window.

## How it works

```
User: "Run competitor research on Notion"
        │
        ▼
Claude reads available skills → matches "competitor-research"
        │
        ▼
┌────────────────────────────────────────────────┐
│  SKILL: competitor-research                    │
│  (instructions Claude follows directly)        │
│                                                │
│  Step 1: Ask scoping question                  │
│  Step 2: WebSearch → pick 3–5 competitors      │
│  Step 3: For each competitor (in parallel)     │
│          ├── WebFetch homepage  ← skill does   │
│          └── WebFetch pricing   ← skill does   │
│  Step 4: For each competitor, spawn ONE        │
│          review-miner subagent in parallel ─┐  │
│  Step 5: Synthesize 6-bullet cards          │  │
│  Step 6: Write Gap Analysis                 │  │
│  Step 7: Ask the closing question           │  │
└─────────────────────────────────────────────┼──┘
                                              │
            ┌─────────────────────────────────┘
            ▼ (one per competitor, all in parallel)
   ┌────────────────────────────────────┐
   │ review-miner (subagent)            │
   │ Reads 10+ pages across G2, Reddit, │
   │ HN, ProductHunt, Trustpilot.       │
   │ Returns recurring strengths +      │
   │ weaknesses (≥3 voices, ≥2 plats).  │
   └────────────────────────────────────┘
```

## The skill / subagent split — and why it matters

This is the central design choice, and it's the workshop's main lesson.

| Task | Where it runs | Why |
| --- | --- | --- |
| Scoping & competitor selection | **Skill** | Light reasoning + one search. Fast. |
| Homepage research | **Skill** (parallel WebFetch) | One page per competitor. Fast and light. |
| Pricing research | **Skill** (parallel WebFetch) | One page per competitor. Fast and light. |
| Review mining | **Subagent** (one per competitor) | 10+ pages across G2/Reddit/HN/PH/Trustpilot. Heavy reading + judgment. Pollutes context if done inline. |
| Synthesis & Gap Analysis | **Skill** | Needs the full picture in one place. |

**Rule of thumb for delegating:** push down work that is (a) heavy enough to
warrant its own context window, (b) parallelizable across N items, and (c) returns
clean structured output. Anything else stays in the skill.

## Skill vs. subagent — the mental model

- **Skill** = a *playbook* the main Claude reads and follows. It's not a separate
  process; it's instructions loaded into the current conversation. It can call
  tools (WebSearch, WebFetch, etc.) directly.
- **Subagent** = an *actual separate Claude instance* with its own context window,
  tool access, and remit. The skill spawns it via the `Agent` tool when it has a
  heavy, parallelizable task to offload.

The user only ever talks to the **skill**. The subagent is invisible — it just
returns a structured report that the skill folds into the final answer.

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

The skill returns:

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
    │       └── SKILL.md                       # user-invocable skill (does the work)
    └── agents/
        └── review-miner.md                    # the one specialist subagent
```

The subagent's wiring:

- **tools**: `WebSearch`, `WebFetch`, `Read` (read-only — no write/edit access)
- **model**: `claude-opus-4-6`

## Design conventions (enforced by the skill + subagent)

- **Recency bias** — prioritize sources from the last 12 months; flag older
- **Cross-referencing** — every non-trivial claim needs ≥2 independent sources
- **Hard 6-bullet cap** per competitor, with `—` placeholders rather than padding
- **Falsifiable gaps** — *"no per-seat pricing under $10/mo for teams <5"* beats
  *"better UX"*
- **Patterns, not anecdotes** — one G2 complaint is noise; ≥3 across ≥2 platforms
  is signal
- **Unknown is valid** — *"Pricing not public"* beats an invented number
