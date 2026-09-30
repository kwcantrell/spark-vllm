# Adopt the agent lifecycle and add a README

Tier: 2
Tier reason: installs the lifecycle itself (AGENTS.md, CLAUDE.md, .claude/, .github/, scripts/, openspec/config.yaml), all high-risk paths.

Approved-by: Kalen

## Why

spark-vllm is a new, empty repo that will hold the components for serving models with vLLM on a
DGX Spark. Most of that work will be done by AI agents, so the review and verification gates
need to exist before the first line of serving code, not after. The repo also has no README
saying what it is for.

## What Changes

- Install agent-lifecycle-template at commit `bf9eebb` with its own installer
  (`scripts/init.sh --owner @kwcantrell`). This adds AGENTS.md, CLAUDE.md, `.claude/` (settings,
  hooks, skills, `/opsx:*` commands), `.agents/skills/`, `.github/` (lifecycle and release
  workflows, CODEOWNERS for `@kwcantrell`, PR template, dependabot), `openspec/` (config and
  empty specs), `docs/` (lifecycle guide, security model, ADRs 0000-0018, templates), `scripts/`
  (gate checker, skill sync, installer helper) and `.pre-commit-config.yaml`.
- Add `README.md` stating the repo's purpose, target platform and current status (lifecycle only,
  no serving components yet), and pointing to the lifecycle docs.
- Add a root `.gitignore` so machine-local files (`.claude/settings.local.json`, `.env*`,
  Python caches, virtualenvs) are never committed.
- Create the repo's first history. The unborn `master` becomes `main`, holding one empty root
  commit as the PR base. **The human pushes `main`** and then applies the main-branch ruleset
  before the PR opens. The agent never pushes `main`, keeping AGENTS.md rule 9 without exception.
- On approval, the agent may also: push branch `adopt-agent-lifecycle`, create the
  `size-override` and `no-test-needed` labels, and open the PR.
- Stack commands (`lifecycle.commands`) and `source_globs` stay empty. There is no stack yet;
  the first serving change sets them.

## Decisions for the approver

- **Commit author email.** The repo is public, so the author email of every commit becomes
  permanent public history. Before task 1.1, choose the GitHub noreply address or accept the
  current `user.email` (task 1.1 records the choice).

## Non-goals

- Any vLLM serving component: container images, launch scripts, model configs, benchmarks.
- Choosing stack commands, source globs or CI toolchain setup ("Stack setup" steps).
- Forge settings other than the main ruleset: secret scanning, workflow token permissions and
  the rest of `docs/security.md` "Setup a human must do" are listed in the PR body for the human.
- Changing any template file. Files land exactly as the installer wrote them.

## Capabilities

### New Capabilities

None. This change adds tooling and docs only, so `.openspec.yaml` sets `skip_specs: true`.

### Modified Capabilities

None.

## Impact

- Every later change must pass the lifecycle gates in CI and the Stop hook locally.
- Agents in this repo run under `.claude/settings.json` deny rules and sandbox, and the
  `Approved-by:` guard hook.
- The PR needs the `size-override` label: the size gate counts 4,596 lines against a 400-line
  budget (see design.md).
- Default branch becomes `main`.
