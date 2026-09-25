# Hermes agent capability audit (issue #4)

## Summary

**The question.** Issue #4 asks whether [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) can serve as the engine for the *Personal Work Footprint* archive — a participant-only capture of the user's chats, meeting recordings, and email, held in a **local encrypted single-user store** (see the Destination statement in issue #1 and the project memory note). This memo audits Hermes against that requirement list: connectors, search, and local run are **must-have**; transcription and extensibility are **high value**; the local encrypted single-user store is the Destination-level requirement.

**The answer.** Hermes **meets the local-run, transcription, and extensibility requirements outright, and is close on connectors and search only in the sense that it can *operate* those services — not *archive* them**. It fails the Destination requirement completely: its canonical store is plaintext SQLite at `~/.hermes/state.db`, and the only encryption it ships is a Fernet vault for browser autofill secrets, not a general encrypted archive. **Verdict: use Hermes as the agent/runtime and capture/transcription layer; do not fork it; build the encrypted archive store, the participant-only capture filter, and the corpus index as separate components that Hermes feeds.** The two things that could still sink this plan are (a) bulk ingest into an encrypted store and (b) Malay/English code-switched transcription accuracy; both are testable spikes, not architectural blockers.

---

## Method and scope

This audit is a **static, read-only review** of a local clone of `main` at commit `7b761da2de4979e424510ca7022bf9527aa65b68` (clone inspected 2026-09-25). No Hermes instance was installed or executed, and no `git` or `gh` command was run, per the task constraints. Repository-level statistics were read from the public GitHub REST API on 2026-09-25 rather than from the clone. Every substantive claim below cites the file, document, or API response that owns it. Where I could not verify something, I say so explicitly in **Open questions / residual risk** instead of asserting it.

The clone contained **16,104 files excluding `.git`** (`find . -path ./.git -prune -o -type f -print | wc -l`). The earlier read-only pass estimated ~17.3k; the difference is almost certainly counting method (inclusion of `.git` or of generated directories), and nothing in the audit depends on the exact figure.

---

## What Hermes is

Hermes Agent is a **general-purpose personal agent framework**, not an archive or ingest product. Its README opens: "**The self-improving AI agent built by Nous Research.** It's the only agent with a built-in learning loop — it creates skills from experience, improves them during use, nudges itself to persist knowledge, searches its own past conversations, and builds a deepening model of who you are across sessions" (`README.md`). The packaging metadata agrees: `pyproject.toml` names the project `hermes-agent` and describes it as "The self-improving AI agent — creates skills from experience, improves them during use, and runs anywhere." The repository's own GitHub description is the marketing line "The agent that grows with you" (GitHub REST API, `/repos/NousResearch/hermes-agent`, fetched 2026-09-25). Nothing in the README, the docs tree, or the top-level module layout describes archival ETL, retention policy, or evidence-grade storage — the concerns the Footprint project is built around.

**Project scale and maturity** (GitHub REST API, `/repos/NousResearch/hermes-agent`, and `/releases`, fetched 2026-09-25):

| Signal | Value |
|---|---|
| License | MIT (`LICENSE`; `license.spdx_id = "MIT"`) |
| Created | 2025-07-22T22:22:28Z |
| Last push | 2026-09-25T06:59:35Z |
| Stars / forks | 248,777 / 52,681 |
| Watchers (subscribers) | 961 |
| Open issues (search API, `type:issue state:open`) | 13,743 |
| Open PRs (search API, `type:pr state:open`) | 29,191 |
| Contributors | ~3,963 (contributors API `Link` last page = 3963, `per_page=1&anon=true`) |
| Default branch | `main` |
| Primary language | Python (91,098,245 bytes); TypeScript 32,427,761 bytes |

