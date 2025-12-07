# Master Architecture Standard

## Purpose and scope
This standard defines how this repository designs systems and governs Git history. It links a set of design fundamentals to the branching, review, and recovery practices that protect them. Every change in this repository should align with these expectations.

## Design fundamentals
The following fundamentals describe how we evaluate architecture decisions:

1. **Clarity and cohesion** – Components communicate intent through minimal, cohesive responsibilities. Avoid ambiguous ownership and implicit coupling.
2. **Separation of concerns** – Boundaries are explicit and enforce invariants. Shared utilities remain composable rather than becoming hidden dependencies.
3. **Contract-first interfaces** – Inputs, outputs, and error modes are documented and versioned. Backward compatibility is explicit when contracts evolve.
4. **State discipline** – State transitions are deliberate and observable. Persistence, caching, and in-memory coordination follow predictable rules.
5. **Resilience and rollback** – Designs prioritize safe failure, graceful degradation, and fast recovery paths.
6. **Observability and auditability** – Critical paths expose metrics, logs, and traces that map to user impact. Changes are traceable to intent.
7. **Incremental delivery** – Work lands in small, reviewable slices with clear rollback options. Each step should be independently releasable.

## Git operating model
All contributors follow a strict, review-first workflow:

- **Default branch**: `main` remains releasable at all times.
- **Branching**: Create topic branches from `main` using descriptive, human-readable names (e.g., `docs/master-architecture`, `feat/<scope>`). Avoid tool-prefixed names.
- **Commit hygiene**: Write concise, intent-revealing messages (e.g., `docs: refine design fundamentals`). Each commit should be logically complete and buildable.
- **History strategy**: Prefer `rebase` to keep topic branches current. Avoid force-pushing shared branches; use additional commits to correct work.
- **Merge policy**: Open a pull request for every change. Use rebase or squash merges to preserve linear history. Do not merge with failing checks.
- **Review expectations**: Reviews focus on alignment with the design fundamentals and on verifying rollback paths. Blocking feedback must cite the specific fundamental it protects.
- **Tagging and releases**: Tag meaningful milestones from `main` using semantic tags where appropriate (e.g., `v1.0.0`).

## Crosswalk: design fundamentals to Git controls
| Design fundamental | Git control | How it protects the fundamental |
| --- | --- | --- |
| Clarity and cohesion | Descriptive branch names and commit subjects | Makes intent evident and isolates concerns in history. |
| Separation of concerns | One-scope-per-branch rule | Prevents unrelated changes from coupling in the same review. |
| Contract-first interfaces | PR checklist item for API/contract notes | Ensures reviewers validate backward compatibility before merging. |
| State discipline | Required rationale for state migrations in PRs | Forces explicit plans for persistence and rollback. |
| Resilience and rollback | Mandatory revert plan in PR description | Guarantees every change can be backed out quickly. |
| Observability and auditability | Commit messages reference user impact where possible | Keeps history traceable to operational signals. |
| Incremental delivery | Small, reviewable commits; rebase onto fresh `main` | Reduces risk and keeps work independently releasable. |

## Pull request checklist
Before opening or merging a PR:
- [ ] Scope is limited to one cohesive objective.
- [ ] Branch is rebased on the latest `main`; no merge commits from `main` are introduced.
- [ ] Tests or linters relevant to the change are documented and passing.
- [ ] Impacted fundamentals are identified in the description.
- [ ] A rollback plan (revert, feature flag removal, or config toggle) is stated.
- [ ] No AI/tooling artifacts or prompts appear in the diff or commit messages.

## Recovery playbook
- **Simple mistakes on a branch**: Use `git revert` for published commits; avoid history rewrites on shared branches.
- **Local cleanups**: Use `git commit --amend` or `git rebase -i` before pushing to clean intent while keeping each commit buildable.
- **Emergency fixes on `main`**: Branch from the problematic commit, apply the fix, and open a PR. Prefer revert + forward-fix over risky cherry-picks.
- **Lost work**: Use `git reflog` to locate orphaned commits; create a recovery branch from the desired reference.

## Governance and stewardship
- **Branch protections**: Configure `main` to require PR reviews, passing checks, and to block force pushes and branch deletions.
- **Documentation authority**: This file is the canonical standard for architecture and Git governance in this repository. Update it via PRs that describe both the design and process impacts.
- **Scope discipline**: Avoid mixing conceptual expansions with mechanical edits. If both are required, split them into separate PRs.

## Adoption guidance
- Use this standard as the default checklist for design discussions and post-incident reviews.
- When adding new documents, mirror this tone: clear, direct, and free of tool references.
- Prefer concrete examples and rollback notes over abstract principles.
