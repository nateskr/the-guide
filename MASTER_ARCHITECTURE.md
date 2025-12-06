# MASTER ARCHITECTURE STANDARD

## Section 1: Git Branching and History Management
A disciplined Git model protects the lineage of changes and keeps collaboration predictable. Practices here draw from the official Git documentation (`git-branch`, `git-switch`, `git-checkout`, `git-merge`, `git-rebase`, `git-cherry-pick`, `git-revert`, `git-reset`, `git-fetch`, `git-push`, `git-remote`, `git-worktree`, `git-stash`, `git-log`, `git-reflog`, `git-tag`, `git-config`, `git-workflows`, `gitrevisions`) and Pro Git guidance on branching, distributed workflows, and history rewriting.

### Branch lifecycle
- **Create and move:** `git switch -c feature/x` (or `git checkout -b feature/x`) starts an isolated line of work. Use `git switch main` (or the default branch name) to return. Example: create `feature/api-pagination` to add pagination without touching a stabilization branch.
- **Track upstream:** `git branch --set-upstream-to origin/main` binds a local branch to its remote peer so `git pull` and `git push` operate predictably.
- **Review readiness:** Keep feature branches short-lived; rebase or merge frequently from the default branch to reduce drift.

### Integration options
- **Fast-forward merge:** When the target branch has not diverged, `git merge --ff-only feature/x` keeps history linear. Example: merging a small documentation update created after the last release tag.
- **Three-way merge:** Default `git merge` records a merge commit when both branches advanced. Use for feature completion when intermediate commits provide review context.
- **Rebase for clean story:** `git rebase main` rewrites local commits on top of the latest base to produce a straight history. Avoid rebasing published branches to prevent rewriting shared history; prefer `git merge` or `git revert` instead.
- **Cherry-pick for selective uplift:** `git cherry-pick <sha>` lifts specific commits (e.g., hotfix from `main` into `release/1.2`) without merging unrelated work.
- **Revert for safe undo:** `git revert <sha>` appends a new commit that inverts a change, preserving history for audit.

### History safety and recovery
- **Inspect history:** `git log --graph --decorate --oneline` shows topology and annotations. `git show <rev>` inspects a commit; `git diff <rev1>..<rev2>` compares ranges (`gitrevisions` syntax applies).
- **Protect shared timelines:** Never force-push (`git push --force-with-lease` only when you own the branch and collaborators agree). For release or ledger-like branches, treat history as immutable; prefer `git revert` for corrections.
- **Reset with intent:**
  - `git reset --soft <rev>`: keep index staged (fix message or add a file).
  - `git reset --mixed <rev>` (default): unstage files to re-sculpt commits.
  - `git reset --hard <rev>`: discard working directory and index to match `<rev>`; use only when local changes are disposable.
- **Recover with reflog:** `git reflog` lists all HEAD movements. To restore a lost state: `git checkout -b recovery/<date> <sha>`.

### Distributed coordination
- **Fetch before merge:** `git fetch --all --prune` synchronizes refs before integrating. Use `git pull --rebase` on private branches to minimize merge noise.
- **Push hygiene:** `git push -u origin feature/x` sets upstream for new work; use `--force-with-lease` only on branches you control and never on protected or shared stability branches.
- **Remotes and worktrees:** `git remote set-url origin <url>` maintains correct endpoints. `git worktree add ../review-main main` creates a parallel checkout for code review or release validation without context switching.
- **Stash for context switches:** `git stash push -m "save:ticket-123"` preserves uncommitted work; `git stash apply` restores while keeping the stash entry for safety.

### Release and support examples
- **Release branches:** Cut `release/1.4` from `main` for stabilization; accept only fixes via `git cherry-pick` from feature branches. Merge back to `main` after tagging with `git tag -a v1.4.0` to mark the snapshot.
- **Hotfix cadence:** When production breaks, branch `hotfix/<issue>` from the default branch, apply the fix, `git cherry-pick` to active release branches, then merge through standard review.
- **Long-running initiatives:** For refactors, keep `topic/refactor-storage` rebased on `main` and integrate in slices to avoid large, risky merges.

## Section 2: The Seven Design Fundamentals
The repository defines seven architectural fundamentals. Their names and intents are preserved exactly.

1. **Sidecar Data Model Pattern**
   - **Intent:** Introduce new logic to a legacy monolith without destabilizing the core schema.
   - **Invariant:** Legacy tables are read-only (or strictly identity-only) regarding new features. Sidecar tables are write-authoritative.
   - **Real-World Use:** Adding a modern subscription entitlement system to a 10-year-old users table without touching the messy legacy code.
   - **Git Governance:** Create a dedicated branch for the sidecar slice. Do not mix legacy refactors with sidecar creation.