Releases are roughly **weekly**: `v2026.9.24` ("Hermes Agent v0.21.5"), `v2026.9.21` (v0.21.4), `v2026.9.14` (v0.21.3), `v2026.9.11` (v0.21.2), `v2026.9.7` (v0.21.1), `v2026.8.31` (v0.21.0) — all fetched from the releases API. The 29,191 open pull requests are the maturity caveat: this is a very fast-moving codebase with a large contribution backlog, which raises the cost of any fork and argues for staying on pinned upstream releases.

---

## Requirement-by-requirement gap map

| Requirement | Priority | Verdict | One-line reason |
|---|---|---|---|
| Connectors (chat / email / meetings) | **Must** | **Partial** | Live service operation and interactive gateways; no bulk archival ETL, no Zoom |
| Search / indexing | **Must** | **Partial** | FTS5 over Hermes's *own* sessions and memory, not an arbitrary personal corpus |
| Local run | **Must** | **Met** | Managed local llama.cpp runtime; documented offline operation |
| Transcription | High | **Met** (accuracy caveat) | Pluggable local/cloud STT, first-class; single language hint |
| Extensibility | High | **Met** | Plugins, skills, MCP, hooks, cron, catalog |
| Local encrypted single-user store | **Destination** | **Not met** | Canonical store is plaintext SQLite; only a credential vault is encrypted |

### Connectors — PARTIAL

**What exists.** Connector capability is MCP-centric. The user guide states: "MCP lets Hermes Agent connect to external tool servers so the agent can use tools that live outside Hermes itself" and supports "Local stdio servers and remote HTTP MCP servers in the same config" (`website/docs/user-guide/features/mcp.md`). The connector implementation lives under `tools/connectors/` — `gateway/bridge.py`, `gateway/client.py`, `mcp.py`, `catalog.py`, `search.py`, `managed.py`, `operation.py`, `contract.py` (directory listing, commit `7b761da`). The catalog machinery is described in `tools/connectors/catalog.py:1-8` as "`manage_catalog` install targets: catalog plugins and hub skills, installed through the card," resolving ids "against the plugin catalog (the Plugins tab's resolver) or the skills hub." Crucially, `tools/connectors/search.py` frames connectors as **hosted** tools whose failure mode is authentication: it returns "`Hosted connector tools could not be … right now`" and, on `SIGN_IN_EXPIRED`, "The user must sign in to Nous again." In other words, the connector path is designed to drive live third-party services through an authenticated session — it is a **tool-operating layer, not an ingestion pipeline**.

There are **65 entries** in `optional-mcps/` (directory listing), a catalogue of MCP server configurations (airtable, algolia, asana, atlassian, dropbox, figma, gitlab, linear, notion, sentry, stripe, supabase, todoist, vercel, …).

**Messaging gateways.** The legacy/direct adapters sit in `gateway/platforms/`: `signal.py`, `whatsapp_cloud.py`, `bluebubbles.py`, `weixin.py`, `yuanbao.py`, `msgraph_webhook.py`, `webhook.py`, `api_server.py`, and `qqbot/` (directory listing). Most platform integrations now ship as bundled plugins under `plugins/platforms/`: `telegram`, `discord`, `slack`, `whatsapp`, `email`, `sms`, `teams`, `dingtalk`, `feishu`, `google_chat`, `irc`, `line`, `matrix`, `mattermost`, `ntfy`, `photon`, `raft`, `simplex`, `wecom`, `a2a`, `buzz`, `homeassistant` (directory listing); SECURITY.md §2.6 confirms "Most messaging integrations ship as bundled plugins under `plugins/platforms/<name>/` (Telegram, Discord, Slack, email, SMS, etc.)" with "discovery and deferred loading via `gateway/platform_registry.py`."

