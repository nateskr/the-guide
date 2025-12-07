# the-guide

Long-form documentation that standardizes Git architecture discipline and core system design fundamentals.

## Table of contents
- [What this repository contains](#what-this-repository-contains)
- [How to use this repo](#how-to-use-this-repo)
- [Contribution and branch discipline](#contribution-and-branch-discipline)
- [Quick navigation](#quick-navigation)
- [Status](#status)

## What this repository contains

### 📌 Master Standard
- **MASTER_ARCHITECTURE.md**
  - A consolidated, strict architecture standard that unifies:
    1. Git branching and history management concepts grounded in official Git documentation and the Pro Git book.
    2. The repository’s canonical **seven design fundamentals**.
    3. A cross-mapping that shows how Git practices operationally support those fundamentals.

## How to use this repo

### 1. Start with the Master Standard
Use **MASTER_ARCHITECTURE.md** as the primary reference for:
- Branching strategy decisions.
- Merge vs rebase judgment calls.
- Reset/revert/reflog safety.
- Architectural review criteria aligned with the seven fundamentals.

### 2. Apply it as a working checklist
When you are about to:
- Introduce a new subsystem.
- Modify core invariants.
- Change branching or release practices.
- Refactor high-risk areas.

You should be able to point to:
- The specific fundamental involved.
- The corresponding Git practice that protects it.
- The rollback strategy if the change violates the invariant.

## Contribution and branch discipline

This repository is managed with an Infrastructure-as-Code mindset for documentation. Keep contributions predictable and traceable:

- Create topic branches from `main` using clear, human-readable names (e.g., `docs/master-architecture`, `feat/<scope>`); avoid tool-prefixed branch names.
- Keep one cohesive objective per PR and avoid bundling unrelated edits.
- Write intent-revealing commit messages and prefer rebasing topic branches over merging `main` into them.
- Open pull requests for every change and merge with rebase or squash to maintain a linear history.
- Apply branch protections on `main`: require PR review, passing checks, and block force pushes and deletions.

## Quick navigation

- **Master Standard:** [`MASTER_ARCHITECTURE.md`](./MASTER_ARCHITECTURE.md)
- Additional standards and topic-specific guides can be added later under a consistent naming scheme.

## Status

Active and evolving as the standards mature. The priority is maintaining strict scope, accuracy, and long-term readability.
