# RoleRadar — AI Job Search Advisor

> **Product case study** · Live at [roleradar.xyz](https://roleradar.xyz)

---

## The Problem

Job searching is broken in a specific, overlooked way.

The tools are fine at volume — job alerts, keyword searches, LinkedIn recommendations. The problem is signal quality. Most alerts are noise. You end up spending hours reading job descriptions that fail on basic criteria you could have stated upfront: wrong compensation range, requires relocation, full-stack when you want pure PM.

The deeper problem is that the matching is stateless. Every search session starts from scratch. The system doesn't know what you've already seen, what you passed on and why, or what your evolving sense of "right fit" looks like. You're doing all the cognitive work the tool should be doing for you.

**RoleRadar is built around a different premise:** the job search advisor should do the reading, filtering, and evaluation — and get smarter about your preferences over time.

---

## How It Works (User Perspective)

1. **Set your guardrails once** — compensation floor, remote preference, deal-breakers (e.g., no defense, no crypto). These are hard constraints, not preferences.

2. **The agent runs autonomously** — it searches a live job database aggregated from major hiring platforms, reads full job descriptions, and evaluates each role against your profile and stated criteria.

3. **You review a curated shortlist** — not 200 listings. Typically 5–15 roles the agent actually recommends, with a short reasoning note for each.

4. **Thumbs up / thumbs down** — your feedback is analyzed to extract preference patterns. The next run uses those patterns to calibrate evaluation.

5. **The advisor remembers** — after each session, the agent writes persistent notes about what it learned. These notes are read at the start of the next session. The advisor compounds context across sessions, like a recruiter who's been working with you for months.

6. **Your pipeline lives here** — jobs move from shortlist → applied → interviewing → offer/rejected. The Today Dashboard surfaces what needs attention: unreviewed picks, pending follow-ups, upcoming interview prep.

---

## Key Product Decisions

### 1. Agentic search instead of keyword matching

Early versions used standard keyword search. The problem: keywords are a proxy for fit, not fit itself. "Senior PM" at a 10-person startup and a FAANG mean completely different things. "Fintech" can mean payments infrastructure or crypto speculation.

The agent reads job descriptions — the actual text — and evaluates them against the user's full context. It also autonomously decides which search strategies to run, because the best searches are often non-obvious (searching by technology stack, by team structure mentioned in JD, by specific product area).

**Why it matters for the user:** fewer things to read, better quality of what surfaces.

### 2. Hard guardrails layer before the LLM

Every job passes through a rule-based hard filter before the LLM sees it. If you set a $150K compensation floor and the job posts $110K–$130K, the LLM never reads it.

This was a deliberate design choice for three reasons:

- **Cost**: LLM evaluation has a per-call cost. Passing clearly disqualified jobs wastes it.
- **Trust**: Users need to trust that their hard requirements are treated as hard. A "smart" system that occasionally surfaces roles below your stated floor — even with good reasoning — erodes confidence.
- **Accuracy**: LLMs are good at nuanced judgment. They are not better than a simple rule check at "does this role meet a stated numeric threshold."

Separation of concerns: rules do what rules are better at, LLMs do what LLMs are better at.

### 3. Feedback learning loop

The system analyzes thumbs-up / thumbs-down signals to extract implicit preference patterns. Not just "you liked 3 companies" — but what did those companies have in common? Company stage, product type, team size signals in the JD, role scope.

The alternative — asking users to fill out detailed preference forms — doesn't work in practice. People don't know what they want until they see it. Revealed preference from actual evaluations is more reliable than stated preference in a setup wizard.

**The design tension here:** you need enough signal before the learning is meaningful. The system is honest about this — early sessions have lighter personalization, later sessions reflect richer patterns.

### 4. Narrative memory across sessions

After each run, the agent writes a short advisory note: what it observed, what the user responded to, any patterns worth carrying forward. These notes are read at the start of the next session.

The alternative — purely vector-based similarity search over past interactions — loses the reasoning. It can surface "user liked X" but not "user liked X because of Y, which suggests they care about Z." Natural language notes preserve that context compactly.

This is the difference between a job board and an advisor. An advisor builds a model of you over time. The note-writing step is what makes that possible without requiring the user to re-explain themselves each session.

### 5. Today Dashboard as a habit surface

Job searching suffers from boom-bust cycles: intense focus for a week, then nothing for two weeks. Applications go stale. Follow-ups get forgotten. Momentum dies.

Today Dashboard is designed to create a daily touch point — light, low-friction, action-oriented. It surfaces: unreviewed picks (the agent found something, take a look), pending follow-ups (you said you'd reach out to this contact), upcoming interview prep needed.

The goal is 5–10 minutes per day, not 3-hour sessions twice a month.

---

## Product Architecture (Concept)

```
USER INTENT + GUARDRAILS
        │
        ▼
┌───────────────────┐
│  Hard Filter      │  ← Rule-based, fast, deterministic
│  (Comp, Remote,   │    Disqualifies obvious mismatches
│   Deal-breakers)  │    before LLM evaluation
└────────┬──────────┘
         │ Passing jobs only
         ▼
┌───────────────────┐
│  Agent Evaluation │  ← LLM reads full JDs
│  (Agentic search, │    Reasons against user profile
│   JD reading,     │    Writes recommendation notes
│   fit scoring)    │
└────────┬──────────┘
         │ Ranked shortlist
         ▼
┌───────────────────┐
│  User Review      │  ← Thumbs up/down
│  (Shortlist UI)   │    Triggers feedback analysis
└────────┬──────────┘
         │ Signals
         ▼
┌───────────────────┐
│  Memory Layer     │  ← Preference patterns extracted
│  (Feedback loop + │    Advisory notes written
│   Narrative notes)│    Both read on next run
└───────────────────┘

PIPELINE: Interested → Applied → Interviewing → Offer/Rejected
TODAY DASHBOARD: Surfaces what needs action today
PREP TOOLS: Interview prep, resume tailoring, networking drafts — per job
```

---

## Honest Limitations

**Coverage is bounded.** RoleRadar searches within a specific aggregated database of jobs from major ATS platforms. It doesn't crawl every company career page. If a company doesn't use one of the supported hiring platforms, it won't appear.

**LLM evaluation is probabilistic.** The agent can be wrong about fit. It can miss context that a human would catch, or over-weight signals that don't matter for a specific person. The shortlist is a starting point, not a final verdict.

**Learning requires volume.** The feedback loop needs interaction data to become meaningful. In the first few sessions, personalization is light. This is intentional — the system shouldn't pretend to know you before it does.

**Interview prep and resume tailoring are drafts.** Generated content needs human review and editing. The agent produces starting points, not finished artifacts.

---

## What I'd Do Differently

**1. Earlier explicit preference capture — but differently.** I avoided setup wizards because they don't work. What I'd add: a short conversational onboarding where the agent asks 3–4 targeted questions based on what it already knows from your LinkedIn/resume. Not a form — a conversation.

**2. Confidence scoring with explanation.** The agent evaluates fit and surfaces recommendations, but the confidence level isn't always visible to the user. I'd surface a simple signal: "Strong match," "Worth considering," "Borderline" — with a one-line reason.

**3. Company context layer.** The agent evaluates JD fit but doesn't deeply contextualize the company — funding stage, recent news, Glassdoor signals, leadership background. Adding that layer would improve evaluation quality significantly.

**4. Richer pipeline analytics.** Right now the pipeline tracks status. A more useful version would surface: average time-to-response by company type, which role types are getting callbacks, where you're dropping off in the funnel. Data the user can act on.

**5. Collaborative mode.** Job searching often involves a partner, mentor, or coach. The current system is single-user. A "share my shortlist" or "get a second opinion" workflow would have real value.

---

## What's Next

- Broadening job database coverage
- Resume-to-JD gap analysis (what skills/framing to emphasize per role)
- Referral network surfacing (who in your network works at a company on your shortlist)
- Mobile-optimized Today Dashboard for truly daily use
- Calendar integration for interview tracking

---

*RoleRadar is a working product, not a demo. Built to solve a problem I had, with product decisions I'd defend in a PM interview.*


