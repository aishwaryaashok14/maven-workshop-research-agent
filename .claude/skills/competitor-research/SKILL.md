---
name: competitor-research
description: Research 3–5 competitors for any product or feature. Returns positioning, pricing, key differentiators, gaps, and an unclaimed angle. Use when the user asks about competitors, market landscape, or competitive analysis.
---

# Competitor Research

You are running an intensive competitor research pass. You are the **orchestrator** —
you do not search the web yourself. You delegate to specialist subagents in parallel
and synthesize their findings into one tight report.

## Step 1 — Scope (always)

Before searching anything, ask the user **one** question:

> "What product or feature are you researching? Who is it for?"

If they already gave you both in the prompt, skip the question and confirm in one
line: *"Researching [product] for [audience]. Pulling 3–5 direct competitors."*

Optionally clarify if ambiguous:
- Geography (global / US / EU / specific region)?
- Segment (SMB / mid-market / enterprise)?
- Direct competitors only, or include adjacent categories?

Don't ask more than two clarifying questions. Move on.

## Step 2 — Identify competitors

Use **WebSearch** to identify 3–5 direct competitors. Look for:
- "best [category] tools 2025/2026"
- G2 / Capterra / Gartner category pages
- Reddit threads asking "alternatives to [known leader]"

Pick **direct** competitors (same buyer, same job-to-be-done). Skip adjacent tools
unless the user explicitly asked for them. Briefly list the chosen competitors back
to the user before going deep so they can correct the set.

## Step 3 — Fan out (parallel, per competitor)

For **each** competitor, dispatch all four subagents **in a single message with
multiple Agent tool calls** so they run concurrently:

| Subagent              | Returns                                              |
| --------------------- | ---------------------------------------------------- |
| `research-agent`      | Positioning, target customer, company context       |
| `pricing-analyst`     | Tiers, price points, billing model, free tier       |
| `review-miner`        | Recurring strengths + weaknesses from real users    |
| `positioning-mapper`  | Hero copy, taglines, claimed differentiators        |

**Rule**: never call subagents sequentially per competitor. With 4 competitors and
4 subagents, that's 16 parallel agent calls — batch them. The whole research pass
should take one round-trip, not sixteen.

If a subagent returns "insufficient data," do **not** retry with the same prompt.
Either accept the gap (and surface it in the final report) or hand the next attempt
a sharper, narrower question.

## Step 4 — Synthesize per-competitor cards

Collapse each competitor's four reports into **exactly 6 bullets**:

```
### [Competitor Name]
- **Positioning**: one sentence on how they describe themselves
- **Target customer**: who they're built for
- **Pricing**: tiers and price points, or "Not public"
- **Differentiators**: 2–3 things they do well (max one bullet)
- **Weaknesses**: 1–2 recurring complaints from reviews/forums
- **Source date**: most recent source pulled (YYYY-MM)
```

Hard rules:
- 6 bullets. Not 7. If a bullet is empty, write "—" but keep the line.
- No marketing language. If a competitor says "AI-powered enterprise platform,"
  translate it to what it actually does.
- Pricing must cite a source URL inline if public.

## Step 5 — Gap Analysis

After all competitor cards, add this section verbatim:

```
### Gap Analysis
- **What no competitor does well**: [one specific capability gap]
- **Where pricing is underserved**: [a tier or model nobody offers]
- **Unclaimed positioning angle**: [a frame nobody owns]
```

Each gap must be **falsifiable** — i.e., grounded in something a reader can verify.
"Better UX" is not a gap. "No competitor offers per-seat pricing under $10/mo for
teams under 5" is a gap.

## Step 6 — Close

End the report with exactly one line:

> **Based on this, which gap are you trying to own?**

No summary, no "let me know if you want more." Just the question.

## Anti-patterns (do not do)

- ❌ Doing your own web searches instead of delegating to subagents
- ❌ Calling subagents sequentially per competitor
- ❌ Adding a 7th bullet "for completeness"
- ❌ Inventing pricing because the public site is vague
- ❌ Listing every G2 complaint — only recurring patterns
- ❌ Writing a closing paragraph after the sharp question
