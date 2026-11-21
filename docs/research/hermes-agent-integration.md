# Driving Hermes Agent from another program

- **Date researched:** 2026-11-21
- **Hermes version examined:** tag `v0.23.3` (latest stable release, published 2026-11-20), commit `caafe5389ae585ef77387a962aee05b1bcf17029` (committed 2026-11-19).
- **Question:** How can another program drive Hermes Agent, and what does a GUI (Budha: Python, GTK4, libadwaita, likely Flatpak) need to know to wrap it well?

## Citation convention

Unless a citation says otherwise, `path:line` refers to the `NousResearch/hermes-agent` repository at commit `caafe53` (tag `v0.23.3`). Docs pages are cited by their source file under `website/docs/`; they are published at `https://hermes-agent.nousresearch.com/docs/<same path without .md>`. I read the code but did not run it. **Unverified** marks a claim I could not confirm from a primary source. **Inference** marks my own reasoning from cited facts.

## Sources

- Repository: https://github.com/NousResearch/hermes-agent (canonical: the GitHub API reports `full_name: NousResearch/hermes-agent`, homepage `https://hermes-agent.nousresearch.com`, license MIT, language Python). The README links back to it (`README.md:12`).
- Releases: https://github.com/NousResearch/hermes-agent/releases, read through the GitHub REST API on 2026-11-21.
- Official docs: https://hermes-agent.nousresearch.com/docs/ (source in `website/docs/`).
- Key docs pages:
  - `website/docs/developer-guide/programmatic-integration.md` (the official "how to drive Hermes from another program" page)
  - `website/docs/developer-guide/gateway-internals.md`
  - `website/docs/developer-guide/acp-internals.md`
  - `website/docs/user-guide/features/acp.md`
  - `website/docs/user-guide/features/api-server.md`
  - `website/docs/reference/cli-commands.md`
  - `website/docs/user-guide/security.md`
  - `website/docs/user-guide/configuration.md`
  - `website/docs/user-guide/sessions.md`
  - `website/docs/developer-guide/session-storage.md`
  - `website/docs/getting-started/installation.md`
  - `website/docs/getting-started/platform-support.md`
  - `website/docs/user-guide/desktop.md`
  - `website/docs/guides/python-library.md`
- Key source files: `acp_adapter/entry.py`, `acp_adapter/server.py`, `acp_adapter/gateway_server.py`, `hermes_cli/gateway_client.py`, `hermes_cli/gateway_chat.py`, `hermes_cli/stream_json.py`, `gateway/control_socket.py`, `gateway/platforms/api_server.py`, `tui_gateway/AGENTS.md`, `tui_gateway/contracts/*.py`, `apps/shared/src/gateway-contract.openrpc.json`, `pyproject.toml`.

---

## 1. What Hermes Agent is

- **What it is:** an LLM agent from Nous Research. It has a terminal UI, a messaging gateway (Telegram, Discord, Slack and others), persistent memory, skills the agent writes for itself, cron jobs and subagents (`README.md:19-31`). It works with many model providers. You switch providers with `hermes model` (`README.md:21`).
- **License:** MIT (`pyproject.toml:13`, `LICENSE`, and the GitHub API license field).
- **Language:** Python, with TypeScript front ends. The Ink TUI is in `ui-tui/`, the Electron desktop app in `apps/desktop/` (`apps/desktop/package.json:195` pins `electron` 44.10.2), and a web UI in `web/`. Only Python 3.14 is supported. `requires-python` is `>=3.11,<3.15` only so that old installs can still update (`pyproject.toml:6-11`).
- **Current version:** `v0.23.3`, published 2026-11-20 (GitHub releases API). The version committed in `pyproject.toml` is always `0.0.0`. Release jobs stamp the real version at build time (`pyproject.toml:3`; `website/docs/developer-guide/stable-releases.md:8-9`). `hermes --version` prints it (`website/docs/reference/cli-commands.md:27`).
- **Versioning and release cadence** (from the GitHub releases API):
  - Stable tags were CalVer up to `v2026.9.24` (2026-09-24). From `v0.21.6` (2026-10-08) they are SemVer `0.x`.
  - Stable releases ship every few days: `v0.23.0` on 11-05, `v0.23.1` on 11-10, `v0.23.2` on 11-16, `v0.23.3` on 11-20.
  - Canary prereleases (`vX.Y.Z+canary.<timestamp>`) are published about daily.
  - A source install follows the **stable** channel by default. Users can switch to `main` with `hermes update --set-channel main` (`website/docs/getting-started/updating.md:33-43`).
  - The SemVer version is `0.x`, and the docs announce breaking wire changes between releases (see §2.2). Inference: treat every release as possibly breaking.
