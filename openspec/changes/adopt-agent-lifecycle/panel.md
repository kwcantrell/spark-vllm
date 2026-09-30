# Panel: adopt-agent-lifecycle
Tier: 2 · Reviewers: assumption tester, failure and abuse, scope and simplicity · Date: 2026-09-29

## Assumption tester

- [x] [major] Task 4.2 applies a `size-override` label that doesn't exist in the new repo, so the PR opens without it and the size gate fails. Evidence: `gh label list -R kwcantrell/spark-vllm` -> GitHub defaults only. Resolved: task 4.2 creates `size-override` and `no-test-needed` first; design Decision 5
- [x] [major] Task 4.2 doesn't specify the PR body. The template's `Tier: 1` mismatches the proposal, so the change gate fails and blocks later gates. Evidence: simulated CI with body `Tier: 1` -> `FAIL change PR says tier 1, proposal says tier 2`. Resolved: task 4.2 requires `Tier: 2`, `Change: adopt-agent-lifecycle` and the override reason
- [x] [major] Task 4.1's expected result is wrong: a simulated run gives FAIL for approval, panel, tasks and size. Evidence: `scripts/check-change.sh --stage pr --base main` (sim) -> 4 FAILs. Resolved: panel.md now exists; 4.1 runs with `LIFECYCLE_OVERRIDE` for size and expects only `tasks` (and `approval` before approval) to fail
- [x] [minor] The size figures (5,875 / about 5,900) don't match what the gate counts. Evidence: sim `--only size` -> `4596 changed lines`. Resolved: proposal and design use 4,596
- [x] [minor] Task 3.2's vague "OpenSpec-generated files" exclusion is unnecessary, and `cmp` errors on directory entries. Evidence: `cmp` over MANIFEST -> only CODEOWNERS and config.yaml differ. Resolved: 3.2 compares every non-directory entry and expects exactly those two differences
- [x] [minor] The rendered `openspec/config.yaml` lost the template's explanatory comments. Evidence: `diff $T/openspec/config.yaml openspec/config.yaml`. Resolved: recorded in design Assumptions; restoring them would edit a template output, which this change excludes
- [x] [minor] Task 3.3 doesn't check external links. Evidence: `grep -oE '\]\([^)#]+' README.md` -> 3 external links. Resolved: 3.3 checks them
- [x] [minor] The design's push of `main` conflicts with AGENTS.md "Never push to `main`". Evidence: `AGENTS.md:50`. Resolved: the human pushes `main` (task 1.2, design Decision 2)

## Failure and abuse

- [x] [major] There's no root `.gitignore`, and task 3.1 would sweep in stray files such as `.claude/settings.local.json` and `.env*`. Evidence: `cat .gitignore` -> missing. Resolved: `.gitignore` added to the change; 3.1 stages an explicit list and checks `git ls-files` against it
- [x] [major] The repo is public, so the local `user.email` becomes permanent public history with the first commit. Evidence: `gh repo view --json visibility` -> PUBLIC; `git config user.email` -> personal address. Declined by human: keep cantrell.kalen@gmail.com as the author email (2026-09-29)
- [x] [major] Nothing protects `main` after it's pushed. Evidence: `.claude/settings.json` -> `git push` is `ask`, not deny; no ruleset. Resolved: the human applies the ruleset in task 1.3, before the PR (design Decision 3)
- [x] [major] "Green checks by hand" is weak: with no config on the base, this PR's CI uses its own checker, exemptions and labels, and `Approved-by:` is text. Evidence: `check_change.py:103`, `:278`. Resolved: stated as a risk in design; the PR body must say CI is self-certified; the human's diff review is the control
- [x] [major] The post-merge rollback doesn't work as written: MANIFEST is gitignored and local-only, `undo` ignores later edits, revert needs a merge commit, and rollback is itself tier 2. Evidence: `adopt.py:390-396`. Resolved: Migration Plan rewritten
- [x] [major] The pre-merge rollback leaves `main` pushed on a public repo. Evidence: design Migration Plan. Resolved: stated in the Migration Plan, and the human pushes `main` (task 1.2)
- [x] [minor] Retrying task 1.1 can create a second root commit, and pushing the branch before `main` makes it the default. Evidence: `gh repo view` -> `defaultBranchRef` empty. Resolved: 1.1 checks `git rev-list --count`; 1.2 pushes `main` first and verifies the default
- [x] [minor] The `size-override` label is missing. Resolved: duplicate of the assumption tester's label finding
- [x] [minor] The first CI run fails `tasks` because 4.2 can only be ticked after the PR exists. Resolved: stated in design Risks
- [x] [minor] `dependency-review` needs the dependency graph. Evidence: assumption tester -> public repo, graph on by default. Resolved: no action needed

## Scope and simplicity

- [x] [major] The proposal never mentions pushing `main`, although AGENTS.md rule 9 forbids it. Evidence: `AGENTS.md:50`. Resolved: the human pushes `main`, and the proposal says so
- [x] [minor] The size count is wrong. Resolved: duplicate; now 4,596
- [x] [minor] The size-override argument omits that about 4,200 counted lines are generated files agents must not edit. Evidence: `.agents/**` 1,476 + `.claude/skills|commands` 2,726. Resolved: added to design Decision 5
- [x] [minor] `pre-commit install` (template setup step 4) is missing. Evidence: `docs/lifecycle.md:79`. Resolved: added to task 3.1
- [x] [minor] The tasks don't name a test written first. Evidence: `openspec/config.yaml:16`. Resolved: design Decision 7
- [x] [minor] `skip_specs: true` is appropriate. Resolved: no action needed
- [x] [minor] "Following best practices" in the README is an empty claim, and the README's Prerequisites, Checking-a-change and Layout sections duplicate AGENTS.md and docs/lifecycle.md. Evidence: `README.md:4`, `README.md:28-52`. Resolved: human chose to trim; README now drops the phrase and links to the lifecycle docs instead of repeating them
- [x] [minor] Task 3.2's exclusion is vague. Resolved: duplicate; 3.2 no longer excludes anything