**Email.** The gateway adapter is explicitly an interactive inbox, not an archiver. `plugins/platforms/email/adapter.py:1-2` documents: "users talk to Hermes by sending email; IMAP (polled) receives, SMTP sends." The same file defines `_NOREPLY_PATTERNS` and `_AUTOMATED_HEADERS` with the comment "Automated senders (address substrings / bulk-mail headers) are silently ignored" (`plugins/platforms/email/adapter.py:37-41`) — i.e. the adapter deliberately **drops bulk mail**. Mailbox *access* does exist through first-party skills: `skills/email/himalaya/SKILL.md` (community-authored, MIT) wraps the Himalaya CLI for IMAP/SMTP, and `skills/productivity/google-workspace/SKILL.md` (authored by Nous Research) covers "Gmail, Calendar, Drive, Contacts, Sheets, and Docs — through Hermes-managed OAuth and a thin CLI wrapper," with a `references/gmail-search-syntax.md`. This **refines the seed finding**: it is not true that Hermes has no first-party Gmail or IMAP access — it has both — but neither is a bulk archival pipeline. Both are per-operation, interactive, and index nothing on their own.

**Meetings.** Microsoft Teams is covered by `plugins/teams_pipeline/`, whose `plugin.yaml` describes "a Microsoft Teams meeting pipeline plugin with durable runtime state and operator CLI flows for **Graph-backed transcript-first meeting summaries**," and whose `meetings.py:1` confirms "Graph-backed Teams meeting helpers," matching Graph `/onlineMeetings`, `/transcripts`, and `/recordings` routes (`plugins/teams_pipeline/meetings.py:17-20`). Google Meet is covered by `plugins/google_meet/`, whose README states: "Let the hermes agent join a Google Meet call, transcribe it, optionally speak in it, and do the followup work afterwards," with a Playwright bot that "joins Meet, scrapes captions to transcript file" (`plugins/google_meet/README.md`). **There is no Zoom meeting integration.** A repository-wide filename search for `*zoom*` returns only desktop UI image-zoom code (`apps/desktop/src/store/zoom.ts`, `apps/desktop/src/components/ui/zoomable.tsx`, `apps/desktop/electron/zoom.ts`), and a grep for "zoom" in non-website Python/YAML/JSON hits only vision-tool image zooming (`tools/vision_tools.py`, `tools/vision_tools_image_prep.py`) and computer-use test fixtures.

**Why this is only partial for Footprint.** Every connector above is built to *drive* a live service (send a message, list recent mail, join a call), or to *receive* messages addressed to the agent. None of them walks a historical mailbox, channel, or meeting back-catalogue and writes it to a durable, queryable archive. The Teams pipeline is the closest to archival (it fetches Graph transcripts/recordings), but it is scoped to Teams and to "transcript-first meeting summaries." So the gap is not "Hermes cannot reach these services" — it clearly can — but "Hermes does not ingest a personal corpus into a store."

### Search / indexing — PARTIAL

Hermes search is **session search**. Sessions are persisted to SQLite with FTS5: `website/docs/user-guide/sessions.md` describes "SQLite database (`~/.hermes/state.db`) — structured session metadata with FTS5 full-text search, plus full message history," and the storage-locations table names `~/.hermes/state.db` as "All session metadata + messages with FTS5" and the "canonical store for all session messages." `hermes_state_fts.py:1-2` builds "the CJK-bigram (cjk_unicode61) index DDL and tokenizer loader" for the `messages` table, and the FTS source view is defined over `messages` (`hermes_state_fts.py`, `FTS_CJK_TABLE_SQL`, selecting `id, role, content, tool_name, tool_calls FROM messages WHERE role <> 'tool'`). `hermes_state_search.py:1-3` is explicit: "Full-text / trigram / CJK message search and FTS maintenance for SessionDB." A native tokenizer, `native/fts5_cjk/` ("unicode61 + CJK character bigrams (Lucene CJKAnalyzer semantics)"), extends this to 1–2-character CJK terms.

Long-term memory is a separate, pluggable layer: `agent/memory_manager.py` plus `plugins/memory/{honcho, mem0, supermemory, byterover, holographic, retaindb, openviking}` (directory listing). This is memory *about the user and the agent's own history*, not an index of the user's documents, mail, or recordings.