- **Installation (supported paths):**
  - Linux, macOS and WSL2: `curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash`. Windows: `install.ps1` (`README.md:37-51`; `website/docs/getting-started/installation.md:49-62`).
  - The script clones the source, bootstraps `uv`, and hands dependencies to Hermes' own package manager "PM". PM supplies pinned Python, Node.js, npm, pnpm, ripgrep and FFmpeg, and installs the `all` Python extra (`installation.md:75-88`).
  - Install layout for the POSIX script: code in `~/.hermes/hermes-agent/`, launcher at `~/.local/bin/hermes`, user data in `~/.hermes/`. `HERMES_HOME` overrides the data location (`installation.md:112-125`).
  - Other installs: macOS and Windows desktop bundles, Docker, Nix (Tier 2), and Termux APT (Tier 2) (`installation.md:22-43`; `platform-support.md:13-36`).
  - **pip, uv and PyPI installs are explicitly unsupported**: "installs via `pypi` (e.g. `uv tool install hermes-agent`, `pip install hermes-agent`, etc.)" (`website/docs/getting-started/platform-support.md:62`). Brew and AUR are also unsupported (`platform-support.md:60,63`). The Python library guide says Hermes "does not publish a supported wheel or source distribution" (`website/docs/guides/python-library.md:25`).
- **Supported platforms:**
  - Tier 1: macOS on Apple Silicon; Windows 10/11 on x86_64 and aarch64; Linux and WSL2 on x86_64 and aarch64 ("glibc, systemd, FHS ... likely to work"); Docker (`platform-support.md:13-22`).
  - Tier 2: Nix and Termux (`platform-support.md:26-36`).
  - **No official Linux desktop app is released.** "Linux desktop AppImages are built on canary tags as diagnostic artifacts that no release offers yet" (`platform-support.md:49-50`). On Linux, `hermes desktop` builds the Electron app from the source checkout (`website/docs/user-guide/desktop.md:609`).

## 2. Programmatic interfaces

The official page lists three protocols for external programs: ACP, the TUI gateway JSON-RPC, and an OpenAI-compatible HTTP API. It says "all three drive the same `AIAgent` core" (`website/docs/developer-guide/programmatic-integration.md:9-17`). It also says there is **no** `--mode rpc` flag (`programmatic-integration.md:297-299`). Beyond those three, the CLI has non-interactive and JSONL modes, and there is an in-process Python API.

**Architecture fact that affects every option below.** In v0.23.3, sessions and agent execution belong to a per-profile **gateway daemon** (the "owner"). The other surfaces are clients of it:

- The CLI's one-shot and classic launch goes "through the canonical gateway, never AIAgent" (`hermes_cli/gateway_chat.py:1`).
- `hermes acp` is a "viewer" of the gateway daemon (`acp_adapter/entry.py:205-213`; `acp_adapter/gateway_server.py:1`; `acp_adapter/server.py:284-288` routes the default constructor to `GatewayACPAgent`).
- The Ink TUI attaches over WebSocket and "no longer starts `tui_gateway.entry` or owns an agent loop" (`ui-tui/CANONICAL_GATEWAY.md:1-6`; `tui_gateway/AGENTS.md:9-21`).
- The desktop app talks to the same gateway (`website/docs/user-guide/desktop.md:634`).
- `connect_gateway()` "ensures one, starting it if needed". A daemon that a client starts exits on its own once idle (`hermes_cli/gateway_client.py:330-366`).

### 2.1 CLI non-interactive modes

| Mode | What it does | Streams? | Approvals |
|---|---|---|---|
| `hermes chat -q "…" --format stream-json` | One JSON object per stdout line. Event types: `system/init` (model, session_id), `text` (streamed assistant delta), `tool_use` (name, input), `tool_result` (output capped at 5000 chars, `duration_ms`, `is_error`), and a terminal `result` (exit_code, text, token counts, `error`, `replay_gap`). Diagnostics and `session_id:` go to stderr (`website/docs/reference/cli-commands.md:167-187`; confirmed in `hermes_cli/stream_json.py:1-31`, which has `_TOOL_OUTPUT_CAP = 5000`). Requires `-q`/`--query-file` and refuses `--tui` (`stream_json.py:19-30`). | Text deltas and tool start/result: yes. No reasoning events are listed. | None interactive. One-shot `-q` sessions use `approvals.single_query_mode`, default `deny` (`website/docs/user-guide/security.md:51`). |
| `hermes chat --oneshot -q` / `-Q` | Answers and exits. Exit codes: 0 completed, 1 failed/partial/budget, 130 interrupted (`cli-commands.md:189-204`). `--query-file PATH` (`-` = stdin) avoids shell interpretation (`cli-commands.md:129`). | Human text only. | As above. |
| `hermes -z "…"` | Final reply text only, with nothing else on stdout or stderr. `--usage-file` writes a JSON usage and cost report. Different exit codes: 2 failed/partial, 1 "answered nothing" (`cli-commands.md:212-257`). | No. | As above. |
| `hermes-agent --query` | Legacy single-query runner (`cli-commands.md:41-43`; script `agent.legacy_cli:main` at `pyproject.toml:595`). | No. | Not checked. |

