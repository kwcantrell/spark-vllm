# Tasks

## 1. Base branch

- [ ] 1.1 Set the commit author email chosen at approval (proposal "Decisions for the approver"), rename the unborn `master` to `main`, and create one empty root commit; verify with `git log --format='%h %ae' main` (exactly one commit, the chosen email) and `git show --stat --format= HEAD` (no files). Before retrying, check `git rev-list --count main` so a second root commit is never made
- [ ] 1.2 Human: push `main` before any other branch (`git push -u origin main`); verify `git ls-remote origin` lists only `refs/heads/main` at the root commit's SHA and `gh repo view --json defaultBranchRef` names `main`
- [ ] 1.3 Human: apply the main ruleset from `docs/security.md` "Setup a human must do" step 1; verify `gh api repos/kwcantrell/spark-vllm/rulesets` lists it

## 2. Pin the plan

- [ ] 2.1 Create branch `adopt-agent-lifecycle` and commit only `openspec/changes/adopt-agent-lifecycle/`; verify `git show --stat --format= HEAD` lists only paths under that directory

## 3. Install and README

- [ ] 3.1 Run `pre-commit install --hook-type pre-commit --hook-type pre-push`, then commit exactly the MANIFEST files, README.md and .gitignore; verify `git status --porcelain` is empty, `git ls-files .lifecycle-adoption .claude/settings.local.json` prints nothing, and `git ls-files` equals MANIFEST files + README.md + .gitignore + the change directory
- [ ] 3.2 Verify the install is unmodified: `cmp` each non-directory MANIFEST path against template `bf9eebb`; the only differences are `.github/CODEOWNERS` and `openspec/config.yaml`
- [ ] 3.3 Verify README.md links: every relative target exists (`test -e`) and every external URL returns HTTP 200 (`gh repo view` for GitHub repos)

## 4. Verify and open the PR

- [ ] 4.1 Run `LIFECYCLE_OVERRIDE="size_budget: generated install" scripts/check-change.sh --stage pr --base main`; verify every gate is PASS, SKIP or WARN except `tasks` (4.1 and 4.2 not yet ticked) and `approval` if not yet approved
- [ ] 4.2 Create labels `size-override` and `no-test-needed` (`gh label create`), push the branch, and open a PR to `main` whose body has `Tier: 2`, `Change: adopt-agent-lifecycle`, the size-override reason, a note that this PR's CI is self-certified, and the remaining forge-setup items from `docs/security.md`; verify `gh pr view --json labels,baseRefName,body`
