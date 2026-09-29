# AGENTS.md — pod-hermes

Standalone candy repo for the `hermes` candy — the Nous Research Hermes
self-improving AI agent, installed as a pixi CLI with a supervised entrypoint.
The candy lives in `charly.yml` at the repo root plus its build inputs.

Canonical files:

- `charly.yml` — the `hermes:` candy entity (description, `require`, `env`,
  `env_accept`, `secret_accept`, `mcp_accept`, `distro`, `volume`, `alias`,
  `service`, `plan`) and its `skill:` entity.
- `build.sh` — the pixi-builder stage (clone `hermes-agent`, pip install, npm
  deps, stage the source tree).
- `hermes-entrypoint` — the runtime wrapper (volume init, one-time provider
  auto-config, `.env` sync).
- `pixi.toml` / `pixi.lock` — the Python environment for the agent.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-hermes:hermes` — the owning skill: candy properties, the provider
  priority, the env/secret contract, and cross-container service discovery. Load
  before editing, building, deploying, or troubleshooting this candy.
- `/charly-hermes:hermes-full-layer` — the metalayer composing the AI CLIs and
  toolchains.
- `/charly-pod:pod` — the `kind: pod` / deploy schema reference (this candy is
  composed into a box; tree-position nesting, volumes, ports).
- `/charly-check:check` — the check/R10 framework: the `check:` step verbs and
  `charly check run <bed>`.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs, `env_accept` / `secret_accept` / `mcp_accept`, service
  declarations).
- `/charly-build:secrets` — the credential store backing the `secret_accept`
  entries (`charly/api-key/*`).
- `/charly-internals:git-workflow` — before any git/PR action.

## Build / validate / test

- `charly box validate` at the repo root — the structural check: the manifest
  must parse and validate at the installed charly.
- The live R10 witness is a composing box's `check` bed; the candy's own `check:`
  steps assert the pixi `hermes` binary, the `hermes-entrypoint` wrapper, the
  staged source tree, the mounted `/opt/data` volume, and the running `hermes`
  service.
- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no**
  per-repo candy gate. Its only workflow file is
  `.github/workflows/tag-on-merge.yml`.

## Modify this repo

- Edit the `hermes:` candy entity in `charly.yml`; the `skill:` entity in the same
  file is the owning skill's source — a candy change and its skill change land
  together.
- The upstream clone + pip install + npm deps live in `build.sh`; the runtime
  volume init and provider auto-config live in `hermes-entrypoint`. Keep the two
  in step (the entrypoint's sentinel and the config it writes).
- Python dependency changes belong in `pixi.toml` / `pixi.lock`, not in the plan.
- The `data` volume at `/opt/data` is the persistent store; keep the service's
  `working_directory` and `HERMES_HOME` in step.
- The `skill:` entity is the source for `/charly-hermes:hermes`; never edit the
  generated `SKILL.md` — regenerate it.

## Landing

- PR-only. Every change lands through a pull request; the org-required
  `charly/pr-validator` validates the diff and body and arms native auto-merge on
  PASS. Direct pushes to `main` are blocked.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time); the PR
  body IS the changelog.
- The authoritative rulebook is the umbrella `AGENTS.md` in
  `opencharly/opencharly` and `charly/AGENTS.md` in the charly repo. Do not
  restate its rules here.
