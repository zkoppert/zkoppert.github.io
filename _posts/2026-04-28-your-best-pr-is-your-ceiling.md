---
title: "Your Best PR Is Your Ceiling. Make It Your Floor."
date: 2026-04-28
categories: [Engineering Practices, Code Review]
tags: [pull-requests, code-review, developer-experience, github-copilot, engineering-management]
image:
  path: /assets/img/posts/pr-ceiling-floor.png
  alt: "A bar chart showing PR description quality dimensions with varying completion rates"
---


A teammate asked me to help them level up their PR descriptions. As a part of this exploration, I built a rubric with seven dimensions, four quality tiers, and real examples from open source and internal projects. It made sense in theory. Then I thought: how do we turn best practices into personalized recommendations?

So I pulled every PR I'd authored across the GitHub org since 2020 and graded myself using the same framework.

The results were humbling. And they gave me a concrete playbook for getting better.

## The Setup: How I Analyzed 196 PRs

I authored 463 PRs across the GitHub org from 2020 through April 2026. These span public open source work and internal projects - I'm sharing patterns and numbers here, not code or confidential details. 246 of those were automation (not useful for evaluating description quality) and 21 were access-control config changes. That left **196 substantive PRs** across 58 repositories.

I used GitHub's search API to build a list of every PR, then read each description individually:

```bash
# Build the list of PRs (search returns paginated results,
# so split by date range for complete coverage)
gh search prs --author=zkoppert --owner=github \
  --created="2020-01-01..2022-12-31" \
  --json repository,number,createdAt,title
gh search prs --author=zkoppert --owner=github \
  --created="2023-01-01..2024-06-30" \
  --json repository,number,createdAt,title
# ... and so on through 2026

# Then fetch each PR's description for scoring
gh pr view 12345 --repo github/my-repo --json body
```

For each substantive PR, I manually read the description and scored it on seven quality dimensions plus two process signals. The three questions (why, what, how) are the foundation - the seven quality dimensions are how I measured whether each question was answered well: problem context, approach explanation, testing details, quantified impact, visual aids, monitoring plan, and tradeoff documentation. I also tracked two process signals - linked issues and template usage - that indicate good habits even though they aren't about description quality per se. Percentages are rounded from manual counts - some PRs partially addressed a dimension, so I used my best judgment.

## The Framework: Three Questions

Before I share my results, here's the framework I used to evaluate quality. Every PR description - from a one-line config change to a 500-line refactor - needs to answer three questions:

1. **Why should I care?** The problem, the motivation, the context.
2. **What did you do?** The approach - not a file list, but a strategy description.
3. **How do I know it works?** Testing evidence, benchmarks, before/after proof.

That's it. A config PR might answer all three in two sentences. A complex refactor might need tables, diagrams, and Datadog links. The framework scales to the complexity of the change.

I scored PRs into four tiers:

| Tier | What it means |
|------|--------------|
| **Best** | Answers all three questions with quantified impact, visual aids, tradeoff discussion, and monitoring plan |
| **Better** | Answers all three questions with meaningful detail |
| **Good** | Answers at least two questions, has basic structure |
| **Poor** | Missing context, one-liner, or just restates the title |

## My Numbers

Here's the honest breakdown across all 196 substantive PRs:

| Metric | Rate |
|--------|------|
| Problem/motivation explained | ~60% |
| Approach documented | ~55% |
| Testing details included | ~35% |
| Linked issues or related PRs | ~55% |
| Template used | ~38% |
| Visual aids (screenshots, tables, diagrams) | ~14% |
| Monitoring plan | ~15% |
| Tradeoff/alternatives documented | ~4% |
| Quantified impact (specific numbers) | ~3% |

That last row hit hard. Three percent. Out of 196 PRs, roughly 6 included any kind of number about *why the change matters*. Not "reduces latency by 200ms" - any number at all. "Affects ~500 users." "Saves ~20 min/week." "Covers 7 of 7 SLOs." Almost never.

## What I Already Do Well

