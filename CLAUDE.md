# Competitor Research Workshop

This project is a small Claude Code harness for **intensive competitor research**.
It exposes one user-invocable skill and a set of specialist subagents that the skill
orchestrates in parallel.

> **Workshop note:** this CLAUDE.md is intentionally read by both Claude (every
> session) and by learners reading the repo. The rules below explain *why* they
> exist, not just *what* they are — that's how you write a CLAUDE.md that ages well.

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

---

## ⚠️ CRITICAL — non-negotiables

These are the rules that, if violated, break the system's value entirely. They are
not preferences.

1. **Never invent pricing.** If a competitor's price isn't public, write
   *"Not public"*. A made-up number is worse than no number — it poisons the gap
   analysis and any downstream decision.
2. **Never claim a single review as a pattern.** A G2 1-star rant is anecdote.
   Patterns require ≥3 confirming voices across ≥2 platforms. The whole point of
   `review-miner` is to filter noise from signal.
3. **Subagents fan out in parallel, not sequentially.** With 4 competitors × 4
   subagents = 16 agent calls — they go in *one* message with 16 tool calls, not
   16 messages. Sequential = 16× the wall-clock time for zero quality gain.
4. **The orchestrator (skill) does not do its own deep web searches.** Quick
   searches to *identify* the competitor list are fine. Once the list is set, all
   research is delegated. Mixing roles pollutes the orchestrator's context window
   and breaks parallelism.
5. **6 bullets per competitor card. Hard cap.** Empty bullets get an `—`. Don't
   add a 7th "for completeness" — the constraint is what forces clarity.

---

## ✅ DO

| Do | Why |
| --- | --- |
| Prioritize sources from the **last 12 months** | Pricing, positioning, and review sentiment all decay fast. Old data looks confident but lies. |
| **Cross-reference** every non-trivial claim against ≥2 independent sources | The company's own site is one source. You need at least one more before stating it as fact. |
| **Cite source URLs inline** for any pricing or differentiator claim | Lets the reader verify and re-check when prices change. |
| **Translate marketing-speak into plain English** in differentiator bullets | "AI-powered next-gen platform" is noise. The reader needs to know what the product *does*. |
| **Quote hero copy verbatim** in `positioning-mapper` output | The exact words a company chooses are signal — paraphrasing destroys it. |
| **Surface contradictions** between sources rather than picking one | Two sources disagreeing is itself a finding. Hiding it is dishonest. |
| **Flag single-source claims** with `(single-source)` | Preserves the signal but warns the reader. |
| **Make Gap Analysis bullets falsifiable** | "No per-seat plan under $10/mo for teams <5" is testable. "Better UX" is hand-waving. |
| **End the report with exactly one sharp question** | Forces the user to decide. The whole research pass is in service of that one decision. |

---

## ❌ DON'T

| Don't | Why it hurts |
| --- | --- |
| Don't add a closing summary paragraph after the question | Padding. The 6-bullet cards + gap section already are the summary. |
| Don't treat the company's "We're the leader" claims as positioning | That's self-praise, not signal. Their actual *positioning* is the hero headline + ICP. |
| Don't fabricate an enterprise tier price because the public site is vague | "Contact sales" is the answer. Inventing a number is the failure mode. |
| Don't list every weakness ever mentioned in a review | Only recurring patterns. Listing 8 one-off complaints buries the real signal. |
| Don't have subagents write to disk or run shell commands | They're read-only researchers. Their `tools:` list is restricted on purpose — minimal blast radius. |
| Don't ask the user more than 2 clarifying questions up front | Research that begins with an interview kills momentum. Bound it. |
| Don't retry a subagent with the same prompt if it returned "insufficient data" | Either accept the gap and surface it, or hand the next attempt a *narrower* question. Same prompt = same answer. |

---

## 🎓 Learning notes (for workshop participants)

These are the design choices worth understanding *why*:

**Why a skill + subagents instead of one big skill?**
A single skill running everything in one context window would burn tokens fast and
serialize work that has no reason to be serial. Splitting the four research
dimensions (positioning, pricing, reviews, messaging) into specialist subagents
means: (a) each can read deeply without polluting the others' context, (b) they
run in true parallel, (c) the orchestrator only sees the synthesized findings.
*Each subagent is a context window you don't have to pay for in the main thread.*

**Why are all four subagents read-only (`WebSearch`, `WebFetch`, `Read`)?**
The `tools:` list is a security boundary, not a hint. A subagent literally cannot
call tools outside its list. For research workers, `Bash` and `Write` would be
attack surface for no benefit — they have nothing to write.

**Why hard caps (6 bullets, 2 clarifying questions, 1 closing question)?**
Constraints force clarity. A skill with "give a thorough report" produces sludge.
A skill with "exactly 6 bullets" produces a thing the user can actually read and
act on. Cap the surface area; quality follows.

**Why is "Unknown" required to be a valid answer?**
The default failure mode of LLMs is to fabricate confident-sounding numbers when
real data is missing. Explicitly making *"Not public"* a first-class output
removes the incentive to invent. The skill checks for this — vague pricing in a
competitor card is treated as a defect, not a feature.

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
