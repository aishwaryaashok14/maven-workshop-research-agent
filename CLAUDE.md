# Competitor Research Workshop

This project is a small Claude Code harness for **intensive competitor research**.
It exposes one user-invocable skill and a set of specialist subagents that the skill
orchestrates in parallel.

## How the pieces fit

```
User → /competitor-research
         │
         ▼
  competitor-research (skill)
   1. Asks scoping questions
   2. Identifies 3–5 direct competitors via web search
   3. For each competitor, fans out 4 subagents IN PARALLEL:
        ├── research-agent       → positioning, target customer, company context
        ├── pricing-analyst      → tiers, price points, billing model
        ├── review-miner         → strengths + weaknesses from real users
        └── positioning-mapper   → messaging, hero copy, differentiators
   4. Synthesizes one 6-bullet card per competitor
   5. Writes a Gap Analysis section
   6. Ends with one sharp question
```

The skill is the orchestrator. The subagents are workers — each has a narrow remit
and returns structured findings only.

## Conventions

- **Recency bias**: prioritize sources from the last 12 months. Flag anything older.
- **Cross-reference**: every non-trivial claim should have ≥2 independent sources.
- **No padding**: bullet limits are hard caps, not targets. Empty bullets > filler.
- **Cite, don't paraphrase pricing**: pricing changes; always include the source URL
  and the date observed.
- **Reviews are signal, not gospel**: a single G2 complaint is anecdote; a recurring
  theme across 5+ reviews is a pattern. Only patterns make it into the final card.
- **Unknown is a valid answer**: if a competitor doesn't disclose pricing publicly,
  write "Not public" — never invent numbers.

## File layout

```
.claude/
├── skills/
│   └── competitor-research/
│       └── SKILL.md           # user-invocable orchestrator
└── agents/
    ├── research-agent.md      # general competitor context
    ├── pricing-analyst.md     # pricing deep-dive
    ├── review-miner.md        # user-voice research
    └── positioning-mapper.md  # messaging & differentiators
```

## When to use this

Trigger the skill when the user asks any of:
- "Who are our competitors for X?"
- "Run a competitive analysis on Y"
- "What's the market landscape for Z?"
- "Where's the gap in [category]?"

Do **not** trigger for: customer research, user interviews, internal product audits.
Those are different jobs with different methods.
