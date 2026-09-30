# Design

## Context

See proposal.md for why. The repo has no commits. The remote `origin`
(`git@github.com:kwcantrell/spark-vllm.git`) is public, has no refs and has no default branch.
The install and README are already in the working tree, uncommitted, written by `init.sh` from
agent-lifecycle-template `bf9eebb`.

## Goals / Non-Goals

**Goals:**
- Land the install unmodified, so later `init.sh`/`adopt.py` runs and template diffs stay clean.
- Give this first PR a real base so the PR-stage gates (artifacts-first, size, approval) run
  instead of skipping.
- Have `main` protected before any PR is opened against it.

**Non-Goals:**
- Retrofitting stack commands or source globs. Their gates WARN until the first serving change.

## Decisions

1. **Install with `init.sh`, not by copying files.** It records every created file in
   `.lifecycle-adoption/MANIFEST`. Alternative: copy the template tree by hand. Rejected: it
   would also copy the template's own archived changes, specs and tests, which describe the
   template, not this repo.
2. **Empty root commit on `main` as the PR base, pushed by the human.** With no base,
   `check-change.sh` skips artifacts-first and size, and there is nothing to open a PR against.
   An empty commit adds no content, so the whole install is reviewed in the PR. AGENTS.md rule 9
   says agents never push `main`, so the human pushes it (task 1.2). `main` is pushed before any
   other branch so GitHub makes it the default. Alternative: the agent pushes `main` under an
   exception. Rejected: it needs a carve-out to a non-negotiable rule.
3. **Ruleset before PR.** Right after pushing `main`, the human applies the main ruleset from
   `docs/security.md` step 1 (require a PR, required checks `gates`/`secrets`/`dependency-review`,
   code owner review). Until then `main` accepts direct pushes, and `git push` is only `ask`
   in `.claude/settings.json`.
4. **Two commits on the branch.** First the change artifacts only (artifacts-first gate), then
   the install, README and `.gitignore`. The install commit stages an explicit list (MANIFEST
   entries, README.md, .gitignore), not `git add -A`.
5. **`size-override` label instead of splitting.** The install is one unit: hooks, settings and
   the gate checker depend on each other, and a partial install fails its own gates. About 4,200
   of the 4,596 counted lines are generated files agents must not edit: OpenSpec skills and
   commands (2,726) and the `.agents/` mirror (1,476). The labels don't exist in a new repo, so
   the agent creates them (task 4.2).
6. **`skip_specs: true`.** The change adds tooling and docs and no product behavior. The
   template's own specs (installer, gate-checker, code-ownership) describe the template and
   were deliberately not installed.
7. **Verification stands in for test-first.** The config rule asks each task to name the test
   written first. Nothing here is code, so each task names the command that verifies it instead.

## Assumptions (each tested)

- Prerequisites are installed. `which openspec python3; python3 -c "import yaml; print(yaml.__version__)"`
  -> `~/.nvm/versions/node/v24.21.0/bin/openspec`, `/usr/bin/python3`, `6.0.1`.
- The remote is empty and public. `git ls-remote origin` -> no output;
  `gh repo view --json visibility,isEmpty` -> `PUBLIC`, `isEmpty: true`.
- The template commit is `bf9eebb`. `git -C <template> log -1 --format='%H %cs'` ->
  `bf9eebb2000f3d2ac24dda7ce147616f73c33d06 2026-09-29`.
- The gate's size count. Panel simulation, `scripts/check-change.sh --stage pr --base main --only size`
  -> `4596 changed lines > budget 400`. It uses the PR's own exemptions because the base has no config.
- Only two installed files differ from the template. Panel: `cmp` over MANIFEST files ->
  only `.github/CODEOWNERS` (rendered for the owner) and `openspec/config.yaml` (rendered with
  empty commands; the installer drops the template's explanatory comments).
- With an empty-commit base, artifacts-first and size run. Panel simulation ->
  `PASS artifacts-first plan pinned before code`.
- CI runs on the PR. Panel: `gh api repos/kwcantrell/spark-vllm/actions/permissions` ->
  `enabled: true`. The dependency graph is on by default for public repos.
- `gh` is authenticated as `kwcantrell`. `gh auth status` -> `Logged in to github.com account kwcantrell`.

## Risks / Trade-offs

- [Green CI on this PR is self-certified] -> the base has no config, so the gate uses the PR's
  own checker, exemptions and override labels, and `Approved-by:` is a text line. The human's
  review of the diff is the real control; the PR body says so. Later PRs are checked against
  `main`'s config.
- [With one collaborator, code owner review can only be met through admin bypass] -> a known gap
  (`docs/security.md` "Known gaps"). The human merges with bypass after reviewing.
- [`size-override` on a large PR hides a problem in the bulk] -> task 3.2 shows every file but two
  is byte-identical to template `bf9eebb`. Review can focus on README.md, `.gitignore`,
  CODEOWNERS, `openspec/config.yaml` and this change.
- [The first CI run shows `tasks` red] -> task 4.2 can only be ticked after the PR exists. A
  follow-up commit ticks it and re-runs CI.
- [Stack gates block legitimate work while the stack is undefined] -> commands and
  tests-with-code only WARN when empty.

## Migration Plan

- **Before merge:** close the PR and delete the branch. Pushed `main` (one empty commit, the
  default branch) stays. Removing it means deleting the branch on a public repo, which is
  effectively permanent, so don't push `main` unless you intend to keep the repo.
- **After merge:** rollback is itself a tier 2 change, since it touches the same high-risk paths.
  It must pass the gates it removes, or be merged by the admin with bypass. Remove the files
  with `git rm` on the MANIFEST list (`adopt.py undo` works only in this checkout, since MANIFEST
  is gitignored, and it deletes files without checking for later edits). Revert the merge commit
  only if the PR was merged with a merge commit.