**Why this is only partial for Footprint.** The index covers Hermes's own conversations and memory. I found no component that indexes an arbitrary personal corpus (Gmail, Drive, meeting recordings, exported chat histories). To search the Footprint archive, the project would need its own index — or would need to funnel every artifact into Hermes sessions first, which is the opposite of the "archive as system of record" model. One useful bridge exists: `hermes sessions export` supports bulk selection and formats — "is one surface for every export format, selected with `--format`," with filters (`--older-than`, `--source`, `--chat-id`, …) and `--redact` (`website/docs/user-guide/sessions.md:355-410`). That makes Hermes-to-archive export feasible; it does not make Hermes the archive index.

### Local run — MET

Local inference is a first-class, documented path. `website/docs/user-guide/local-models.md` states: "Hermes can run open models entirely on your own machine. It downloads and manages the inference engine (llama.cpp), picks the right build of each model for your hardware, and handles memory so you never configure context sizes, GPU layers, or quantization," and — directly relevant to a PDPA-bound archive — "**Nothing leaves your computer: no account, no API key, and no network access after a model is downloaded.**" The runtime is implemented in `hermes_cli/local_runtime/` (`binaries.py`, `catalog.py`, `catalog.json`, `gguf.py`, `hardware.py`, `supervisor.py`, `presets.py`, …). Beyond the managed llama.cpp path, the provider system supports 38 provider plugins (`plugins/model-providers/`, 39 entries including `README.md`) and a Custom Endpoint flow for local Ollama at `http://localhost:11434/v1` with no key (`website/docs/integrations/providers.md:429-430`). Model choice is not locked: "Use any model you want — Nous Portal, OpenRouter, OpenAI, your own endpoint, and many others" (`README.md`).

### Transcription — MET, with an accuracy caveat

Speech-to-text is first-class and pluggable. `tools/transcription_local.py:1-6` implements "Local STT backends. faster-whisper loading (CUDA->CPU fallback, Apple Silicon pinning), the anti-hallucination transcribe kwargs and segment gate, and the local whisper CLI (`local_command`) provider." `tools/transcription_cloud.py` provides the OpenAI/Groq cloud path, and `agent/transcription_provider.py` is the provider protocol; its docstring notes the language parameter is "an optional BCP-47 hint" (`agent/transcription_provider.py:33`). The user guide documents local Faster-Whisper with "**zero API keys**" and a ~150 MB `base` model (`website/docs/user-guide/features/voice-mode.md:96-107`), plus a hallucination filter of "26 known hallucination phrases across multiple languages" (line 238). It also documents a real platform limit: "Local Faster-Whisper is excluded on native Windows ARM64 and Intel macOS" (line 60).

**The caveat is code-switching.** The provider takes a *single* language hint, and the configuration default is English: `tools/transcription_common.py:16-17` sets `DEFAULT_LOCAL_MODEL = "base"` and `DEFAULT_LOCAL_STT_LANGUAGE = "en"`; the config reference shows `language: ""` as "optional ISO-639-1 hint; blank = use HERMES_LOCAL_STT_LANGUAGE if set, else auto-detect" (`website/docs/user-guide/features/voice-mode.md:475`). I found **no code-switching support**: a grep for `code.switch`/`codeswitch` across Python and Markdown returned no relevant implementation (only unrelated "op-code switch" and "bytecode switch" matches). For the Footprint use case — Malay/English code-switched speech in real meetings — a single-language hint plus a small default model is a genuine accuracy risk. I cannot quantify that risk from source; it needs the spike in §"De-risking spikes".

### Extensibility — MET

