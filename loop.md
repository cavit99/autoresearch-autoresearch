# Autoresearch Loop

## Purpose

This document abstracts the core architecture behind `karpathy/autoresearch` and the broader autoresearch-style pattern into a reusable loop for any domain where:

- an agent can run bounded experiments or trials
- outcomes can be checked by a verifier
- results can be recorded and compared
- the mutable search surface can be separated from the truth layer

The key idea is not "train LLMs autonomously." The key idea is:

- human defines policy
- agent explores a constrained search space
- verifier scores outcomes
- keep or discard based on evidence
- repeat

That pattern can be applied to many domains:

- market research over frozen captures
- ranking and retrieval experiments over fixed evaluation sets
- pricing or policy search over replayable simulations
- workflow optimization over reproducible task logs
- document extraction or labeling systems with stable test corpora
- ETL or rules-engine tuning against known-good fixtures

## Core Components

### 1. Human Control Surface

This is the small, explicit place where the human states goals, priorities, and guardrails.

Examples:

- a `program.md`
- a task policy file
- a shortlist of experiment families
- explicit "allowed" and "forbidden" mutations

Requirements:

- small enough to reread often
- explicit about success criteria
- explicit about what the agent may and may not change

### 2. Truth Layer

This is the part the agent should not casually rewrite during experimentation.

Examples:

- frozen datasets
- replayable event logs
- captured market artifacts
- golden test fixtures
- deterministic simulators
- fixed evaluation corpora

Requirements:

- stable during an experiment cycle
- auditable
- reproducible
- separable from the experiment surface

### 3. Mutable Experiment Surface

This is the smallest set of things the agent is allowed to change in order to search for improvement.

Examples:

- a candidate spec
- one training file
- a set of thresholds or filters
- a feature-selection recipe
- a rules configuration

Requirements:

- bounded
- reviewable
- cheap to mutate
- easy to revert

### 4. Verifier

This decides whether a candidate is good enough to keep.

Examples:

- holdout metrics
- replay scorecards
- regression fixtures
- simulator outcomes
- operational SLAs against a known dataset

Requirements:

- stable across experiments
- hard to game accidentally
- cheap enough to run repeatedly
- produces machine-readable output

### 5. Execution Harness

This runs the candidate against the truth layer under a fixed comparison budget.

Requirements:

- explicit time, step, or resource budgets that stay stable within a search loop
- deterministic enough for comparison
- clear timeout and crash handling
- no hidden human steps during unattended runs

### 6. Ledger And Artifacts

Every run should leave behind enough evidence to understand what happened.

Minimum outputs:

- candidate or diff
- named incumbent or baseline reference
- status: `keep`, `discard`, or `crash`
- key metrics
- artifact locations
- timestamp or run id

## The Portable Loop

1. Read the control surface and current state.
2. Identify the current incumbent or baseline state.
3. Pick one bounded mutation to the experiment surface from that incumbent.
4. Run the candidate under the same verifier and comparison budget.
5. Record metrics and artifacts.
6. Compare the candidate against the incumbent.
7. If it improved enough, keep it and make it the new incumbent.
8. If it failed or regressed, discard or rewind to the incumbent.
9. Repeat until interrupted or budget is exhausted.

## Non-Negotiable Design Rules

- Keep the mutable surface tiny.
- Keep the verifier fixed during a search loop.
- Keep the truth layer separate from the experiment surface.
- Compare candidates against a named incumbent, not against memory or vibes.
- Every kept change becomes the new incumbent for the next step.
- Compare candidates under the same budget envelope.
- If the budget, backend, dataset, or metric changes materially, treat it as a sibling loop or new baseline.
- Prefer cheap, comparable runs over heroic but noisy runs.
- Make crashes first-class outcomes, not invisible failures.
- Keep generated artifacts out of git unless they are canonical inputs.
- Do not let unattended runs depend on hidden local approval prompts when that can be avoided.

## Porting Template

Use this mapping when moving the pattern into another repo:

| Abstract role | Questions to answer | Example implementations |
|---|---|---|
| Human control surface | Where does policy live? | `program.md`, policy YAML, experiment brief |
| Truth layer | What must stay stable while searching? | captures, fixtures, datasets, simulators |
| Mutable surface | What may the agent change? | candidate spec, rules file, one code module |
| Incumbent state | What is the current candidate to beat? | kept commit, deployed config, active ruleset |
| Verifier | How is success measured? | holdout metric, scorecard, tests, replay outcome |
| Execution harness | How is a run launched and bounded? | CLI, task runner, scheduled job |
| Ledger | How are outcomes recorded? | TSV, JSON scorecards, database rows |
| Artifact store | Where do run outputs go? | local workdir, S3, blob store |

## Common Repo Shape

A small repo often operationalizes this pattern as:

- `program.md` or equivalent for policy
- one small mutable target
- a stable truth layer
- a local ledger such as `ledger.md`, `results.tsv`, or scorecards
- a runner or automation that executes one bounded cycle

Exact file names and formats do not matter; optimize them for the specific domain and use case. The important property is separation: small policy, narrow mutable surface, stable verifier and truth layer within a loop, and enough ledger state to compare future runs.

## Porting Checklist

- Define the human control surface.
- Define the immutable or stable truth layer.
- Define the smallest possible mutable experiment surface.
- Define the incumbent or baseline reference.
- Define the verifier.
- Define run budgets and crash handling.
- Define output artifacts and ledger shape.
- Define the keep/discard rule.
- Define what must remain untracked.
- Define what should be checked daily from upstream inspirations.

## Anti-Patterns

- letting the agent rewrite the verifier while searching
- letting the agent mutate raw truth data to make metrics look better
- mixing orchestration, evaluation, and experiment policy into one giant mutable file
- recording only wins and silently dropping crashes
- turning every run into a bespoke snowflake with no comparable budget
- allowing generated artifacts to pollute version control
- assuming the pattern only applies to ML training

## What Changes Across Domains

The domain changes:

- data type
- candidate representation
- runtime environment
- budget definition
- metric definition
- failure modes

The architecture does not change:

- human policy
- bounded mutation
- fixed comparison budget within a loop
- named incumbent plus keep/discard
- auditable artifacts

## Update Policy

When upstream projects such as `karpathy/autoresearch` change, ask:

1. Is this a domain-specific tweak, or a portable architectural lesson?
2. Does it change the control surface, verification boundary, artifact model, or unattended-run ergonomics?
3. Should this repo's abstraction change?
4. Should downstream repos adopting this pattern change too?

Only update this document when the lesson survives abstraction and generalizes to other systems.