Per-run flags that matter for a host (`cli-commands.md:23-39,124-150`):

- `--resume <id|title|latest>`, `--continue`, `--in <dir>`
- `-m/--model`, `--provider`, `-t/--toolsets`, `-s/--skills`, `--max-turns`
- `--yolo` (skip approvals), `--ignore-user-config`, `--ignore-rules`, `--safe-mode`
- `--source <tag>`: "Use `tool` for third-party integrations that should not appear in user session lists"
- `-p/--profile`

Some options are refused on the gateway-backed classic and one-shot path. These include `--image` and `--worktree`; the refusal points to the TUI or Desktop instead (`hermes_cli/gateway_chat.py:14-30`). Note that the reference table still lists `--image` (`cli-commands.md:137`).

**Assessment:** stable and documented, but one-shot only. A host cannot answer approvals, clarify questions or sudo prompts mid-turn, so dangerous commands are denied by default. It does not fit an interactive chat GUI. It does fit a simple first prototype.

### 2.2 TUI gateway JSON-RPC (`hermes-gateway-v1`): the official route for custom desktop hosts

- **Positioning:** "You're writing a custom desktop / web / TUI host and want every Hermes feature (slash commands, approvals, clarify, multi-agent, session branching) → TUI gateway JSON-RPC" (`programmatic-integration.md:277`). It is the backend for the Ink TUI, the Electron Desktop app and the dashboard (`programmatic-integration.md:39`; `tui_gateway/AGENTS.md:3-5`).
- **Transport:** newline-delimited JSON-RPC over WebSocket at `/api/ws`, subprotocol `hermes-gateway-v1` (`tui_gateway/AGENTS.md:27-31`; `programmatic-integration.md:39`).
- **How a local client connects:**
  1. Run `hermes gateway attach --json`. Use `--start` on first launch and `--recover` after the owner dies. Rate-limit `--recover` (`gateway-internals.md:584-596`).
  2. On success it prints one JSON line containing `url` (`ws://127.0.0.1:<port>/api/ws`), `protocols` (`["hermes-gateway-v1", "hermes-gateway-ticket.<ticket>"]`) and `endpoint` (`profile_id`, `instance_id`, `capabilities`, `code_sha`) (`gateway-internals.md:605-615`).
  3. Dial from loopback with **no `Origin` header**, then call `runtime.describe`, then `session.create` or `session.resume` (`gateway-internals.md:610-615`).
  4. The ticket is single-use and expires after 30 s, so mint a new one for every connection and reconnect (`gateway-internals.md:620-627`).
  5. Exit codes 0 and 2–7 report the verdict: incompatible, auth mismatch, deadline, draining/updating, inaccessible (`gateway-internals.md:458-466`).
  6. If `client_code_sha != endpoint.code_sha`, the gateway has outlived `hermes update` and should be restarted (`gateway-internals.md:613-615`).
- **Spawn-free alternative, the local control socket:** a documented "public client contract" with three verbs: `identify`, `resolve-profile` and `session-ticket`. Envelope: one JSON line in, one JSON line out (`gateway-internals.md:479-532`).
  - Socket location on POSIX: `$HERMES_HOME/gateway.sock`, or a `gateway.sock.path` pointer file to `…/hermes-gw-<hash>/control.sock` under `$XDG_RUNTIME_DIR` or the temp dir (`gateway-internals.md:534-556`; `gateway/control_socket.py:33-34,60-81`).
  - Same-user check: the gateway reads the peer uid via `SO_PEERCRED` (`gateway-internals.md:558-562`).
  - Replayable conformance vectors: `apps/shared/src/node/control-socket.vectors.json` (`gateway-internals.md:578-580`).
- **Machine-readable contract:**
  - `apps/shared/src/gateway-contract.openrpc.json` is OpenRPC 1.3.2, title "Hermes TUI/Desktop gateway". It has 263 methods plus `x-canonical-methods`, `x-canonical-namespaces`, `x-server-requests` and `x-notifications`, and is generated from Pydantic models in `tui_gateway/contracts/`.
  - TypeScript types are generated alongside it in `apps/shared/src/gateway-contract.generated.ts` (`tui_gateway/AGENTS.md:46-64`).
  - The only ready-made client is Node/TypeScript (`@hermes/shared/gateway-client`) (`programmatic-integration.md:41-82`). **There is no Python client.** Budha would have to write its own.
- **Which methods actually work on the shared socket:**
  - The docs list 112 methods "served by the gateway", 16 "refused (`-32601`)" and 160 "answered by an older handler". The last group has "no owner decision yet ... treat `-32601` ... as 'not available here'" (`programmatic-integration.md:84-105`).
  - The 16 refused methods include `session.close`, `session.delete`, `session.title`, `session.branch`, `session.compress`, `session.usage` and `session.undo` (`programmatic-integration.md:99`).
  - The OpenRPC `x-canonical-methods` list (53 names) is not the list of methods a client may call (`programmatic-integration.md:88`).