Extensibility is broad and multi-layered, which is why "use, don't fork" is viable. `plugins/` contains the loader (`plugin_loader.py`) plus first-party plugins (`browser`, `memory`, `platforms`, `model-providers`, `teams_pipeline`, `google_meet`, `kanban`, `image_gen`, `video_gen`, `cron_providers`, `context_engine`, `dashboard_auth`, `observability`, …). Skills come in a bundled `skills/` tree and a separate `optional-skills/` tree (23 skill directories), and the README notes compatibility "with the agentskills.io open standard." MCP is pervasive — 637 Python files reference it (grep count) — with `mcp_serve.py` exposing Hermes itself as a server and 65 `optional-mcps/` catalog entries. `plugin-catalog/` holds 298 YAML files (299 entries) including `microsoft365.yaml`. Hooks are documented in `website/docs/user-guide/features/hooks.md` and backed by `gateway/builtin_hooks/`; scheduling is implemented in `cron/` (`scheduler.py`, `job_definition.py`, `delivery_queue.py`, …).

The counterweight is the trust model: SECURITY.md §2.5 states "Plugins load into the agent process and run with full agent privileges: they can read the same credentials, call the same tools, register the same hooks, and import the same modules as anything shipped in-tree. The boundary for third-party plugins is operator review before install." Extensibility is real, but each added plugin/skill widens the attack surface inside a process that SECURITY.md says has no internal containment boundary.

### Local encrypted single-user store — NOT MET

This is the decisive gap. The canonical store is **plaintext SQLite in WAL mode**:

- `website/docs/user-guide/sessions.md`: "SQLite database (`~/.hermes/state.db`) — structured session metadata with FTS5 full-text search, plus full message history," storing "Session ID, source platform, user ID … Full message history (role, content, tool calls, tool results) … Timestamps." The storage table names `~/.hermes/state.db` as the canonical store, and notes "The SQLite database uses WAL mode."
- No at-rest encryption exists for it. A repository-wide search for `sqlcipher` returned **0 hits**; grep for `encrypt`/`cipher` across `hermes_state*.py`, `agent/session_persistence.py`, and `gateway/session_persistence.py` returned **nothing**; there is no `PRAGMA key` usage anywhere. Connections are ordinary `sqlite3.connect` calls (`hermes_state_repair.py`, `hermes_state_wal.py`, and others).
- The only encryption found is a **credential vault for browser autofill**, not a data store. `agent/vault_store.py:1-9` documents "Local encrypted vault for browser autofill secrets … the payload is encrypted at rest with a locally generated Fernet key," storing "Three item kinds: `login` (password-only secret), `payment` (card fields) and `address`," with key and vault files "created 0600 under `<HERMES_HOME>/vault/`." The user guide describes it as "Passwords & Logins … Passwords are encrypted on this machine and injected straight into the page; the model never sees them" (`website/docs/user-guide/features/credential-vault.md`). This is a secrets vault, not an archive.
- Other persisted state is likewise plaintext: the Teams pipeline writes `teams_pipeline_store.json` under `HERMES_HOME` (`plugins/teams_pipeline/store.py:17-19`), and session exports/snapshots land as JSON/JSONL under `~/.hermes/sessions/` (`website/docs/user-guide/sessions.md`).

"Single-user" is satisfied — SECURITY.md §2.2: "Hermes Agent is a single-tenant personal agent" — but single-tenant is not the same as encrypted, and the Footprint Destination requires **local encrypted**. Hermes cannot be the archive store as shipped. It can sit *in front of* one, capturing and transcribing and then handing artifacts to an encrypted store the project owns.

---

## Security and trust model (relevant to "participant-only" and PDPA)

The trust model matters twice for this project: once because the archive will hold sensitive personal communications, and once because Hermes ingests untrusted content (inbound email, group chats) into an agent with shell access.

