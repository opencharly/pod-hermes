# pod-hermes

The `hermes` candy of the OpenCharly candy library, as a standalone repo
(kind-prefixed naming). It ships the [Hermes](https://github.com/NousResearch/hermes-agent)
self-improving AI agent by Nous Research as a supervised service.

## What it provides

Installs the upstream Nous Research `hermes-agent` as a non-editable pip package
into the pixi default environment, so the `hermes` CLI lands at
`${HOME}/.pixi/envs/default/bin/hermes`. It stages the full `hermes-agent` source
tree under `$HOME` (for the bundled `.env` / config examples, skills, and the
WhatsApp bridge) and installs an executable `hermes-entrypoint` wrapper on the
user PATH.

At deploy time the entrypoint runs under supervisord, mounts the persistent
`/opt/data` volume, and one-time auto-configures an LLM provider from the
supplied API key (guarded by a `# charly:auto-configured` sentinel in
`config.yaml`).

| Property | Value |
|---|---|
| Services | `hermes` (`hermes-entrypoint`, `restart: always`), `hermes-whatsapp` (node bridge) |
| Requires | `layer-nodejs`, `layer-supervisord`, `layer-ripgrep`, `layer-ffmpeg`, `pod-pipewire` |
| Env | `HERMES_HOME=/opt/data`, `TERMINAL_ENV=local` |
| Volume | `data` at `/opt/data` (sessions, skills, memories, logs, config, `.env`) |
| Aliases | `hermes`, `hermes-agent` |
| env_accept | `OLLAMA_HOST`, `HERMES_MODEL`, `CHARLY_MCP_SERVERS`, `BROWSER_CDP_URL`, `IMMICH_API_URL` |
| secret_accept | `OPENROUTER_API_KEY`, `OLLAMA_API_KEY`, `TELEGRAM_BOT_TOKEN`, `SLACK_BOT_TOKEN`, `DISCORD_BOT_TOKEN`, `IMMICH_API_KEY` |

## Provider auto-configuration

The entrypoint picks an LLM provider by priority, first set wins:

| Priority | Env var | Provider |
|---|---|---|
| 1 | `OLLAMA_HOST` | Local Ollama |
| 2 | `OLLAMA_API_KEY` | Ollama Cloud |
| 3 | `OPENROUTER_API_KEY` | OpenRouter |

Override the default model with `HERMES_MODEL`. API keys are synced to `.env` on
every start (rotation-safe). To reconfigure, delete `/opt/data/config.yaml` and
restart.

```bash
charly box build hermes
charly config hermes -e OLLAMA_API_KEY=your-key   # or OPENROUTER_API_KEY
charly start hermes
charly shell hermes -c "hermes chat"
```

## Layout

- `charly.yml` — the `hermes:` candy entity plus its `skill:` entity.
- `build.sh` — the pixi-builder stage: clone + pip install + npm deps + source
  staging.
- `hermes-entrypoint` — the runtime wrapper: volume init + provider auto-config.
- `pixi.toml` / `pixi.lock` — the Python environment.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `CHANGELOG/` — per-CalVer history.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-hermes:hermes` — the candy properties, the provider
  priority, the env/secret contract, and the cross-container service discovery.
- `/charly-hermes:hermes-full-layer` — the metalayer composing the AI CLIs and
  toolchains.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