- **Core flow:**
  - `session.create` takes `model`/`provider` overrides and refuses bad pairs with `-32602` (`programmatic-integration.md:115-117`). It also takes `source`/`client` attribution (`programmatic-integration.md:72-80`).
  - `prompt.submit` takes `text`, optional `attachments: [{path, mime}]`, `queued`, `finite` and `unattended` (`tui_gateway/contracts/canonical_sessions.py:249-274`; `gateway-internals.md:103-118`).
  - Other session methods: `session.interrupt`, `session.steer`, `session.resume`, `session.list` (picker mode, cursor paging), `session.transcript` (paged display rows), `session.history` and `session.events.since` (`programmatic-integration.md:101,111`; `canonical_sessions.py:364-376`).
  - Rewind and edit use `prompt.submit` with `truncate_before_row_id` and `confirm_truncate`. A **next-release breaking change** is announced: an unconfirmed deep truncate will be refused with code `4033` (`programmatic-integration.md:119-135`).
- **Streaming events (`programmatic-integration.md:137-139`; full list in `tui_gateway/contracts/events.py`):**
  - Text: `message.start`, `message.delta`, `message.interim`, `message.complete`.
  - Reasoning: `reasoning.delta`, `thinking.delta`.
  - Tools: `tool.start`, `tool.generating`, `tool.progress`, `tool.complete` (with diff and todo snapshot).
  - Other: `todo.updated`, `status.update`, `session.usage`, `session.title`, `approval.cancelled`, `sessions.changed`, `skills.changed`, `cron.changed`, and more.
- **Approvals and questions:** these are **server→client JSON-RPC requests**, not events (`programmatic-integration.md:141-154`).
  - Methods: `approval` → `{choice}`, `clarify` → `{answers}`, `sudo`/`secret` → `{value}`, plus vault, connection and desktop bridges.
  - The approval request carries `request_id`, `command` (redacted server-side), `description`, `choices`, `allow_permanent`, `allow_session`, `smart_denied` and `tool_name`. Choices are `once | session | always | deny` (`tui_gateway/contracts/server_requests.py:166-194`).
  - **A client must call `client.capabilities {"server_requests": true}` once per connection after `gateway.ready`.** Otherwise every approval is failed (withdrawn) immediately. The docs call this change "breaking for existing WebSocket integrations" (`programmatic-integration.md:152`).
  - A client that reconnects gets still-open prompts back in `open_requests` on `session.resume` and `session.events.since` (`programmatic-integration.md:154`). It gets the in-flight turn back in `inflight` (`programmatic-integration.md:156-158`).
  - The canonical socket also exposes `approval.respond` with `prompt_id`, `choice` and `execution_generation` (`canonical_sessions.py:305-333,373`). That is what the ACP viewer uses (`acp_adapter/gateway_server.py:1215`).
- **Multi-client:** several clients can attach to one live session. Streams go to all of them (`programmatic-integration.md:113`).
- **Stdio variant:** the same frames can run over stdio through `python -m tui_gateway.entry` (`programmatic-integration.md:39`). This is not a `hermes` subcommand.
  - The dispatch ledger says the shared WebSocket is the path the project builds on. Ink no longer uses stdio (`ui-tui/CANONICAL_GATEWAY.md:1-6`).
  - `ui-tui/README.md:10-25` still describes the old stdio spawn. **Inconsistent docs.** Treat stdio as legacy (inference).
- **Stability:**
  - Well documented, and the wire is declared in Pydantic and generated. Unknown param keys are rejected (`tui_gateway/AGENTS.md:46-53`).
  - It is also large, changes fast, carries explicit "undecided" method ownership, and has had breaking changes announced in the docs.
  - It is the richest surface: tokens, reasoning, tool calls and approvals all stream.

### 2.3 ACP (Agent Client Protocol)

- **Launch:** `hermes acp`, `hermes-acp` or `python -m acp_adapter`. JSON-RPC over stdio. Logs go to stderr (`website/docs/user-guide/features/acp.md:96-119`; `pyproject.toml:596`).
  - Dependency: `agent-client-protocol==0.9.0` (`pyproject.toml:450`). It is included in the `all` extra (`pyproject.toml:531-560`), which the source installer installs (`installation.md:79`).
  - The server is started with `acp.run_agent(agent, use_unstable_protocol=True)` (`acp_adapter/entry.py:213`).
- **Methods implemented:**
  - `initialize`, `authenticate`, `new_session`, `load_session`, `resume_session`, `fork_session`, `list_sessions` (paginated, 50 per page) and `cancel` (`acp_adapter/gateway_server.py:140-656`; `acp_adapter/server.py:52`).
  - `prompt`, `set_session_model`, `set_session_mode` and `set_config_option`.
  - Capabilities advertised: `load_session`, image prompts, fork/list/resume (`gateway_server.py:140-149`).