2. **Immutable Ledger Pattern**
   - **Intent:** Ensure financial or audit events are append-only.
   - **Invariant:** Entries are INSERT ONLY. No UPDATE, no DELETE. Derived balances are computed, not stored as mutable cells.
   - **Real-World Use:** Premium currency history, audit logs, chargeback dispute trails.
   - **Git Governance:** Strict prohibition on force-pushing or rebasing any branch that alters ledger semantics. History rewriting in Git mirrors history rewriting in the ledger—both are forbidden.

3. **Idempotent Boundary Pattern**
   - **Intent:** Make external endpoints safe to retry infinitely without duplicating side effects.
   - **Invariant:** f(x) = f(f(x)). Retrying a payment webhook with the same idempotency_key must result in the same outcome (Success) without charging the user twice.
   - **Real-World Use:** Payment webhooks (Stripe/PayPal), reward claims.
   - **Git Governance:** The idempotency key schema constraint and the logic test must be committed in the same atomic unit.

4. **Stateful Queue Pattern**
   - **Intent:** Decouple authorization (API) from execution (worker) using a database-backed state machine.
   - **Invariant:** Jobs must move through explicit states: PENDING -> PROCESSING -> COMPLETED | FAILED.
   - **Real-World Use:** Delivering items to a game server. The API records the intent; the worker retries the network call until success.
   - **Git Governance:** State machine definitions (enums) must be documented in the same commit as the database CHECK constraints.

5. **Strategy Pattern (Policy)**
   - **Intent:** Encapsulate volatile business rules behind a stable interface.
   - **Invariant:** The workflow engine knows the contract (e.g., calculateDiscount()), not the implementation (e.g., SummerSaleStrategy).
   - **Real-World Use:** Battle pass logic (Season 1 vs Season 2), regional pricing rules.
   - **Git Governance:** Isolate rule changes. If the Season 1 logic changes, the commit message must explain the policy delta.

6. **Observer Pattern (Consistency)**
   - **Intent:** React to domain events without tight coupling.
   - **Invariant:** Producers emit events; consumers subscribe. Events are never the sole source of truth for money (use a ledger for that).
   - **Real-World Use:** Analytics, dashboards, achievement notifications.
   - **Git Governance:** Event schema changes are systemic changes. Use semantic commit messages to warn of breaking schema changes for downstream consumers.

7. **Adapter Boundary Pattern**
   - **Intent:** Isolate external third-party constraints from the core system.
   - **Invariant:** Only the adapter knows the external API exists. The core system speaks only its own internal language.
   - **Real-World Use:** Normalizing Stripe and PayPal into a single PaymentProvider interface.
   - **Git Governance:** One adapter per branch. Do not mix "Stripe Integration" and "Game Server Integration" in one PR.

## Section 3: Cross-Mapping Git Practices to the Seven Fundamentals
- **Sidecar Data Model Pattern:** Create one branch per sidecar slice (e.g., `feature/sidecar-entitlements`). Keep sidecar schema and logic together; integrate through reviewed merges to avoid mixing with legacy refactors.
- **Immutable Ledger Pattern:** Treat ledger branches like the ledger itself—no history rewrites. Use merge or revert for corrections; forbid rebases or force-pushes on shared branches that modify ledger semantics.
- **Idempotent Boundary Pattern:** Commit schema constraints (idempotency keys) and handler logic atomically on the same branch. Use fast-forward merges or rebased feature branches to keep the boundary change cohesive.
- **Stateful Queue Pattern:** Version the state machine with its enforcement. Keep enum definitions, database checks, and worker transitions in one branch and one commit series; document transitions in the merge message.
- **Strategy Pattern (Policy):** Isolate policy deltas in focused branches (e.g., `policy/discount-q3`). Use merge commits to narrate rule changes while keeping the stable interface intact.
- **Observer Pattern (Consistency):** When event schemas evolve, coordinate with consumers via clearly titled branches and semantic commit messages. Avoid squashing away schema history needed by subscribers.
- **Adapter Boundary Pattern:** Ship one adapter per branch to keep integrations decoupled. Use `git cherry-pick` only when an adapter change must be uplifted to multiple release lines without pulling unrelated code.

## Glossary (Git essentials)
- **Branch:** A movable pointer to a commit; enables parallel lines of development.
- **Commit:** An immutable snapshot of the project state; identified by a SHA-1 hash.
- **HEAD:** The current checked-out commit or branch reference.
- **Upstream:** The remote branch a local branch tracks for pulls and pushes.
- **Fast-forward:** A merge where the target branch tip is advanced to the source without creating a merge commit.
- **Three-way merge:** A merge that creates a new commit combining divergent histories.
- **Rebase:** Replay of commits onto a new base, producing new commit IDs; avoid on shared history.
- **Reflog:** Local log of reference movements used for recovery.
- **Worktree:** An additional working directory linked to the same repository, allowing multiple checkouts.
- **Stash:** A saved work-in-progress stack entry that stores uncommitted changes temporarily.
