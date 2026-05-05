---
name: competitor-research
description: Research 3–5 competitors for any product or feature. Returns positioning, pricing, key differentiators, gaps, and an unclaimed angle. Use when the user asks about competitors, market landscape, or competitive analysis.
---

# Competitor Research

You research competitors **yourself** — search the web, fetch their pages, judge
what matters. The one task you delegate is **review mining**, because distilling
user sentiment from 10+ scattered pages is heavy read-work that benefits from its
own context window.

> **Workshop note:** the split between "skill does it" and "subagent does it" is
> deliberate. Light, fast tasks (a homepage fetch, a pricing-page scan) stay in
> the skill — that's the orchestrator's job. Heavy, parallelizable read-tasks
> (10+ review pages per competitor, distilled into patterns) get delegated. That
> contrast is the whole lesson.

---

## Step 1 — Scope

Ask the user **once**:

> "What product or feature are you researching? Who is it for?"

Skip if they already gave both. Cap at **2** clarifying questions max (geography,
segment, direct vs. adjacent). Don't run an interview.

## Step 2 — Identify competitors

Use **WebSearch** to find 3–5 direct competitors. Look for:

- "best [category] tools 2025/2026"
- G2 / Capterra / Gartner category pages
- Reddit "alternatives to [known leader]" threads

Pick **direct** competitors (same buyer, same job-to-be-done). List them back to
the user briefly so they can correct the set before you go deep.

## Step 3 — Research each competitor (do this yourself)

For each competitor, in **parallel WebFetch calls**, pull:

1. **Homepage** — extract:
   - Hero headline (verbatim)
   - Subhead (verbatim)
   - Top 3 features in homepage order
   - Stated differentiators
2. **Pricing page** — capture:
   - Tier names and price points (USD/mo)
   - Billing cadence (monthly / annual / one-time)
   - Free tier limits
   - Hidden costs / add-ons
   - If pricing is "Contact sales," write that exactly. **Never invent a number.**

Translate marketing buzzwords into plain English in your notes ("AI-powered
next-gen platform" → what does it actually do?). But **quote the hero headline
verbatim** — that's signal, not slop.

## Step 4 — Delegate review mining

For each competitor, spawn one `review-miner` subagent. Send them all in **a
single message with multiple Agent tool calls** so they run concurrently.

**Why this one task is delegated:**
- Reviews live across G2, Capterra, Reddit, HN, ProductHunt, Trustpilot — that's
  10+ pages per competitor.
- The subagent must distinguish *patterns* from *anecdotes* (≥3 confirming
  voices across ≥2 platforms = signal). That's heavy reading + judgment.
- Doing it inline pollutes the orchestrator's context with raw review text.
  Doing it in a subagent returns clean structured findings.

The subagent returns: recurring strengths, recurring weaknesses, sources scanned.

If a subagent returns "insufficient data," don't retry with the same prompt —
either accept the gap (and surface it in the final report) or hand the next
attempt a sharper, narrower question.

## Step 5 — Synthesize per-competitor cards

Combine your homepage + pricing research with the subagent's review findings into
**exactly 6 bullets** per competitor:

```
### [Competitor Name]
- **Positioning**: one sentence (their words, plain English)
- **Target customer**: who they're built for
- **Pricing**: tiers and price points, or "Not public"
- **Differentiators**: 2–3 things they do well
- **Weaknesses**: 1–2 recurring complaints from reviews/forums
- **Source date**: most recent source pulled (YYYY-MM)
```

Hard rules:
- 6 bullets exactly. If a bullet is empty, write "—" but keep the line.
- Pricing must cite a source URL inline if public.
- No marketing language in your translation.

## Step 6 — Gap Analysis

Add this section verbatim:

```
### Gap Analysis
- **What no competitor does well**: [specific capability gap]
- **Where pricing is underserved**: [a tier or model nobody offers]
- **Unclaimed positioning angle**: [a frame nobody owns]
```

Each gap must be **falsifiable** — grounded in something a reader can verify.
"Better UX" is not a gap. "No competitor offers per-seat pricing under $10/mo
for teams under 5" is.

## Step 7 — Close

End the report with exactly one line:

> **Based on this, which gap are you trying to own?**

No summary. No "let me know if you want more." Just the question.

---

## Anti-patterns (do not do)

- ❌ Delegating homepage / pricing research to subagents — it's fast and light;
  the orchestrator handles it
- ❌ Calling review-miners sequentially per competitor instead of in parallel
- ❌ Adding a 7th bullet "for completeness"
- ❌ Inventing pricing because the public site is vague
- ❌ Listing every G2 complaint — only recurring patterns
- ❌ Writing a closing paragraph after the sharp question