- **Streaming:**
  - Agent message chunks and tool-call events. Tool output is rendered into ACP `Diff`/`ToolCall` content (`programmatic-integration.md:25`; `acp-internals.md:83-126`).
  - Reasoning is forwarded through `step_callback`. `thinking_callback` is `None` (`acp-internals.md:87-91`).
- **Approvals:**
  - Dangerous commands become ACP `request_permission` with allow once, allow for session, allow always and deny (`acp.md:398-428`; `acp-internals.md:105-115`).
  - Timeouts deny. The timeout is `approvals.timeout`, default 300 s (`acp.md:411-413`).
  - Clarify questions use ACP elicitation when the client supports it, and fall back to a permission request otherwise (`gateway_server.py:1162-1175`).
- **Slash commands:** a registered `/command` in prompt text is answered by a gateway verb or refused. It is never sent to the model (`gateway_server.py:25-42`). `/model` sent as a prompt is how ACP switches models (`programmatic-integration.md:290`).
- **Sessions:** persisted to `state.db` and restored across restarts (`acp-internals.md:199`; `acp.md:375-387`).
  - `acp.md:373` still says "in-memory session manager", while `entry.py` says MCP and execution "belong to the daemon; this process is a viewer". **Docs lag code here.**
- **Toolset:** the curated `hermes-acp` toolset excludes messaging and cron management. It can be overridden with `platform_toolsets.acp` (`acp.md:28-60`).
- **Provider auth:** ACP reuses Hermes' configured provider and credentials. It also advertises a terminal-setup auth method (`hermes acp --setup`) (`acp-internals.md:167-176`).
- **Assessment:**
  - A standard protocol with a Python SDK (the same `agent-client-protocol` package). Production users include Zed, VS Code and JetBrains (`programmatic-integration.md:23`).
  - Covers chat, streaming, tools, approvals and session list/resume/fork.
  - Does not cover Hermes-specific management: cron, skills management, memory, profiles, provider setup UI, usage/cost.
  - Non-text prompt blocks are ignored for request text (`acp-internals.md:200`).
  - The "unstable protocol" flag means parts of ACP used here are pre-1.0 upstream. Inference from the flag name. **Unverified** which exact methods depend on it.

### 2.4 OpenAI-compatible HTTP API server

- **Enabling it:**
  - Set `API_SERVER_ENABLED=true` and `API_SERVER_KEY=…` in `~/.hermes/.env`, or `gateway.api_server.*` in `config.yaml`, then run the gateway (`website/docs/user-guide/features/api-server.md:17-40,824-855`).
  - Defaults: `127.0.0.1:8642` (`gateway/platforms/api_server.py:171-184`).
  - It is a platform adapter inside the gateway process (`programmatic-integration.md:15`).
- **Endpoints (`programmatic-integration.md:184-200`; `api-server.md:161-760`):**
  - OpenAI-style: `/v1/chat/completions` (SSE), `/v1/responses`, `/v1/models`, `/v1/capabilities`.
  - Runs: `/v1/runs` (start), `/v1/runs/{id}/events` (SSE), `/approval`, `/clarify`, `/steer`, `/stop`.
  - Sessions REST: `/api/sessions` list/create/get/patch/delete/messages/fork/chat/chat-stream.
  - Discovery: `/v1/skills`, `/v1/toolsets`, `/api/model/options`. Jobs: `/api/jobs`. Health: `/health`.
- **Streaming:** `/v1/runs/{id}/events` carries token deltas, `message.interim`, `tool.started`/`tool.completed` (preview truncated to 500 chars), subagent start/complete, `approval.request`, and terminal `run.*` events (`api-server.md:537-560,672-677`).
- **Approvals:**
  - `POST /v1/runs/{id}/approval` with `{request_id, execution_generation, choice}` (`api-server.md:656-670`).
  - Other API-server sessions fall under `approvals.unattended_mode`, default `deny`. The exception is `/v1/runs` and streaming chat completions, whose client can answer (`security.md:52`).
- **Limits:** no file upload (only inline images), max 100 stored responses (`api-server.md:941-947`).
- **Assessment:**
  - Documented and good for remote or language-neutral use.
  - Requires the user to enable it, set a bearer key, and keep a gateway running.
  - Fewer Hermes-specific events than `hermes-gateway-v1`.

### 2.5 Python library (in-process)

- `from run_agent import AIAgent`, then `agent.chat(...)` or `agent.run_conversation(...)` (`website/docs/guides/python-library.md:34-77`).
- Requires a PM-prepared **source checkout**. There is no supported wheel (`python-library.md:13-25`). Create one `AIAgent` per thread (`python-library.md:220`).
- Product surfaces no longer use it: ACP and the CLI go through the gateway (§2 intro).
- **Not applicable to Budha:** Budha must run in a separate process, and the library needs Hermes' own Python 3.14 environment.

### 2.6 Other surfaces

