# Program

This file is the operating program for a scheduled automation run.

It is intended to work as the control surface for Codex and Claude Code automations.

Each invocation should complete exactly one observe cycle and then stop. Repetition belongs to the scheduler.

It watches `karpathy/autoresearch`, its forks, directly inspired repos, and broader public discussion, extracts only the portable lessons, and decides whether this repo's docs should change.

`README.md` is repo context. `loop.md` is the abstraction under evaluation. This file is the single control surface for the run itself.

## What Counts As The Pattern

Portable pattern:

1. human sets policy
2. agent mutates a bounded experiment surface
3. a verifier checks outcomes
4. the system records artifacts and keeps or discards changes
5. the loop repeats over a stable truth layer

This repo applies a review loop derived from the autoresearch loop:

- Human policy:
  - this file
- Truth layer:
  - upstream repos, commits, discussions, prior memory sections, and dated web sources
- Mutable promotion surface:
  - `loop.md`, and occasional contract edits in `README.md` when the repo contract itself changes
- Local state and ledger:
  - `memory.md`, which carries forward run state and doc pressure but is not part of the promoted canonical surface
- Verifier:
  - the inclusion, KISS, and recommendation rules below
- Keep or discard:
  - update the canonical abstraction only when the evidence clears the threshold

## Run Program

1. Read the most recent dated section in `memory.md` if it exists.
2. Reuse its `Checked through` timestamp as the lower bound for the next run.
3. If no prior section exists, use the last 48 hours and say so explicitly.
4. Check upstream changes in `karpathy/autoresearch` since that lower bound.
5. Search GitHub for architecture-relevant repos directly inspired by autoresearch, not just forks.
6. Scan broader web signals for non-repo evidence such as articles and papers.
7. Pull strong web leads back to primary sources when possible, such as repos, commits, docs, or explicit author statements.
8. Keep the search broad enough to catch non-fork innovation, but narrow enough to stay focused on portable architecture rather than hype.
9. Distinguish:
   - genuinely new discoveries
   - materially updated earlier discoveries
   - already known items with no meaningful new information
10. Extract only the lessons that survive abstraction away from the original domain.
11. Decide whether this repo should:
   - `update now`
   - `watch only`
   - `no action`
12. If the decision is `watch only` or `no action`, append one dated section to `memory.md` and stop.
13. If the decision is `update now`, create a fresh dedicated branch `autoresearch/<YYYYMMDD-HHMMZ>` from the default branch, currently `master`, then run the bounded promotion loop below.
14. Append one dated section to `memory.md` including the promotion outcome.
15. Stop. Do not begin another cycle in the same invocation.

## Discovery Rules

Use stable ids in the prose when they help disambiguate repeated signals:

- GitHub repo or fork: `github:<owner>/<repo>`
- web page, article, paper etc: normalized URL

Signal status:

- `new`
- `updated`
- `unchanged`

Only elevate a signal if it is architecture-relevant.

For GitHub discovery:

- always check upstream `karpathy/autoresearch`, but do not treat forks as the only meaningful search surface
- actively search for non-fork repos that reference `autoresearch`, implement the same loop, or clearly cite the upstream idea
- use metadata first
- use commit inspection before cloning if necessary
- inspect files only for the strongest candidates
- be selective about the candidate set
- prefer repos with concrete implementation evidence over idea-only repos

Good GitHub candidates include:

- repos named after `autoresearch` or obvious domain ports of it
- repos whose README or docs explicitly mention `karpathy/autoresearch`
- repos that recreate an autoresearch-style pattern under a different name
- notable non-forks surfaced by discussions, articles, or social posts

For web discovery:

- search beyond repo hosting; some important signals first appear as X posts or blog posts before code is easy to find
- prefer sources that point to concrete repos, commits, docs, or implementation notes
- treat Reddit, HN, X, daily.dev, personal blogs, and similar sites as discovery channels, then trace strong claims back to primary evidence when possible
- keep first-party statements from authors or implementers even when they are not yet backed by a polished repo, as long as the architectural claim is concrete
- do not elevate generic commentary

Stop expanding the search once you have enough evidence to support the recommendation.

## KISS Recommendation Gate

Default recommendation: `watch only`

Use `update now` when the overall case is strong enough to keep a change.

The non-negotiables are:

- the current canonical abstraction has a real gap or would mislead a careful reader
- the lesson is portable beyond the immediate source domain
- the clarity gained is greater than the conceptual surface area added

Do not require every supporting signal below but treat them as confidence factors that strengthen the case:

- concrete evidence from the current run
- pressure accumulated across prior runs
- repeated evidence across independent implementations
- strong first-principles reasoning, even when the public evidence is still thin
- the change fits into an existing section; if it needs new structure, the burden is higher

