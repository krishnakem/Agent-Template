# OpenClaw Agent Plugin Template

**The boilerplate every one of my OpenClaw agents starts from.** It solves the boring-but-hard part once — long-running background work, progress streaming, cooperative cancellation, artifacts on disk — so a new plugin is just domain logic dropped into a proven lifecycle.

```bash
npm run create:plugin -- ../weather-agent \
  --id weather-agent --name "Weather Agent" --tool-prefix weather --init-git
```

---

## Why this exists

Agentic plugins share the same skeleton: start a session, kick off work in the background, let the caller poll for progress, allow a clean abort, write results somewhere durable, tear down. I kept rewriting that skeleton — and getting the abort/lifecycle edge cases subtly wrong — for every new agent. So I factored it into a template.

It intentionally ships **no** browser automation, scraping, LLM calls, or domain logic. The reusable asset is the *lifecycle*, and it's the same lifecycle that powers [Kowalski](https://github.com/krishnakem/kowalski-openclaw) and [Silicon Sandbox](https://github.com/krishnakem/JEPA-Silicon-Sandbox).

## The six tools it registers

| Tool | Purpose |
| --- | --- |
| `start_session` | Create a session, return a `session_id` |
| `run_agent` | Start the example runner in the background; returns immediately |
| `get_session_status` | Poll progress, recent events, final result, or error |
| `stop_run` | Abort the active run cooperatively via `AbortSignal` |
| `reset_all` | Dry-run (default) or confirmed deletion of scratch/output dirs |
| `end_session` | Abort in-flight work and forget the session |

Canonical flow: `start_session → run_agent → get_session_status → end_session`.

## Layout

```text
src/
  core/AgentSession.ts        # session id, dirs, run config, events, abort signal
  main/AgentRunner.ts         # generic runner contract + abort helpers
  main/ExampleAgent.ts        # replace-me example implementation
  plugin/index.ts             # OpenClaw register(api) + tool definitions
  plugin/session-registry.ts  # event buffer + active-run bookkeeping
  shared/config.ts            # template identity constants
skills/agent-template/SKILL.md  # playbook telling an OpenClaw agent how to drive this
scripts/create-plugin.mjs     # scaffold a new plugin from this template
scripts/test-plugin.ts        # local lifecycle smoke test
```

## Scaffolding a new plugin

Mark the repo as a GitHub template (`Settings → General → Template repository`) for the point-and-click path, or use the script for local scaffolding:

```bash
npm run create:plugin -- ../weather-agent \
  --id weather-agent \
  --name "Weather Agent" \
  --tool-prefix weather \
  --init-git
```

That command copies the template (skipping `.git`, `node_modules`, `dist`), rewrites the plugin id / display name / package name / config paths / skill name, optionally namespaces every tool with your prefix, and optionally starts fresh git history. Add `--install` to run `npm install` in the new plugin. `npm run create:plugin -- --help` lists every option.

## License

See repository.