- **`hermes serve`:** a headless backend server, default port 9119, the same server as `hermes dashboard`. It prints `HERMES_BACKEND_READY port=<port>` on success, and exits 75 with `BACKEND_PORT_IN_USE` when the port is taken (`cli-commands.md:1972-1980`). The desktop app connects to it for **remote** backends (`desktop.md:636-667`). Locally, Desktop uses the gateway directly (`desktop.md:634`).
- **Messaging gateway adapters** (Telegram, Discord and others): not relevant for a local GUI (`README.md:144-160`).
- **Machine-readable helper commands:** `hermes config get <key> --json`, `hermes sessions refresh-prompts --json`, `hermes clipboard image --out PATH --json` (`cli-commands.md:1410,1878-1894`; `programmatic-integration.md:280`).

## 3. Sessions, memory, skills on disk

- **Home directory:**
  - `get_hermes_home()` resolves a context override first, then `HERMES_HOME`, then `~/.hermes` (Linux and macOS) or `%LOCALAPPDATA%/hermes` (Windows) (`website/docs/developer-guide/session-storage.md:16-24`).
  - Named profiles live at `~/.hermes/profiles/<name>/`, each with its own `config.yaml`, `.env`, `SOUL.md`, memories, sessions, skills, cron and `state.db` (`website/docs/user-guide/profiles.md:11`; `session-storage.md:26-32`).
  - Subprocess spawners "should pass `HERMES_HOME` explicitly" (`session-storage.md:41-45`).
- **Sessions:**
  - All sessions (CLI, TUI, gateway, ACP) are stored in SQLite at `$HERMES_HOME/state.db`, in WAL mode, with FTS5 search. Tables: `sessions`, `messages`, `messages_fts` (`website/docs/user-guide/sessions.md:1149-1193`).
  - Schema version is **31**, with a migration history of frequent column additions (`session-storage.md:332-362`).
  - `~/.hermes/sessions/sessions.json` is only a legacy routing mirror. `sessions/saved/*.json` holds `/save` exports. Legacy `*.jsonl` files are no longer read (`sessions.md:1160-1185`).
- **Listing and resuming from outside:**
  - CLI: `hermes sessions list|browse|export|delete|rename|prune|archive|stats` (`cli-commands.md:1868-1894`). Resume with `hermes --resume <id|title|latest>` (`cli-commands.md:28`).
  - Gateway: `session.list` (picker fields, `query`, cursor paging), `session.transcript` and `session.resume` (`programmatic-integration.md:111`).
  - ACP: `list_sessions`, `load_session` and `resume_session` (§2.3).
  - HTTP: `/api/sessions*` (§2.4).
  - Session browsing opens `state.db` read-only, and the gateway owns schema reconciliation (`session-storage.md:313-330`). Inference: a GUI should not write `state.db` directly. Reading it directly would couple Budha to a schema that changes often.
- **Memory:**
  - `$HERMES_HOME/memories/MEMORY.md` (2,200-char limit) and `USER.md` (1,375 chars). Entries are separated by `§`. The agent writes both through the `memory` tool. They are injected as a frozen snapshot at session start (`website/docs/user-guide/features/memory.md:17-20,27-54`; `website/docs/user-guide/which-file-does-what.md:15-18`).
  - `SOUL.md` (persona) is at `$HERMES_HOME/SOUL.md` (`which-file-does-what.md:15`).
- **Skills:**
  - `$HERMES_HOME/skills/<category>/<name>/SKILL.md` is the source of truth. Extra read locations come from `skills.external_dirs` (`website/docs/user-guide/features/skills.md:11,339-353,411-425`).
  - `GET /v1/skills` lists skills over HTTP (`api-server.md:745-760`).
- **Other directories:** `cron/`, `logs/` and `auth.json` (OAuth credentials) (`website/docs/user-guide/configuration.md:19-32`).

## 4. Configuration

- **Files:**
  - `$HERMES_HOME/config.yaml` holds non-secret settings.
  - `$HERMES_HOME/.env` holds API keys and secrets.
  - `$HERMES_HOME/auth.json` holds OAuth credentials such as Nous Portal (`configuration.md:19-32`).
  - Precedence: CLI args, then `config.yaml`, then `.env`, then built-in defaults (`configuration.md:57-68`).
- **Credentials written by Hermes:** `config.yaml` stores only `${VAR}` references, and the value goes to `.env`. Examples: `HERMES_MODEL_API_KEY`, `HERMES_CUSTOM_<HOST>…_API_KEY` (`configuration.md:70-90`). A running gateway re-reads `.env` at the start of every turn (`configuration.md:86`).
- **Supported way to read or write config:**
  - `hermes config get <key> [--json] [--raw]` and `hermes config set <key> <value>`. Dotted keys go to `config.yaml` and `UPPER_SNAKE` names go to `.env`. Values are type-checked, and some names are refused (`HERMES_YOLO_MODE`, `PATH`) (`cli-commands.md:1398-1444`).
  - Over the gateway: `config.get`/`config.set` (served), and `model.options` / `model.save_key` (`programmatic-integration.md:101-103,213-229`).
  - Inference: a GUI should use these rather than edit the YAML itself, because of the validation and secret-routing rules.