- **The only containment boundary is the OS.** SECURITY.md §2.2: "**The only security boundary against an adversarial LLM is the operating system.** Nothing inside the agent process constitutes containment — not the approval gate, not output redaction, not any pattern scanner, not any tool allowlist. Any in-process component that screens LLM output is a heuristic operating on an attacker-influenced string." §2.4 lists the approval gate, output redaction, and Skills Guard as explicitly "not boundaries." §2.2 also warns that operators "running the default local backend with untrusted input surfaces … are operating outside the supported security posture" — which is exactly the Footprint situation (inbound mail, group chats).
- **External surfaces require operator-configured authorization.** SECURITY.md §2.6: "Authorization is required at every surface that crosses a trust boundary. For messaging and network HTTP surfaces, the boundary is the network: authorization means an operator-configured caller allowlist." The user-guide overview lists eight layers, including "**Cross-session isolation** — sessions cannot access each other's data or state" (`website/docs/user-guide/security.md:11-22`).
- **Participant-only filtering does not exist as a feature.** I found no capture policy keyed on "the user is a participant." What exists is *access* control (platform allowlists, DM pairing — `website/docs/user-guide/security.md:409-508`) and *session scoping* (per-user group sessions, e.g. `group_sessions_per_user` in `website/docs/user-guide/sessions.md:871-874`). Those govern who may talk to the agent and how conversations are partitioned; they do not filter *which of the user's own communications get archived*. Participant attribution is partially available at the session-key level, but the "only capture where I am a participant/recipient/attendee" rule must be **built**, not configured.
- **Telemetry surfaces exist and must be audited.** `agent/monitoring/otlp_exporter.py` is present; NeMo Relay observability exporters are opt-in via the `HERMES_NEMO_RELAY_PLUGINS_TOML` environment variable and write under `~/.hermes/telemetry/` (`website/docs/user-guide/features/built-in-plugins.md:218-265`); the computer-use driver "ships with anonymous usage telemetry (PostHog) enabled by default," though Hermes sets `cua_telemetry: false` by default (`website/docs/user-guide/features/computer-use.md:491-508`). For a PDPA-bound archive, egress must be verified off, not assumed.

Legal interpretation (data-subject rights, retention, cross-border transfer) is out of scope here and belongs to issue #3; this section only records what the software does.

---

## Verdict: use, don't fork; build the store and the ETL

**Use Hermes as the agent/runtime and capture/transcription layer.** It satisfies local run, transcription, and extensibility, and it can reach the chat, email, and meeting surfaces the project cares about. Its MIT licence and 248k-star ecosystem reduce integration risk, and `hermes sessions export` gives a clean hand-off path to an external archive.

**Do not fork.** The repository is ~16k files, ~1 GB of git object size (`size: 1071321` KB), a Python/TypeScript monorepo with a 29k-deep open-PR backlog and weekly releases. The one thing Hermes is missing for this project — an encrypted archive store — is **additive**, not a core change; forking would buy a maintenance burden and no capability. Pin a release tag and stay on upstream.

**Build three things alongside Hermes, not inside it:**

1. **The encrypted archive store** (SQLite+SQLCipher, or an age-encrypted blob store, or an encrypted filesystem) holding the participant-only corpus. Hermes's plaintext `state.db` is a transient working cache, excluded from the evidence store.
2. **The participant-only capture filter** — a policy layer that decides which chats/emails/meetings are in scope before anything is persisted.
3. **The corpus index and ETL** that pulls from source APIs (Graph, Gmail, IMAP, chat exports), transcribes recordings, and writes into the encrypted store with provenance.

Hermes then consumes that store as a tool/data source (via MCP or a plugin), rather than owning it.

---

## Acceptance bar for "good enough"

Before Hermes is accepted as the capture/agent layer, all of the following should hold:

