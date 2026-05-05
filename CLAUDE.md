# Competitor Research Workshop

This project is a small Claude Code harness for **competitor research**.
It exposes one user-invocable skill that does the bulk of the work itself,
plus one specialist subagent for the one task that genuinely benefits from its
own context window.

> **Workshop note:** this CLAUDE.md is intentionally read by both Claude (every
> session) and by learners reading the repo. The rules below explain *why* they
> exist, not just *what* they are — that's how you write a CLAUDE.md that ages well.

## How the pieces fit

```
User → /competitor-research
         │
         ▼
  competitor-research (skill)
   1. Asks scoping question
   2. WebSearch → identifies 3–5 direct competitors
   3. For each competitor, in parallel WebFetch:
        ├── homepage  → hero copy, differentiators, top features
        └── pricing   → tiers, price points, free tier, hidden costs
   4. For each competitor, spawns ONE review-miner subagent in parallel:
        └── review-miner → recurring strengths + weaknesses from real users
   5. Synthesizes one 6-bullet card per competitor
   6. Writes a Gap Analysis section
   7. Ends with one sharp question
```

## The skill / subagent split (and why)

The skill **does its own research** for homepages and pricing — those are fast,
light fetches. The skill **delegates** review mining because:

- Reviews live across G2, Capterra, Reddit, HN, ProductHunt, Trustpilot — that's
  10+ pages per competitor of dense user-voice text
- The subagent must distill *patterns* (≥3 confirming voices across ≥2 platforms)
  from *anecdotes* — a real reasoning task
- Doing it inline pollutes the orchestrator's context with raw review text;
  delegating returns a clean, structured summary

This split is the workshop's central lesson: **delegate the heavy, parallelizable,
read-intensive tasks. Keep the lightweight orchestration logic in the skill.**

---

## ⚠️ CRITICAL — non-negotiables

These are the rules that, if violated, break the system's value entirely.

1. **Never invent pricing.** If a competitor's price isn't public, write
   *"Not public"*. A made-up number poisons the gap analysis and any downstream
   decision.
2. **Never claim a single review as a pattern.** A G2 1-star rant is anecdote.
   Patterns require ≥3 confirming voices across ≥2 platforms.
3. **Review-miner subagents fan out in parallel, not sequentially.** All of them
   go in *one* message with multiple Agent tool calls — sequential = N× the
   wall-clock time for zero quality gain.
4. **Don't delegate the easy stuff.** Homepage and pricing fetches are light;
   the skill does them. Delegating fast tasks adds latency and obscures what the
   skill is actually doing.
5. **6 bullets per competitor card. Hard cap.** Empty bullets get an `—`. Don't
   add a 7th "for completeness" — the constraint is what forces clarity.

---

## ✅ DO

| Do | Why |
| --- | --- |
| Prioritize sources from the **last 12 months** | Pricing, positioning, and review sentiment all decay fast. Old data looks confident but lies. |
| **Cross-reference** every non-trivial claim against ≥2 independent sources | The company's own site is one source. You need at least one more. |
| **Cite source URLs inline** for any pricing or differentiator claim | Lets the reader verify and re-check when prices change. |
| **Quote hero copy verbatim** when you fetch a homepage | The exact words a company chooses are signal — paraphrasing destroys it. |
| **Translate marketing-speak into plain English** in differentiator bullets | "AI-powered next-gen platform" is noise. Reader needs to know what the product *does*. |
| **Surface contradictions** between sources rather than picking one | Two sources disagreeing is itself a finding. Hiding it is dishonest. |
| **Make Gap Analysis bullets falsifiable** | "No per-seat plan under $10/mo for teams <5" is testable. "Better UX" is hand-waving. |
| **End the report with exactly one sharp question** | Forces the user to decide. The whole research pass is in service of that decision. |

---

## ❌ DON'T

| Don't | Why it hurts |
| --- | --- |
| Don't add a closing summary paragraph after the question | Padding. The 6-bullet cards + gap section already are the summary. |
| Don't treat the company's "We're the leader" claims as positioning | That's self-praise, not signal. Their actual *positioning* is the hero headline + ICP. |
| Don't fabricate an enterprise tier price because the public site is vague | "Contact sales" is the answer. Inventing a number is the failure mode. |
| Don't list every weakness ever mentioned in a review | Only recurring patterns. Listing 8 one-off complaints buries the real signal. |
| Don't have the subagent write to disk or run shell commands | It's a read-only researcher. Its `tools:` list is restricted on purpose — minimal blast radius. |
| Don't ask the user more than 2 clarifying questions up front | Research that begins with an interview kills momentum. Bound it. |
| Don't retry the subagent with the same prompt if it returned "insufficient data" | Either accept the gap or hand it a *narrower* question. Same prompt = same answer. |

---

## 🎓 Learning notes (for workshop participants)

These are the design choices worth understanding *why*:

**Why one subagent and not four?**
Earlier drafts of this project had four specialists (one each for positioning,
pricing, reviews, messaging). It was an impressive fan-out — and pedagogically
confusing. Three of those tasks (homepage, pricing, messaging) were light
fetches; delegating them added latency and obscured what the skill itself
*does*. Reviews are the only task that genuinely earns its own context. Picking
the **right thing to delegate** is more interesting than delegating everything.

**Why is the subagent read-only (`WebSearch`, `WebFetch`, `Read`)?**
The `tools:` list is a security boundary, not a hint. A subagent literally
cannot call tools outside its list. For a research worker, `Bash` and `Write`
would be attack surface for no benefit — it has nothing to write.

**Why hard caps (6 bullets, 2 clarifying questions, 1 closing question)?**
Constraints force clarity. A skill with "give a thorough report" produces sludge.
A skill with "exactly 6 bullets" produces something the user can read and act on.

**Why is "Unknown" a required valid answer?**
The default failure mode of LLMs is to fabricate confident-sounding numbers when
real data is missing. Making *"Not public"* a first-class output removes the
incentive to invent. Vague pricing in a competitor card is treated as a defect,
not a feature.

**Why does the skill ask only 1–2 questions up front?**
Research with too many gates feels like a customer service script. The skill is
designed to make a reasonable choice and proceed; the user can correct course
mid-flight. The closing question (*"Which gap are you trying to own?"*) is where
the real decision happens — that's why it's the only one that's mandatory.

---

## File layout

```
.claude/
├── skills/
│   └── competitor-research/
│       └── SKILL.md           # user-invocable skill — does the work itself,
│                              # delegates only review mining
└── agents/
    └── review-miner.md        # the one specialist subagent
```

## When to use this

Trigger the skill when the user asks any of:
- "Who are our competitors for X?"
- "Run a competitive analysis on Y"
- "What's the market landscape for Z?"
- "Where's the gap in [category]?"

Do **not** trigger for: customer research, user interviews, internal product audits.
Those are different jobs with different methods.