- **Providers and models:**
  - `hermes model` is interactive, and `hermes setup` is the full wizard (`README.md:109-119`).
  - The `--provider` flag lists about 40 provider ids (`cli-commands.md:133`).
  - The model picker catalog with pricing and capability hints: `model.options` (RPC) or `GET /api/model/options` (`programmatic-integration.md:207-229`).
  - Mid-session switch: `/model provider:model` via `command.dispatch` (`programmatic-integration.md:284-293`).
- **Environment variables:** the full list is in `website/docs/reference/environment-variables.md`. I did not read it line by line.

## 5. Tool calls and safety

- **Approval modes:** `approvals.mode` is `smart` (default), `manual` or `off` (`security.md:28-65`).
  - `smart` asks an auxiliary LLM. It auto-approves low-risk commands, auto-denies clearly dangerous ones, and escalates the rest to a prompt (`security.md:58`).
  - Timeout is `approvals.timeout`, default 300 s. On timeout the command is denied (fail-closed) (`security.md:196-200`).
  - Headless defaults: `cron_mode`, `single_query_mode` and `unattended_mode` are all `deny` (`security.md:49-52`).
- **Choices:** `once`, `session`, `always` (saved to `command_allowlist` in `config.yaml`) and `deny` (`security.md:264-282,335-345`).
- **Bypass:** `--yolo`, `/yolo` and `HERMES_YOLO_MODE=1` bypass prompts. A hardline blocklist (filesystem wipes, fork bombs, block-device writes) still applies under YOLO (`security.md:67-150`).
- **File writes:** a separate protected-path denylist covers `write_file`/`patch`, with an optional `HERMES_WRITE_SAFE_ROOT` sandbox (`security.md:416-462`).
- **Answering from an external client:**
  - `hermes-gateway-v1`: the `approval` server→client request, or `approval.respond`. Requires `client.capabilities {server_requests:true}` (§2.2).
  - ACP: `request_permission` (§2.3).
  - HTTP: `POST /v1/runs/{id}/approval` (§2.4).
  - CLI one-shot: none, so commands are denied by default (§2.1).
- **Other questions Hermes asks the user** beyond approvals: `clarify` (multiple-choice questions), `sudo` (password), `secret`, vault unlock and MCP setup (`programmatic-integration.md:143-150`). A GUI needs UI for each, or must reply with error `-32601` so the agent fails fast (`programmatic-integration.md:150`).
- **Where tools run:** the default terminal backend is `local`, which runs commands "directly on your machine with no isolation" (`configuration.md:346-353`). Commands run on the **host where the gateway runs**. ACP says "Commands and paths run on the gateway host" (`acp.md:80`). Other backends include Docker, SSH, Modal, Daytona and Vercel (`README.md:29`; `configuration.md:333-345`).

## 6. Running from inside a Flatpak sandbox

Hermes' docs say nothing about Flatpak as a host for a Hermes client. The only Flatpak references in the repo are about *launching other* Flatpak apps (`hermes_platform/resolver/known_dirs.py:97-99`; `hermes_cli/browser_connect.py:155`). Everything in this section is **inference** from the facts cited, unless marked otherwise.

- **Hermes cannot be bundled into Budha's Flatpak in a supported way.**
  - PyPI, pip and uv installs are unsupported (`platform-support.md:62`).
  - The supported Linux install is the source script into `~/.hermes`, with a PM-managed Python 3.14, Node, ripgrep, FFmpeg and Chromium (`installation.md:75-88`).
  - So Budha should treat Hermes as a separately installed **host** program and drive it across the sandbox boundary.
- **Running Hermes inside the sandbox would also be the wrong place for its tools.** With the `local` backend, the agent's shell commands would run inside Budha's sandbox and not on the user's machine (`configuration.md:346-353`).
- **Ways across the boundary:**
  - **Spawning host commands** (`hermes gateway attach --json`, `hermes acp`, `hermes chat …`) needs `flatpak-spawn --host`. That needs the `--talk-name=org.freedesktop.Flatpak` permission, which in effect gives full host command execution. This is Flatpak behaviour, not a Hermes source, so it is **unverified here**.
  - The launcher is at `~/.local/bin/hermes` (`installation.md:116`), which may not be on the `PATH` that `flatpak-spawn --host` sees (**unverified**).
  - **WebSocket:** the ticketed URL is `ws://127.0.0.1:<port>/api/ws` (`gateway-internals.md:607-610`). Reaching host loopback needs `--share=network`.
  - **Control socket without spawning:** the socket is at `$HERMES_HOME/gateway.sock`, or reached through a pointer to `…/hermes-gw-<hash>/control.sock` under `$XDG_RUNTIME_DIR` or the temp dir (`gateway/control_socket.py:60-81`). The client must canonicalize paths and check ownership and modes (`gateway-internals.md:534-556`).
    - From a sandbox, that needs filesystem access to `~/.hermes` and to the runtime-dir location.
    - It is unknown whether the sandbox's view of path, owner and mode checks will match what the gateway expects. **Unverified; needs a test.**
    - The gateway checks the peer uid via `SO_PEERCRED` (`gateway-internals.md:558-562`). Flatpak maps the same uid, so this should pass (**unverified**).
  - **Using `hermes gateway attach --json` through `flatpak-spawn --host`** sidesteps the socket-path issues. The cost is one host spawn per connection and reconnect (the 30 s ticket, `gateway-internals.md:620-627`).
  - **ACP over `flatpak-spawn --host hermes acp`** needs only stdio pipes and no network or filesystem permissions. It still needs the Flatpak talk-name permission.
  - **HTTP API** needs only `--share=network`, but the user must enable it and give Budha the bearer key, which lives in the host's `.env` (§2.4).