It wasn't all bad. The analysis surfaced some genuine strengths I want to keep building on:

**Incident-driven PRs are my best work.** When a production incident motivated the change, my descriptions were detailed, structured, and evidence-based. One PR after an availability incident included a latency percentile table (P50/P95/P99 steady vs. spikes during the incident), a full call chain trace, blast radius analysis covering 10+ call sites, and a rationale for why the fix was scoped the way it was. That's the quality bar I want to hit everywhere.

**UI fixes get proper visual evidence.** When I fix something visible, I consistently include before/after screenshots - sometimes three (the problem, the fix, and proof that unrelated paths aren't affected). Screenshots are the highest-leverage addition to any UI PR.

**Comprehensive documentation PRs.** My best docs PRs trace every change back to its source - org-wide announcements, review feedback, specific Slack threads. They itemize changes by origin so a reviewer knows *why* each line changed, not just *what* changed.

**Pre-mortem and operational thinking.** One of my strongest PRs was a gameday scenario that mapped architecture dependencies and feature flag blast radius into a pre-mortem analysis. That kind of systems thinking shows up in my operational work but rarely in my everyday code PRs.

Those strengths were real, but they weren't consistent over time.

## The Growth Trajectory

Splitting the data by time period told an interesting story:

| Period | PRs | Typical Quality |
|--------|-----|----------------|
| Early (2020 - 2023) | 21 | Mostly terse - short descriptions, no templates, minimal context |
| Mid (2024) | 54 | Mixed - batch operations alongside improving substantive PRs |
| Recent (2025 - 2026) | 121 | Improving - template adoption, problem context, testing details |

The trajectory is positive. My recent PRs are meaningfully better than my 2020 work. But the improvement is uneven - I bring strong descriptions to high-stakes work (incidents, operational dashboards, complex refactors) and phone it in on "routine" changes. I want to make sure that a routine change doesn't end up causing a catastrophic incident because I didn't give it enough attention to have a clear PR description and the larger plan that a good description requires.

## My Five Strongest PR Descriptions

These represent the quality I'm aiming for consistently:

**1. A timeout fix after an incident** - A service was timing out under load, affecting multiple downstream consumers. The description includes a latency percentile table, traces the full call chain through three service boundaries, lists every affected call site, and explains why the fix was scoped to one service rather than applied generically. A reviewer could understand, evaluate, and approve the change without reading a single line of code first.

**2. A feed rendering fix** - A UI component was failing to render for a subset of users on one deployment path. Three screenshots: the problem state, the fixed state, and proof that the primary deployment isn't affected. An explicit "Tradeoffs" section with two documented tradeoffs. An "Alternatives considered" section explaining rejected approaches. Risk assessment scoped to a single deployment path.

**3. An availability dashboard update** - The team's monitoring dashboard was missing several SLOs and had no decision-support framework for incident response. Structured "what changed" with five categories of updates. Specific SLO IDs and target percentages for each addition. Links to the runbook notebook. Each change explains the *why* ("the first responder can see the target without needing to read the SLO widget details").

**4. A gameday scenario design** - The team needed a realistic incident simulation to practice feature flag rollback under pressure. Scenario summary with type, coverage, learning goals, structure, and duration. Expandable pre-mortem analysis mapping architecture dependencies. Mock artifacts listed for facilitator use.

**5. A training alignment PR** - Two org-wide announcements changed how teams should track and report on operational readiness. Changes organized by source announcement (two separate org-wide threads). Every change itemized with specific file references. A "Review-driven fixes" section documenting corrections from code review rounds.

## The Five Growth Areas I'm Working On

Based on the data, I identified five specific habits to build. I've already added these as instructions to my Copilot configuration so I get reminded on every PR I write.

### 1. Quantify Impact (was ~3%)

My best PR has a full latency percentile table. Most of my PRs don't include any numbers at all. Even docs and workflow PRs can benefit: "saves ~20 min/week of manual report generation" or "covers 7 of 7 SLOs (was 4 of 7)."

**The prompt I now use:** Before writing the description, I ask myself: "Can I put a number on why this matters?"

### 2. Document Tradeoffs (was ~4%)

When I do it, I do it well - my best PRs have full tradeoffs sections and scoping rationale. The gap is making this habitual rather than reserved for high-stakes PRs.

Even small PRs benefit from one sentence: "Chose to guard at the view layer rather than the controller because the feed data is already computed by this point."

### 3. Use Templates Consistently (was ~38%)

I use the team's PR template when working in our largest codebase, but most of my other repos either don't have templates or I skip the structure. I've adopted a lightweight fallback for repos without templates:

```markdown
## Why
## What changed
## Testing
## Rollout
```

Four headers. Fills in two minutes. Forces the three questions.

### 4. Make Non-Visual Work Visible (was ~14%)

My UI screenshots are strong. The opportunity is extending visual evidence to backend work:

- **Tables** mapping old behavior to new behavior
- **Mermaid diagrams** for architecture or workflow changes
- **Terminal output** for before/after of CLI or job output
- **Latency tables** for performance-sensitive changes

### 5. Include Monitoring Context in Code PRs (was ~15%)

My dedicated monitoring PRs are strong - specific SLO IDs, decision frameworks, runbook links. Bringing that same operational thinking to code PRs would close the loop between "I shipped it" and "I can prove it works in production."

A single line like "Watch `your_feature.error_rate` in your monitoring dashboard after deploy - expect < 0.1%" transforms a code PR into an operational artifact.

## How I'm Encoding These Habits

Knowing what to improve is covered now. Actually changing behavior is critical to improving and maintaining high quality. Here's my approach:

**I added the five growth areas to my global Copilot instructions file.** Now every time Copilot helps me draft or review a PR description, it prompts me on impact quantification, tradeoffs, monitoring context, and visual aids. It's like having a reviewer nudge me *before* I hit "Create pull request." If you don't use Copilot, the same approach works with a saved PR template, a checklist pinned in your editor, or even a text expander snippet.

Here's what those instructions look like:

```markdown
## Pull Requests
- **Quantify impact in every PR** - include at least one concrete number:
  latency change, number of users affected, percentage of requests covered,
  time saved, error rate reduction. Even rough estimates are better than
  no numbers at all.
- **Document tradeoffs and alternatives** - every PR description should
  include at least one sentence on what was considered and why this
  approach was chosen.
- **Use templates in every repo** - if no template exists, use:
  ## Why / ## What changed / ## Testing / ## Rollout
- **Make non-visual work visible** - for backend PRs, include at least
  one visual aid: a table, a mermaid diagram, or terminal output.
- **Include monitoring context in code PRs** - link the relevant
  dashboard, add a "what to watch after merge" note.
```

This turns a self-improvement goal into a system that nudges me on every PR. I don't have to remember the five areas - the tooling does. If you want to learn more about how I keep these instructions consistent across CLI and Codespaces, I wrote about that in [One File, Every Session](https://zackkoppert.com/posts/copilot-instructions-everywhere/).

## Try It Yourself

If you want to run this same analysis on your own PRs, here's the approach:

1. **Pull your PRs** using `gh search prs --author=YOUR_HANDLE --owner=YOUR_ORG`, then fetch each description with `gh pr view`
2. **Filter out automation** - Dependabot, config toggles, and other programmatic PRs inflate your count without reflecting description quality
3. **Score on the seven dimensions:** problem context, approach, testing, quantified impact, visual aids, monitoring, tradeoffs. For each PR, mark whether it includes each element - don't overthink partial matches, just use your best judgment.
4. **Look at your best 5 PRs** - that's your ceiling. The goal is to make your ceiling your floor.
5. **Encode what you learn** into your Copilot instructions, a saved PR template, or a team checklist so the improvement sticks

The gap between your best PR and your average PR is the growth opportunity. For me, that gap was significant - my incident-driven PRs were genuinely strong while my routine PRs were often bare minimum. Closing that gap doesn't require much more time per PR once you have a system in place. It requires a system that reminds you to apply the same rigor every time.