1. **Offline end-to-end.** Capture → transcribe → summarise runs with a local model and local STT, and a network capture shows **no content egress**. Ties to `local-models.md`'s "no network access after a model is downloaded" claim — verified, not assumed.
2. **Transcription accuracy on the real language mix.** On a held-out set of ≥30 real Malay/English code-switched clips from the user's own meetings, the pipeline meets a stated threshold — e.g. ≤15% WER overall and ≥90% correct capture of names, decisions, and amounts. If it does not, transcription moves out of Hermes to a dedicated STT step and Hermes consumes the text.
3. **Ingest completeness with zero over-capture.** ≥95% of in-scope participant communications captured, reconciled by source count vs archive count, and **zero** archived items where the user is not a participant/recipient/attendee.
4. **Encryption at rest with key separation.** The archive is encrypted at rest, the key is not stored beside the ciphertext, and Hermes's own plaintext state is documented as out-of-scope and purgeable. A retention/deletion path exists and is exercised.
5. **Egress controlled and verified.** Telemetry and observability exporters disabled and confirmed; any cloud provider used for any step is an explicit, documented, per-source decision consistent with the PDPA memo (issue #3).
6. **Portability.** The archive exports to an open format and can be re-indexed without Hermes.
7. **Upgrade safety.** Hermes pinned to a release tag; upgrades tested against the ingest pipeline; **no local patches to Hermes core**.

---

## De-risking spikes

**Spike A — bulk ingest into an encrypted store.** Hermes has no bulk ETL (see Connectors). Prototype the path: pull a real mailbox/channel/meeting back-catalogue using the Graph/Gmail/IMAP APIs or the relevant skills, write it to an encrypted store **outside** Hermes, then index it separately; evaluate `hermes sessions export` as an intermediate bridge. Success = completeness reconciliation passes (acceptance bar #3) and nothing lands in `state.db` as the system of record.

**Spike B — Malay/English code-switched transcription.** Measure WER and entity accuracy on real code-switched audio across: `base` (default) vs `large-v3`; `language='ms'` vs `'en'` vs auto-detect; `tools/transcription_local.py` (faster-whisper) vs a cloud provider. Success = acceptance bar #2, or a clear decision that transcription lives outside Hermes.

---

## Open questions / residual risk

- **Nothing was executed.** This is a static review of commit `7b761da`. Accuracy, latency, memory use, and actual network egress are unverified empirically; the acceptance bar exists precisely to close that gap.
- **Connector residency.** Hosted connectors require a Nous sign-in (`tools/connectors/search.py`). Whether they can be used without routing through Nous infrastructure — and what that means for Malaysian data residency and the "commonly used solutions in Malaysia" constraint — is unresolved and may interact with issue #2.
- **Third-party licensing.** Hermes itself is MIT, but skills, plugins, and `optional-mcps` entries carry their own licences (e.g. the `himalaya` skill is community-authored, MIT). Any component the project bundles needs its own licence review; this audit did not enumerate them.
- **Media/recording retention is unmapped.** I found no durable, documented, encrypted store for downloaded media or recordings — `gateway/platforms/media_cache.py` is inbound media dispatch and the Teams pipeline persists JSON metadata. This is absence of evidence, not proof of absence; a targeted follow-up should trace where `plugins/google_meet` and `plugins/teams_pipeline` write recordings and transcripts.
- **"No SQLCipher" is a static finding.** Code search returns zero hits, but a plugin could add an encrypted SQLite dependency at runtime; not verified dynamically.
- **Churn.** With weekly releases and ~29k open PRs, these findings decay. Re-verify file paths and the storage claim before implementation.
- **Participant semantics.** The archive's definition of "participant/recipient/attendee" (and how to derive it from each source's metadata) is a project design question, not something Hermes answers; it should be settled in issue #5/#6.
- **PDPA specifics** (retention periods, data-subject access, cross-border transfer) are deliberately deferred to issue #3.

---

## Sources

**Primary — repository at commit `7b761da2de4979e424510ca7022bf9527aa65b68` (local clone, inspected 2026-09-25):**

- `README.md` — identity ("The self-improving AI agent built by Nous Research"), capability table, agentskills.io, terminal backends.
- `pyproject.toml` — project name, description, `license = "MIT"`.
- `LICENSE` — MIT License, "Copyright (c) 2025 Nous Research".
- `SECURITY.md` §2.1–§2.6 — single-tenant trust model, "The only security boundary … is the operating system," in-process heuristics, plugin trust model, external surfaces.
- `agent/vault_store.py:1-9` — Fernet-encrypted browser-autofill vault (login/payment/address), 0600 under `<HERMES_HOME>/vault/`.
- `agent/session_persistence.py`, `gateway/session_persistence.py` — session persistence (no encryption).
- `hermes_state_fts.py:1-2` and `FTS_CJK_TABLE_SQL` — FTS5 index over `messages`.
- `hermes_state_search.py:1-3` — "Full-text / trigram / CJK message search … for SessionDB."
- `hermes_state_dbfile.py:1-6` — `state.db` file-level helpers.
- `hermes_constants.py:239` — `_HERMES_HOME_MARKERS = ("config.yaml", ".env", "state.db")`.
- `tools/transcription_local.py:1-6`, `tools/transcription_cloud.py`, `tools/transcription_common.py:16-17`, `agent/transcription_provider.py:33`.
- `tools/connectors/catalog.py:1-8`, `tools/connectors/search.py` (hosted connectors; Nous sign-in).
- `plugins/platforms/email/adapter.py:1-2, 37-41` — IMAP/SMTP gateway; automated senders silently ignored.
- `plugins/teams_pipeline/plugin.yaml`, `plugins/teams_pipeline/meetings.py:1-20`, `plugins/teams_pipeline/store.py:17-19`.
- `plugins/google_meet/README.md` — Playwright Meet bot, caption scraping.
- `skills/email/himalaya/SKILL.md`, `skills/productivity/google-workspace/SKILL.md`.
- Directory listings: `tools/connectors/`, `optional-mcps/` (65), `gateway/platforms/`, `plugins/platforms/`, `plugins/memory/` (7), `plugins/model-providers/` (38 + README), `hermes_cli/local_runtime/`, `plugin-catalog/` (298 YAML), `native/fts5_cjk/`, `cron/`.
- `website/docs/user-guide/sessions.md` — session storage locations, schema, WAL mode, `hermes sessions export`.
- `website/docs/user-guide/local-models.md` — "no network access after a model is downloaded."
- `website/docs/user-guide/features/voice-mode.md` — local Faster-Whisper, zero API keys, ~150 MB `base`, hallucination filter, Windows/macOS limits, `language` config.
- `website/docs/user-guide/features/mcp.md` — MCP local stdio + remote HTTP.
- `website/docs/user-guide/features/credential-vault.md` — "Passwords are encrypted on this machine."
- `website/docs/user-guide/features/hooks.md`, `website/docs/user-guide/features/built-in-plugins.md:218-265`, `website/docs/user-guide/features/computer-use.md:491-508`.
- `website/docs/user-guide/security.md` — eight-layer model, allowlists, DM pairing, protected paths.
- `website/docs/integrations/providers.md:429-430` — local Ollama via Custom Endpoint.
- `website/docs/developer-guide/relay-connector-contract.md` — gateway ↔ connector (Node/TS) contract, EXPERIMENTAL.

**Primary — GitHub REST API, fetched 2026-09-25:**

- `https://api.github.com/repos/NousResearch/hermes-agent` — licence, timestamps, stars, forks, subscribers, size, default branch, language.
- `https://api.github.com/repos/NousResearch/hermes-agent/languages` — Python/TypeScript byte counts.
- `https://api.github.com/repos/NousResearch/hermes-agent/releases?per_page=6` — release cadence and tags.
- `https://api.github.com/repos/NousResearch/hermes-agent/contributors?per_page=1&anon=true` — `Link` last page = 3963.
- `https://api.github.com/search/issues?q=repo:NousResearch/hermes-agent+type:issue+state:open` — 13,743 open issues.
- `https://api.github.com/search/issues?q=repo:NousResearch/hermes-agent+type:pr+state:open` — 29,191 open PRs.

**Not retrievable / not verified:** no primary source was consulted for Malaysian data-residency treatment of hosted Nous connectors, and no runtime or accuracy measurement was performed. Those are flagged in **Open questions / residual risk** and the spikes above.