- **Gateway lifetime:** a client must cope with the owner being absent, starting, draining or updating (`gateway-internals.md:497-510`). A daemon started by `attach --start` without an installed service exits when idle (`gateway-internals.md:598-601`).
  - `desktop.md:634` says the gateway that Desktop starts via `hermes gateway ensure` "has no idle shutdown". **These two docs read as inconsistent.** Check which applies before relying on it.
- **Hermes' own Chromium and computer-use driver** run on the host under the gateway (`installation.md:80-87`). Budha does not need them in its sandbox.

## 7. Implications for Budha

Recommended integration interfaces, ranked by facts above:

1. **`hermes-gateway-v1` over WebSocket, bootstrapped by `hermes gateway attach --json`.**
   - It is the documented route for "a custom desktop … host" that wants every feature (`programmatic-integration.md:277`).
   - It streams text, reasoning, tool calls and diffs. It carries approvals, clarify and sudo prompts. It supports multi-client attach, reconnect replay, session list/transcript, and the model catalog.
   - Risks:
     - Budha must write its own Python client. The only client is Node.
     - The surface is large (263 methods in OpenRPC, 160 "undecided").
     - Breaking changes have been announced in the docs (§2.2), and stable releases arrive every few days.
     - Flatpak access needs `flatpak-spawn --host` plus network, or socket access to `~/.hermes`.
   - Mitigations from the facts:
     - Generate Python models from `gateway-contract.openrpc.json`.
     - Gate on `runtime.describe` capabilities and `/v1/capabilities`-style feature flags.
     - Treat `-32601` as "not available".
2. **ACP over `hermes acp` stdio.**
   - A standard protocol with an existing Python SDK. It covers chat, streaming, tool calls, permissions, and session list/load/resume/fork. It has the simplest sandbox crossing (stdio).
   - Risks:
     - No Hermes-specific management (cron, skills, memory, profiles, usage).
     - Uses `use_unstable_protocol=True`.
     - Its docs lag its code.
     - It is itself a gateway client, so it adds a process but no isolation.
   - A reasonable fallback, or a first milestone.
3. **HTTP API server.**
   - Documented, with runs, SSE events, approvals and a sessions REST API. Only `--share=network` is needed.
   - Risks: off by default, and the user must enable it and hand over the bearer key. It has fewer events than (1).
   - Best suited if Budha later targets remote Hermes hosts.
4. **`hermes chat -q --format stream-json`.** Stable and simple, but one-shot with approvals denied. Fit only for a throwaway prototype.
5. **Python library.** Not viable: it runs in-process, needs a source checkout, and needs Hermes' Python 3.14 environment.

For config, sessions and skills, use Hermes' interfaces (`hermes config get/set --json`, gateway `config.*`, `session.list`/`session.transcript`, `/v1/skills`) rather than reading `config.yaml`, `.env` or `state.db` directly (§3–4).

**Open questions for the user:**

- Must Budha ship as a Flatpak? If yes, is the `org.freedesktop.Flatpak` talk-name permission (host command execution) acceptable? That decides between ACP stdio and attach plus WebSocket.
- Should Budha require an existing host install of Hermes (via `install.sh`), or offer to install it? Desktop's model is to "install the Hermes Agent runtime into `HERMES_HOME`" (`desktop.md:634`).
- How many Hermes-specific features are in scope (cron, skills, memory, profiles, cost/usage, subagents)? That decides whether ACP is enough.
- Which Hermes versions will Budha support, given 0.x SemVer, releases every few days, and announced wire changes? Pin to stable only?
- Remote Hermes hosts (`hermes serve` / HTTP API) in scope, or local only?
- Needs a hands-on test (not done here): whether the control-socket path, ownership and permission checks pass from inside a Flatpak sandbox, and whether `attach --start` from `flatpak-spawn --host` leaves a gateway running that a sandboxed WebSocket client can dial.
