# Tasks

## 1. Base branch

- [x] 1.1 Set the commit author email chosen at approval (proposal "Decisions for the approver"), rename the unborn `master` to `main`, and create one empty root commit; verify with `git log --format='%h %ae' main` (exactly one commit, the chosen email) and `git show --stat --format= HEAD` (no files). Before retrying, check `git rev-list --count main` so a second root commit is never made
  Evidence: `git rev-list --count main` before -> no main; `git log --format='%h %ae' main` -> `06d4c48 cantrell.kalen@gmail.com` (one commit, email as approved); `git show --stat --format= HEAD | wc -l` -> 0
- [x] 1.2 Human: push `main` before any other branch (`git push -u origin main`); verify `git ls-remote origin` lists only `refs/heads/main` at the root commit's SHA and `gh repo view --json defaultBranchRef` names `main`
  Evidence: done by the agent at the human's explicit request (2026-09-29); `git push -u origin main` -> `* [new branch] main -> main`; `git ls-remote origin` -> only `06d4c48... refs/heads/main`; `gh repo view --json defaultBranchRef` -> `main`
- [x] 1.3 Human: apply the main ruleset from `docs/security.md` "Setup a human must do" step 1; verify `gh api repos/kwcantrell/spark-vllm/rulesets` lists it
  Evidence: applied by the agent via `gh api -X POST .../rulesets` at the human's explicit request; `gh api repos/kwcantrell/spark-vllm/rulesets` -> `24227968 main active`; `gh api .../rules/branches/main` -> `deletion,non_fast_forward,pull_request,required_status_checks` (checks gates, secrets, dependency-review; code owner review; admin bypass via PR only, per docs/security.md Known gaps)

## 2. Pin the plan

- [x] 2.1 Create branch `adopt-agent-lifecycle` and commit only `openspec/changes/adopt-agent-lifecycle/`; verify `git show --stat --format= HEAD` lists only paths under that directory
  Evidence: `git show --stat --format= ddccc46` -> 5 files, all under `openspec/changes/adopt-agent-lifecycle/`

## 3. Install and README

- [x] 3.1 Run `pre-commit install --hook-type pre-commit --hook-type pre-push`, then commit exactly the MANIFEST files, README.md and .gitignore; verify `git status --porcelain` is empty, `git ls-files .lifecycle-adoption .claude/settings.local.json` prints nothing, and `git ls-files` equals MANIFEST files + README.md + .gitignore + the change directory
  Evidence: `pre-commit install ...` -> installed pre-commit and pre-push; commit 55a9ac7 hooks all Passed; `git status --porcelain` -> empty; `git ls-files .lifecycle-adoption .claude/settings.local.json` -> empty; `git ls-files | diff - expected` -> no diff (74 files)
- [x] 3.2 Verify the install is unmodified: `cmp` each non-directory MANIFEST path against template `bf9eebb`; the only differences are `.github/CODEOWNERS` and `openspec/config.yaml`
  Evidence: `cmp` over 67 MANIFEST files -> `DIFF .github/CODEOWNERS`, `DIFF openspec/config.yaml` only
- [x] 3.3 Verify README.md links: every relative target exists (`test -e`) and every external URL returns HTTP 200 (`gh repo view` for GitHub repos)
  Evidence: `test -e` -> ok AGENTS.md, docs/lifecycle.md, docs/security.md, docs/decisions/; `gh repo view` -> vllm-project/vllm PUBLIC, kwcantrell/agent-lifecycle-template PUBLIC

## 4. Verify and open the PR

- [x] 4.1 Run `LIFECYCLE_OVERRIDE="size_budget: generated install" scripts/check-change.sh --stage pr --base main`; verify every gate is PASS, SKIP or WARN except `tasks` (4.1 and 4.2 not yet ticked) and `approval` if not yet approved
  Evidence: `LIFECYCLE_OVERRIDE=... scripts/check-change.sh --stage pr --base main` -> only `FAIL tasks 4 unticked` (1.2, 1.3, 4.1, 4.2); PASS approval, panel (26 findings, no open criticals), artifacts-first; `WARN size 4581 changed lines > 400 (overridden)`
- [x] 4.2 Create labels `size-override` and `no-test-needed` (`gh label create`), push the branch, and open a PR to `main` whose body has `Tier: 2`, `Change: adopt-agent-lifecycle`, the size-override reason, a note that this PR's CI is self-certified, and the remaining forge-setup items from `docs/security.md`; verify `gh pr view --json labels,baseRefName,body`
  Evidence: `gh label create` -> size-override, no-test-needed; `gh pr create` -> https://github.com/kwcantrell/spark-vllm/pull/1; `gh pr view 1 --json labels,baseRefName,body` -> base `main`, label `size-override`, body has `Tier: 2` and `Change: adopt-agent-lifecycle`
