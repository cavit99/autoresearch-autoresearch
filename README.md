# Autoresearch Autoresearch

This repo maintains a canonical portable autoresearch loop in `loop.md`: a reference abstraction that others can adapt to specific domains and implementations.

Its main reference case is `karpathy/autoresearch`, but the goal here is not to mirror that repo. The goal is to extract the underlying loop architecture, keep it portable, and refine it when new evidence justifies a change.

## Files That Matter

This repo is deliberately small and only really has three tracked files that matter:

- `program.md` - the agent control surface and daily run instructions
- `loop.md` - the canonical abstraction and porting guide
- `README.md` - the repo contract

Local untracked artifacts:

- `ledger.md` - append-only local run log
- `.scratch/github-inspection/` - temporary inspection area

`program.md` is the automation entry point. Each scheduled run completes one observe cycle and stops. Only `update now` runs may enter a bounded promotion loop over `loop.md`; recurrence comes from the scheduler.

`ledger.md` is local run state and an append-only ledger for future runs, not part of the promoted canonical doc surface. `ledger.md` and `.scratch/` stay out of git.

## Scope

This repo tracks portable architecture changes. The examples below are illustrative, not a binding checklist:

- control surfaces
- mutable versus fixed boundaries
- verification and scorekeeping
- crash handling
- unattended-run ergonomics
- artifact logging and git hygiene
- coordination layers above the core loop
- platform, backend, or metric shifts that create a genuinely different loop

If a signal is interesting only because of the original training domain, it is out of scope here.

See `program.md` for the run procedure and `loop.md` for the abstraction being maintained.


## License

MIT

Cavit Erginsoy, 2026