Use `watch only` when:

- the lesson is plausible and important but not ripe for canon yet
- the current abstraction is still directionally correct
- the signal increases pressure but is better recorded than adopted
- the evidence is thin and the first-principles case is not yet strong enough

Use `no action` when:

- the signal is still too domain-specific
- the signal is mostly packaging, setup, mirror churn, or hype
- the current docs already cover the lesson well enough
- the candidate is only stylistic and does not change likely reader interpretation or decisions

## Candidate Pressure To Watch

These themes are especially worth tracking, but should not be adopted casually:

- coordination substrates above many local loops
- worker isolation plus shared evidence plus explicit adoption
- sibling loops when backend, dataset, or metric changes
- budget, concurrency, crash, and promotion gates for unattended runs

## Promotion Loop

Only run this phase when the decision is `update now`.

- create a fresh dedicated branch `autoresearch/<YYYYMMDD-HHMMZ>` for this run; use a UTC timestamp so scheduled runs do not collide
- promotion targets `loop.md`
- do not begin promotion if the tracked worktree is not clean; record `deferred` in the memory section and stop after appending the section
- use the current `HEAD` on the run branch as the baseline
- each candidate should be one conceptual change, not a bundle of unrelated edits
- rejected candidates should stay in scratch or ephemeral working state and must not touch tracked files
- when a candidate wins, write it to `loop.md` and commit it immediately on the run branch using `docs: promote loop <short-theme>`
- never promote directly on the default branch
- keep at most one winning promotion commit per scheduled run
- after a win, leave the promotion loop and proceed to the output contract; do not start another candidate

## Output Contract

`memory.md` is the only local run artifact and the only persistent run state for this loop.

Each run should append one dated section that stays concise and answers:

- what mattered
- what is worth remembering
- why the recommendation was conservative or decisive
- what exact doc pressure is building, if any

Each section should include, in this exact order:

- `Decision`
- `Promotion`
- `Checked through`
- `What mattered upstream`
- `Signals worth remembering`
- `Doc pressure building`
- `Why this recommendation`
- `Checked and discarded`
- `Likely next change`

Use `- none` when a section has nothing worth recording. 

In `Promotion`:

- use `- none` for `watch only` or `no action`
- use `- deferred: tracked worktree not clean` when promotion is blocked
- use one line per kept commit in the form `- kept <short-hash>: <theme>`
- summarize losses as `- discarded <n> candidate(s)` instead of logging every micro-attempt

In `Checked through`:

- record one UTC timestamp in ISO 8601 form

Include source links inline instead of maintaining a second machine-readable ledger.

Do not duplicate schema, policy, or templates in `memory.md`. `memory.md` is state only.

Do not use platform-specific automation sidecar memory files as loop state. The repo-local `memory.md` is the source of truth for both Codex and Claude Code automations.

## File Boundaries

Touch by default:
- append new dated section to `memory.md`
- use `.scratch/github-inspection/` for temporary inspection work and clean it afterward

Touch only when the evidence clearly justifies a repo change:

- `loop.md` only through the bounded `update now` promotion loop
- `program.md` only when the run procedure, discovery rules, or output schema should change
- `README.md` only when the repo contract or file roles change
- `.gitignore` only when artifact tracking policy changes

Strongly prefer not to touch unless the evidence unequivocally justifies it:

- `README.md` or `program.md`
- `AGENTS.md` unless the human explicitly requests it
- tracked files unrelated to the current abstraction or repo contract

Default rule: write the digest, use scratch space, and leave the control docs alone unless the evidence crosses the threshold for changing them.

Git behavior:

- do not commit directly on the default branch
- if the decision is `watch only` or `no action`, append the memory section, do not commit
- do not create a run branch unless the decision is `update now`
- if the decision is `update now` and the tracked worktree is not clean, do not create a branch or promote; record `deferred` in the memory section
- if the decision is `update now` and the tracked worktree is clean, create `autoresearch/<YYYYMMDD-HHMMZ>` from the default branch and run the bounded promotion loop over `loop.md` only on that run branch
- commit at most one winning promotion on the run branch; that commit should touch only `loop.md`
- if no promotion wins, leave the run branch disposable
- leave integration of kept promotion commits back to the default branch to the human after review
- don't commit `memory.md` or `.scratch/`
- include kept short hashes and discarded candidate count in that day's `Promotion` section

## Working Constraints

- Prefer primary sources where available.
- Use concrete commit links for claims.
- Reuse scratch clone location under `.scratch/github-inspection/`.
- Clean the scratch clone contents when done.
- Do not propose doc changes just to stay busy.
- If there are no meaningful portable lessons, say that clearly.
