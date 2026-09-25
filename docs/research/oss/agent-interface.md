# Agent-readable interfaces over a local corpus: OSS survey

**Scope:** footprint, project "footprint" — a participant-only, single-user, application-level-encrypted personal work archive that must expose a **read-only** surface to an external agent (Hermes).
**Question:** does existing OSS already expose a local personal corpus to an external agent, and can footprint adopt any of it instead of reinventing it?
**Method:** primary sources only — repo READMEs, LICENSE files, GitHub license metadata (`api.github.com/repos/...` → `license.spdx_id`), the MCP specification, and vendor license pages. Licences were read from the LICENSE file or the repo's own license metadata, not from blog posts.
**Date of survey:** 2026-09-25.

---

## 1. Short answer

**No.** There is no OSS project that already does what footprint needs, and the gap is not a small one.

The MCP ecosystem is real, active and now institutionally stable (Anthropic donated MCP to the Linux Foundation's Agentic AI Foundation on 2025-12-09), and it has several excellent *components* footprint should reuse. But every "personal knowledge base over MCP" project in this survey is a **memory store that owns and writes its own data**. None of them is a **read-only query surface over a corpus that something else captured**, and none of them implements **application-level encryption at rest**. Those two properties are the core of footprint, so the answer to "adopt as-is" is no — but the answer to "adopt the design" is yes, in several specific places.

The four questions asked, answered:

1. **Is there an MCP server that reads a local SQLite/vector corpus and exposes semantic search *plus* timeline retrieval?** Not as a read-only surface. The closest are `doobidoo/mcp-memory-service` (Apache-2.0; SQLite + `sqlite-vec` + hybrid semantic search + time-based filters, but write-oriented and unencrypted) and `basicmachines-co/basic-memory` (AGPL-3.0; Markdown corpus + full-text/vector/graph index with read-only-annotated tools, but a write-oriented store). Both *own* the data; neither is a reader over an existing corpus. No candidate implements an append-only evidence model with verifiable timestamps.
2. **What is MCP's authorization model, and can it keep a consumer read-only?** MCP has **no protocol-level read-only enforcement**. Authorization is *optional* and specified for HTTP transports only; **stdio servers are explicitly told *not* to implement it and to take credentials from the environment**. Tool annotations such as `readOnlyHint` are, per the spec, **untrusted hints** that a malicious or buggy server can lie about. The boundary has to be built by the server: expose only read tools, run as a separate process, and if you use HTTP then bind `127.0.0.1`, validate `Origin`, and authenticate. This is not hypothetical: parallel surveys found MCP servers that ship an explicit read-only/destructive vocabulary and *still* expose `shell`, `crawl` and `create-*` tools to the agent (§3.10). The one genuinely read-shaped surface found (Onyx, §3.10) shows the limit of the good case: it authenticates each consumer with a bearer token and then enforces authorisation in its **data layer**, which footprint cannot do — it is single-user with no per-document ACLs to enforce against — and its scope is a single coarse `mcp:use` with no read/write split. So even the best example in the ecosystem hands footprint no read-only guarantee it can reuse; it hands it a *shape* to copy.
3. **Encryption at rest?** Essentially none, anywhere in the MCP server ecosystem. The only app-level E2EE precedent found in the personal-knowledge space is Joplin's note-level E2EE, and even that does not encrypt the whole local database. Footprint must build this itself — `SQLCipher` Community Edition (BSD-style, attribution required) or `SQLite3MultipleCiphers` (MIT) are the realistic building blocks.
4. **Is MCP stable enough to build a product surface on, and what transport for a desktop app?** Yes, with a pinned revision. Dated spec revisions run `2024-11-05 → 2025-03-26 → 2025-06-18 → 2025-11-25 → 2026-07-28` (latest), governance is now a Linux Foundation directed fund, there is an official registry (in **preview** since 2025-09-08), and there are ten official SDKs. For a local desktop app the answer is **stdio**: the client launches the server as a subprocess, there is no network surface at all, and the spec says clients SHOULD support stdio whenever possible.

---

## 2. Candidate table

Licence column = the exact licence as verified from the LICENSE file or GitHub's `license.spdx_id`. "Last activity" = `pushed_at` from the GitHub API on 2026-09-25.

| Candidate | Licence | What it covers | What it does NOT cover | Last activity | Verdict |
|---|---|---|---|---|---|
| **MCP spec + official SDKs** (`modelcontextprotocol/modelcontextprotocol`, `.../servers`) | Reference servers: **MIT → Apache-2.0 transition** (mixed); spec-repo licence not verified | The protocol, transports, auth framework, ten SDKs | No storage, no encryption, no corpus semantics | servers: 2026-09-22, 90.6k★ | **ADOPT** (protocol) / **ADOPT-AS-PATTERN** (reference servers) |
| **Official filesystem server** (`servers/src/filesystem`) | MIT/Apache-2.0 (same repo) | "Secure file operations with configurable access controls" — explicit allow-listed directories | Not a corpus; no search, no DB, no timeline | 2026-09-22 | **ADOPT-AS-PATTERN** — the allow-list + path-containment design |
| **Official memory server** (`servers/src/memory`) | MIT/Apache-2.0 | Knowledge graph (entities / relations / observations); tools `create_entities`, `create_relations`, `add_observations`, `delete_*`, `read_graph`, `search_nodes`, `open_nodes` | No semantic search (keyword match), no timeline, read-write, no encryption | 2026-09-22 | **ADOPT-AS-PATTERN** (graph shape) / **REJECT** as a store |
| **Official sqlite server** (`servers-archived/src/sqlite`) | **MIT** | SQLite query/schema tools | **Archived**; no vector search, no timeline, read-write SQL | archived 2025-05-28 | **REJECT** (unmaintained) |
| **`doobidoo/mcp-memory-service`** | **Apache-2.0** | SQLite + **`sqlite-vec`** single-file backend; hybrid semantic+keyword search (`memory_search`), `memory_list`/`memory_delete` by **timeframe**, knowledge graph (`memory_graph`, `memory_explore`, `memory_detail`), doc ingest | Write-oriented memory store (it owns the data); **no encryption at rest**; HTTP transport defaults to **`0.0.0.0:8000`**; no documented read-only mode; no append-only/evidence semantics | 2026-09-25 (very active) | **ADOPT-AS-PATTERN** (architecture) / **REJECT** as a dependency |
| **`basicmachines-co/basic-memory`** | **AGPL-3.0** ⚠️ | Plain-Markdown corpus + full-text + vector + knowledge-graph index; structured search by frontmatter; schemas; MCP tools annotated with behaviour hints incl. read-only; single-user local | Not a read-only reader (it writes the corpus); **AGPL-3.0**; **open-core** (paid "Teams" tier); **no encryption at rest**; timeline/"recent activity" tool **not verified** | 2026-09-24 | **REJECT** as a dependency (AGPL) / **ADOPT-AS-PATTERN** (tool annotations, schemas) |
| **`StevenStavrakis/obsidian-mcp`** | **MIT** | Best-in-survey local boundary: vaults allow-listed at process start, vault-relative segment-checked paths, symlinks blocked, **never listens on a network interface**, no telemetry, stdout reserved for MCP; v2 speaks MCP `2026-07-28` | Obsidian-specific (requires `.obsidian`), Markdown files not a DB, **read-write**, no semantic search, no timeline | 2026-09-10 | **ADOPT-AS-PATTERN** (the containment model) |
| **`MarkusPfundstein/mcp-obsidian`** | **MIT** | Read + write Obsidian notes via the Local REST API plugin | Requires Obsidian running + plugin; read-write; no semantic search; no timeline | (not re-verified) | **WATCH** |
| **`jacksteamdev/obsidian-mcp-tools`** | **MIT** | Semantic search + Templater prompts into Obsidian | **Archived** | archived 2026-05-13 | **REJECT** (archived) |
| **Khoj** (`khoj-ai/khoj`) | **AGPL-3.0** ⚠️ | Self-hostable "second brain": semantic search over local docs (pdf/md/org/word/notion), REST API `/api/search`, multi-client | **No MCP server found by two independent surveys** (open issue #1364, 2026-07-01, asks for exactly this; no MCP page in the docs sitemap, `mcp/README.md` and `src/khoj/mcp/__init__.py` both 404); **AGPL-3.0**; Postgres/pgvector-scale, not single-file; no encryption at rest | 2026-08-02 | **REJECT** (AGPL + no MCP surface) |
| **ArchiveBox** (`ArchiveBox/ArchiveBox`) | **MIT** | Ships an MCP server (`archivebox/mcp/server.py`, protocol `2025-11-25`) with read-only/destructive vocabulary, and a web-archive corpus | Exposes `shell`, `crawl` and `snapshot` to the agent; tools generated by dynamic Click introspection over the CLI; not a personal-message corpus; no encryption at rest | 2026-09-25, 28.6k★ | **ADOPT-AS-PATTERN** (negative lesson, §3.10) / **REJECT** as a dependency |
| **Karakeep** (`karakeep-app/karakeep`) | **AGPL-3.0** ⚠️ | Official MCP server (`ghcr.io/karakeep-app/karakeep-mcp`): `search-bookmarks`, `get-bookmark`, `get-bookmark-content`; per-agent bearer API key (`KARAKEEP_API_KEY`) | Also ships `create-*` write tools; AGPL-3.0; bookmark corpus, not messages/recordings; no encryption at rest | 2026-09-24, 29.3k★ | **ADOPT-AS-PATTERN** (API-key scoping) / **REJECT** (AGPL) |
| **Onyx** (`onyx-dot-app/onyx`) | **MCP *server* is MIT Expat** (verified: `backend/onyx/mcp_server/`, outside `ee/`); the MCP ***client*/actions registry is the Enterprise half** ⚠️ | The one genuinely **read-shaped** MCP surface found: server exposes query tools only — "Search your knowledge base", "Search the web", "Fetch full page content" (`docs.onyx.app/overview/onyx_anywhere/mcp_server`); per-consumer bearer token; authorisation pushed to the data layer | **HTTP only, no stdio**; one of three tools is **open-world egress** (web search / fetch any URL); single coarse scope `mcp:use` (no read/write differentiation); Postgres/Vespa-scale team deployment, not single-user local; no encryption at rest | 2026-09-25, 32.2k★ | **ADOPT-AS-PATTERN** (positive precedent, §3.10) / **REJECT** as a dependency for a local single-user install |
| **AnythingLLM** (`Mintplex-Labs/anything-llm`) | **MIT** | MCP **client** for its agents; servers configured in `anythingllm_mcp_servers.json`; management UI | Does **not** expose its workspace corpus as an MCP server; not a local-corpus provider | 2026-09-25 | **REJECT** as a provider / **WATCH** as consumer pattern |
| **Open WebUI** | **Open WebUI License** ⚠️ (BSD-3-derived **+ branding clause**; non-OSI, source-available) | Chat UI + tools/MCP plumbing | Branding may not be removed above a 50-user threshold; not open source per OSI; not a corpus provider | 2026-09-25 | **REJECT** as a dependency |
| **`chroma-core/chroma-mcp`** | **Apache-2.0** | Semantic query, full-text search, metadata filtering, document CRUD over Chroma collections | No timeline; **no encryption**; read-write; activity has slowed | 2025-09-17 | **WATCH** |
| **`qdrant/mcp-server-qdrant`** | **Apache-2.0** | Official Qdrant MCP: store/find over a Qdrant collection | Requires a separate Qdrant process (not single-file); no timeline; no encryption; read-write | 2026-09-04 | **WATCH** |
| **`asg017/sqlite-vec`** | **Apache-2.0** | Vector search as a SQLite extension, runs anywhere, single file | A library, not a server; no encryption of its own | 2026-05-18 | **ADOPT** (as footprint's vector index) |
| **Joplin MCP servers** (`alondmnt/joplin-mcp`, `biontdv/Joplin-MCP`) | **MIT** (both) | Read/search/write Joplin notes over the local Data API; one supports a **strict read-only mode**; OCR text extraction | Requires Joplin desktop running + API token; no semantic search; no timeline | 2026-09-12 / 2026-07-16 | **ADOPT-AS-PATTERN** (read-only mode) / **REJECT** as dependency |
| **Logseq** | **AGPL-3.0** ⚠️ | Local outliner app | AGPL app, not a server surface | 2026-09-25 | **REJECT** (AGPL) |
| **Reor** | **AGPL-3.0** ⚠️ | Local AI PKM app | **Archived**; AGPL | archived 2025-05-13 | **REJECT** |
| **LlamaIndex** / **Haystack** | **MIT** / **Apache-2.0** | Retrieval frameworks (index, retrieve, rerank) | Frameworks, not MCP servers; no storage policy, no encryption | both active 2026-09-25 | **ADOPT-AS-PATTERN** (library, if needed) |
| **mem0** / **cognee** / **graphiti** | **Apache-2.0** (all three) | Agent memory / graph layers | Write-oriented memory; no read-only corpus view; no encryption | all active 2026-09 | **WATCH** |
| **SQLCipher** (Zetetic) | **BSD-style, Community Edition**, attribution required | Page-level SQLite encryption; key management | Community Edition is attribution-constrained; FIPS/enhanced builds are commercial | 2026-07-30 (licence page) | **ADOPT** (encryption at rest) |
| **`utelle/SQLite3MultipleCiphers`** | **MIT** | SQLite encryption (multiple cipher schemes) | A library; you own key management | 2026-09-24 | **ADOPT** (alternative to SQLCipher, no attribution clause) |
| **`tursodatabase/libsql`** | **MIT** | Embedded SQLite fork with server mode | Encryption-at-rest support **not verified** in this survey | 2026-09-16 | **WATCH** |

⚠️ = licence is a distribution constraint or is not open source. AGPL-3.0 appears three times (Khoj, basic-memory, Reor, plus Logseq): for a *distributable* product this is the single most important licensing hazard in the survey.

---

## 3. Per-candidate detail with citations

### 3.1 The protocol itself

**Specification and versioning.** The spec is published as dated revisions. The page for `2025-06-18` carries a banner reading *"You are viewing an older version (2025-06-18) of the specification. View the latest version (2026-07-28)"* — https://modelcontextprotocol.io/specification/2025-06-18/basic/transports. So the revision ladder observed is 2024-11-05, 2025-03-26, 2025-06-18, 2025-11-25, 2026-07-28. The 2026-07-28 revision moved all request metadata into `_meta.io.modelcontextprotocol/*` fields, added `subscriptions/listen`, and introduced multi-round-trip requests (MRTR) — i.e. the protocol is still moving at the edges even though the core (JSON-RPC 2.0 over stdio/HTTP, tools/resources/prompts) is settled. Source: https://raw.githubusercontent.com/modelcontextprotocol/modelcontextprotocol/main/docs/specification/2026-07-28/basic/transports and `.../basic/transports/stdio.mdx`.

**Transports (Q4).** Two standard transports: **stdio** (client launches the server as a subprocess; newline-delimited JSON-RPC on stdin/stdout; stderr free for logging; the server MUST NOT write non-MCP bytes to stdout) and **Streamable HTTP** (POST to a single MCP endpoint, replies as JSON or a request-scoped SSE stream). The spec states: *"Clients SHOULD support stdio whenever possible."* Source: https://modelcontextprotocol.io/specification/2026-07-28/basic/transports and the stdio binding page.

**Security warnings for a local HTTP server.** From the Streamable HTTP section (2025-06-18 revision, carried forward):

1. the server **MUST** validate the `Origin` header on all incoming connections to prevent DNS-rebinding attacks;
2. when running locally, the server **SHOULD** bind only to localhost (127.0.0.1) rather than all interfaces (0.0.0.0);
3. the server **SHOULD** implement proper authentication for all connections.

Source: https://modelcontextprotocol.io/specification/2025-06-18/basic/transports (read via the static mirror `mcp.gjxx.dev`, since the canonical page would not render for this tool; the wording is unambiguous and matches the 2026-07-28 page structure).

**Authorization (Q2).** The authorization framework is **optional**, and explicitly scoped to HTTP transports:

- *"Authorization is optional for MCP implementations."*
- HTTP-based transports **SHOULD** conform to the spec.
- **STDIO implementations SHOULD NOT follow this spec, and instead retrieve credentials from the environment.**
- The authorization server **MUST** implement OAuth 2.1.
- The MCP server **MUST** implement OAuth 2.0 Protected Resource Metadata (RFC 9728); the client **MUST** use it for authorization-server discovery; the authorization server **MUST** provide Authorization Server Metadata (RFC 8414).
- OAuth 2.1, RFC 8414, RFC 7591 (dynamic client registration) and RFC 9728 are the referenced standards.

Source: https://modelcontextprotocol.io/specification/2025-06-18/basic/authorization. I could **not** read the `2026-07-28` authorization page directly (the fetch returned empty), so I cannot confirm whether resource indicators (RFC 8707) or Client ID Metadata Documents became normative in the newest revision — see §5.

**Consequence for footprint:** MCP gives you *authentication* (who is calling) but never *authorization semantics* (what they may read). There is no scope like `corpus:read` defined by the protocol, no "read-only mode", and no way for the protocol to stop a server from exposing a write tool. If Hermes is launched by footprint over stdio, the MCP auth framework does not even apply — credentials come from the environment. The boundary is entirely footprint's to design.

**Tool annotations are not a security boundary.** MCP tools may carry four boolean annotations — `readOnlyHint` (default `false`), `destructiveHint` (default `true`), `idempotentHint` (default `false`), `openWorldHint` (default `true`). The defaults are pessimistic on purpose: an un-annotated tool is treated as write-capable, destructive, non-idempotent and open-world. The spec's position is that clients **must treat annotations as untrusted unless they come from a trusted server**, because an untrusted server can simply lie. Sources: the MCP organisation's own post https://blog.modelcontextprotocol.io/posts/2026-03-16-tool-annotations/ and a detailed secondary analysis quoting the spec at https://dreaming.press/posts/mcp-tool-annotations-explained.html. **Caveat:** the canonical tools page (https://modelcontextprotocol.io/specification/2026-07-28/server/tools) would not render for this tool, so I am relying on the spec's own blog post plus a quoting secondary source for the exact wording. Verify the sentence before citing it in a design doc.

**Governance and registry (Q4).** Anthropic announced the donation of MCP to the Linux Foundation's **Agentic AI Foundation (AAIF)** on 2025-12-09: https://www.anthropic.com/news/donating-the-model-context-protocol-and-establishing-of-the-agentic-ai-foundation and https://www.linuxfoundation.org/press/linux-foundation-announces-the-formation-of-the-agentic-ai-foundation. The official registry (https://registry.modelcontextprotocol.io) launched **in preview on 2025-09-08** and still self-describes as *"A community driven registry service for Model Context Protocol (MCP) servers"* (https://registry.modelcontextprotocol.io/docs). The registry page is JavaScript-rendered, so I could not read a server count.

**Licensing of the reference servers.** The `modelcontextprotocol/servers` LICENSE file reads:

> *"The MCP project is undergoing a licensing transition from the MIT License to the Apache License, Version 2.0 … All new code and specification contributions to the project are licensed under Apache-2.0. Documentation contributions (excluding specifications) are licensed under CC-BY-4.0."*

GitHub therefore reports the licence as `NOASSERTION`/`Other`, but the substance is **MIT and Apache-2.0 (mixed during transition)** — permissive either way. The repo's README also carries an important warning:

> *"The servers in this repository are intended as **reference implementations** … They are meant to serve as educational examples for developers building their own MCP servers, **not as production-ready solutions**."*

Source: https://raw.githubusercontent.com/modelcontextprotocol/servers/HEAD/README.md.

### 3.2 Official reference servers

**Filesystem.** The README describes it as *"Secure file operations with configurable access controls"*. The design (allow-listed directories passed at startup, path validation, roots as client-advertised boundaries) is the closest thing in the official set to a containment model, and it is worth copying. **I could not read the filesystem README verbatim** — the raw URL returned empty for this tool on three attempts — so treat the specific symlink/roots semantics as unverified until read (§5).

**Memory.** A local knowledge graph: entities carry `name`/`entityType`/`observations`; relations are directed and stored in active voice; tools are `create_entities`, `create_relations`, `add_observations`, `delete_entities`, `delete_observations`, `delete_relations`, `read_graph`, `search_nodes`, `open_nodes`. `search_nodes` is a keyword match over entity names/types/observations — **not semantic**. There is no time dimension and no read-only mode. Source: https://raw.githubusercontent.com/modelcontextprotocol/servers/HEAD/src/memory/README.md. Useful as a *shape* for footprint's entity model; useless as footprint's search.

**SQLite.** Archived. It now lives in `modelcontextprotocol/servers-archived` (MIT, archived 2025-05-28, per GitHub API) — https://github.com/modelcontextprotocol/servers-archived/tree/main/src/sqlite. Community successors exist (e.g. `johnnyoshika/mcp-server-sqlite-npx`, `jparkerweb/mcp-sqlite`), but none is the official one and none adds vector search or timeline semantics. **REJECT** the archived server; if footprint wants a SQL surface it should expose a narrow, purpose-built tool set rather than raw SQL, because raw SQL over a corpus is an unbounded read surface that leaks everything.

### 3.3 `doobidoo/mcp-memory-service` — the closest *architecture*

Licence **Apache-2.0** (GitHub API), 1,965★, pushed 2026-09-25. Default backend is **`sqlite-vec`**, a single file:

> *"Lightweight: Single file database with no external dependencies … Portable: Easy to backup, copy, and share memory databases … Memory Efficient: Low memory footprint suited to single-user deployments."*
> Source: https://raw.githubusercontent.com/doobidoo/mcp-memory-service/main/docs/sqlite-vec-backend.md

The v10 unified tool surface (from https://raw.githubusercontent.com/doobidoo/mcp-memory-service/main/docs/mastery/api-reference.md) includes `memory_search` (*"Hybrid semantic + keyword search"*), `memory_list` (*"List/filter memories"*), `memory_delete` (*"Delete by hash, tag, **timeframe**, or filter"*), `memory_ingest`, and a graph family `memory_graph` / `memory_explore` / `memory_detail`. Time-range filtering therefore exists, but as a filter on a write-oriented store, not as a chronological browse.

Two things disqualify it as a dependency:

- **Transport defaults to an open socket:** *"Transport: `mcp.run("streamable-http")`, default host `0.0.0.0`, default port `8000`"* — the exact configuration the MCP spec warns against.
- **No encryption.** The sqlite-vec backend guide documents the file path (`MCP_MEMORY_SQLITE_PATH`) and portability, and mentions no encryption, key management, or at-rest protection.

Also relevant: it is a **memory service**, i.e. the agent writes memories into it. footprint's corpus is captured by footprint's own pipelines (chats, recordings, email) and Hermes must not be able to mutate it. The data-flow direction is inverted.

**Verdict:** ADOPT-AS-PATTERN for "SQLite + sqlite-vec + MCP in one process, single file, hybrid search"; REJECT as a dependency (no encryption, write-oriented, 0.0.0.0 default).

### 3.4 `basicmachines-co/basic-memory` — the closest *functionality*, blocked by AGPL

Licence **AGPL-3.0** (GitHub API, `spdx_id: "AGPL-3.0"`), 4,034★, pushed 2026-09-24. Its own site (https://basicmemory.com) describes exactly the shape footprint wants for search:

> *"Plain Markdown you own, with a full-text, semantic, and knowledge-graph index on top. The files stay plain; the search gets smart."*
> *"Semantic search — Find by meaning, not just keywords. Vector and hybrid search across everything you've written."*
> *"Structured search — Filter by frontmatter. Status, type, tags, priority, or any custom field."*
> *"Knowledge graph — Typed relations and observations connect notes into a graph your AI can actually traverse."*

The repo README states that *"Every tool is annotated with MCP behavior hints (read-only, destructive, idempotent, open-world)"* (quoted at https://github.com/basicmachines-co/basic-memory). So the project is a working demonstration of two patterns footprint should copy: **MCP tool annotations used honestly**, and **structured frontmatter/schema filtering layered over a plain-text corpus**.

Why it cannot be adopted:

- **AGPL-3.0.** For a distributable product, AGPL is a distribution constraint. Even for a locally-run desktop app, the safe engineering position is to not link to, vendor, or derive from AGPL code. (This is a legal judgement, not a technical one — footprint should get its own advice before relying on any AGPL component.)
- **Open core.** The site leads with *"NEW · Basic Memory for Teams is live →"* and prices the local tier at `$0 · open source`. The local OSS version is the one that would matter, but it means the vendor's incentives point at the hosted tier.
- **Wrong data-flow direction.** It is a memory store the agent writes to. footprint's corpus is append-only and written by capture pipelines, not by the consumer.
- **No encryption at rest** documented anywhere on the site or in the repo metadata I read.
- I could **not** verify a timeline / "recent activity" tool; the documented retrieval is full-text + vector + hybrid + graph traversal (https://deepwiki.com/basicmachines-co/basic-memory/3.2-search-and-context-tools, secondary source).

**Verdict:** REJECT as a dependency; ADOPT-AS-PATTERN for annotations, schema/frontmatter filtering, and the "plain text + index" separation.

### 3.5 `StevenStavrakis/obsidian-mcp` — the best local containment design

Licence **MIT** (GitHub API), 735★, pushed 2026-09-10, and v2 speaks the `2026-07-28` revision. From its README (https://raw.githubusercontent.com/StevenStavrakis/obsidian-mcp/main/README.md):

> *"Vault access is explicitly allowlisted at process startup."*
> *"Every tool path is vault-relative, segment-checked, and blocked from symlinks and reserved state."*
> *"File mutations are journaled, conflict-checked, atomically replaced, and rolled back as one transaction."*
> *"The server never listens on a network interface or sends telemetry."*
> *"stdout is reserved exclusively for MCP messages; structured diagnostics go to stderr."*
> *"Results are bounded, paginated where appropriate, and available as both text and structured content."*

It also treats the *explicit allow-list itself* as the authorization decision: *"an explicit `--vault` is treated as authorization"* — and it warns users that MCP clients can invoke destructive tools, recommending backups and review of permission prompts.

This is the single best answer in the survey to "how does a local server enforce a boundary the consumer cannot cross": **one process per corpus, allow-list fixed at launch, no network listener at all, path containment checked on every call, and bounded results**. footprint should copy this shape almost wholesale for its read-only surface. Its limitations are only that it is Obsidian-specific, read-write, and does no semantic search.

### 3.6 Khoj

Licence **AGPL-3.0** (GitHub API), 37,490★, pushed 2026-08-02. Self-hostable, semantic search over local docs (pdf/markdown/org/word/notion), a REST API at `/api/search`, and multi-client access (Obsidian/Emacs/desktop/phone). Its README claims *"Khoj is open-source, self-hostable. Always."*

But the MCP surface is not there. GitHub issue **#1364**, *"Docs/Feature: MCP server endpoint undocumented — no guide for wiring self-hosted Khoj as MCP knowledge source for Claude Code / Cursor / Kilo Code"*, opened 2026-07-01, asks precisely: *"Does Khoj expose an MCP-compatible server? If yes, at what path/port? What transport does it use? … What tools does it expose? … What authentication is needed?"* and notes *"Khoj REST API at `/api/search` exists but MCP wrapping is undocumented."* A predecessor issue (#1023, "Evaluate Claude MCP with Khoj") was closed without a guide. The current docs page https://docs.khoj.dev/features/agents/ describes agents and custom prompts but does not mention MCP.

So Khoj is a full competing application with a search API, not an MCP server footprint can consume. Combined with AGPL-3.0 and a Postgres/pgvector-scale deployment model (not single-file, not per-install), it is a **REJECT** — though its retrieval-quality benchmarks are worth reading as a sanity check on footprint's own search ambitions.

### 3.7 AnythingLLM, Open WebUI, Notion — the "consumer side"

**AnythingLLM** is **MIT** (verified from the LICENSE file) and 66k★. It is an MCP **client**: *"AnythingLLM supports all Model Context Protocol (MCP) tools for use with AI Agents"*, configured through `anythingllm_mcp_servers.json` with a management UI (https://docs.anythingllm.com/mcp-compatibility/overview). It does **not** expose its workspace corpus as an MCP server. Useful to footprint only as a reference for how a desktop app hosts and manages MCP servers — and as evidence that stdio + config-file is the accepted desktop pattern.

**Open WebUI** is **not open source**. The LICENSE file is a BSD-3-derived text with an added clause 4:

> *"Notwithstanding any other provision of this License … licensees are strictly prohibited from altering, removing, obscuring, or replacing any 'Open WebUI' branding … except (i) deployments … where the total number of end users … does not exceed fifty (50) within any rolling thirty (30) day period; (ii) the licensee has obtained specific prior written permission …; or (iii) … a duly executed enterprise license."*

Adding a field-of-use/branding restriction to BSD-3 makes it source-available but not OSI-approved open source. It is a **REJECT** as a dependency. (The file also notes `LICENSE_HISTORY` retains earlier MIT/BSD terms for older material.)

**Notion's official MCP server** (`makenotion/notion-mcp-server`) is **MIT** (verified from the LICENSE file). I include it only to close the loop from the brief: it is a client of Notion's **hosted** API, so using it would send the corpus to a third party — directly contrary to footprint's PDPA purpose-limitation and no-third-party-disclosure constraints. **REJECT** for footprint's surface; noted for completeness.

### 3.8 Vector and graph components

- **`asg017/sqlite-vec`** — **Apache-2.0**, 8,134★, pushed 2026-05-18. *"A vector search SQLite extension that runs anywhere!"* This is the natural vector index for footprint's single-file corpus: same file, same transaction, same encryption story as the rest of the DB. **ADOPT.**
- **Chroma** (`chroma-core/chroma-mcp`, **Apache-2.0**) — semantic query + full-text + metadata filtering over collections. Note the server activity has slowed (last push 2025-09-17 vs. active peers), it is read-write, and it offers no timeline. **WATCH.**
- **Qdrant** (`qdrant/mcp-server-qdrant`, **Apache-2.0**, 1,538★, pushed 2026-09-04) — official, active, but requires a separate Qdrant process, which is wrong for a single-user install-without-sudo product. **WATCH.**
- **LanceDB** — **Apache-2.0**, embedded, active; a viable alternative to sqlite-vec if footprint ever needs multimodal vectors. **WATCH.**
- **mem0 / cognee / graphiti** — all **Apache-2.0**, all active, all write-oriented agent-memory layers with no read-only corpus view and no encryption. **WATCH.**
- **LlamaIndex (MIT) / Haystack (Apache-2.0)** — retrieval frameworks. If footprint needs reranking or evaluation harnesses, these are the permissively-licensed choices. **ADOPT-AS-PATTERN.**

### 3.9 Encryption at rest (Q3)

**No MCP server in this survey implements application-level encryption at rest.** I looked for it specifically in the `sqlite-vec` backend documentation of mcp-memory-service (the most storage-explicit candidate) and in the basic-memory and Chroma documentation, and found none. The ecosystem assumption is that the host's disk encryption covers you — which is exactly the assumption footprint cannot make (the reference host is Ubuntu 26.04 on a plain ext4 root, no LUKS).

The realistic building blocks, all verified:

- **SQLCipher, Community Edition** — *"available under a BSD-style license and requires attribution and reproduction of license grants in the application interface and/or materials … in an about or licensing screen in the application, in the product documentation on a website linked from the application, but must be a user accessible location."* The Commercial Edition covers enhanced performance/FIPS/etc. Source: https://www.zetetic.net/sqlcipher/license/. Note the attribution obligation is a real product requirement, not a footnote.
- **`utelle/SQLite3MultipleCiphers`** — **MIT**, 682★, pushed 2026-09-24. Same job, no attribution clause, MIT-clean. Source: GitHub API.
- **`tursodatabase/libsql`** — **MIT**, 17,235★. A SQLite fork with server mode; encryption-at-rest support **not verified** here.

The only app-level E2EE precedent in the personal-knowledge space is **Joplin**, whose note-level E2EE encrypts note content and resources (but not the whole local database). That precedent matters for footprint's threat model: even a PKM app with an explicit E2EE feature does not encrypt its entire local store, so footprint's "whole corpus ciphertext at rest" requirement is genuinely more ambitious than anything in this ecosystem.

**One design consequence worth flagging now:** if the MCP server process must answer semantic queries, it must hold the decryption key while running. Encryption at rest therefore does not protect against a compromised *running* Hermes session — it protects the corpus at rest and in backups. That is still the right property, but it means the read-only boundary and the key lifetime are separate problems.

### 3.10 Read-only is not free: evidence from the parallel surveys

Two sibling surveys in this repo (`docs/research/oss/archive-platforms.md`, plus the transcription memo) turned up three further projects that expose an MCP server. I verified their licences directly via the GitHub API; the MCP tool inventories below are their findings, and I flag them as such rather than as my own primary-source reads.

- **ArchiveBox** — **MIT**, 28,592★, pushed 2026-09-25. Its MCP server (`archivebox/mcp/server.py`, protocol revision `2025-11-25`) declares a read-only/destructive vocabulary (`READ_ONLY_ACTIONS = {"help", "list", "search", "status", "version"}`, `DESTRUCTIVE_ACTIONS = {"delete", "remove"}`) and yet exposes `add`, `search`, `crawl`, `snapshot`, `archiveresult` and **`shell`** to the agent. The tools are generated by **dynamic Click introspection over the CLI**, which is precisely the mechanism by which a boundary erodes: adding a CLI command silently adds an agent tool. Licence verified; the tool inventory is from the archive-platform survey.
- **Karakeep** — **AGPL-3.0** ⚠️, 29,264★, pushed 2026-09-24. Official MCP server image `ghcr.io/karakeep-app/karakeep-mcp` with read tools `search-bookmarks`, `get-bookmark`, `get-bookmark-content` **plus** `create-*` write tools. Authentication is a **per-agent bearer API key** (`KARAKEEP_API_KEY`). The AGPL licence is a blocker for footprint, but the **API-key-per-agent** pattern is the closest thing in the survey to reusable per-consumer scoping, and it is worth copying: issue Hermes its own key, bind that key to a read-only scope, and revocation becomes a single operation rather than a redeploy.
- **Onyx** — **the MCP server is MIT Expat; the MCP client is the Enterprise half.** This is the one genuinely read-shaped MCP surface in either survey, so it is the positive precedent rather than a cautionary tale. Verified directly (2026-09-25):
  - The server code is **outside** `ee/`. `backend/onyx/mcp_server_main.py` exists and is titled *"Entry point for MCP server - HTTP POST transport with API key auth"*; it runs `uvicorn` on `MCP_SERVER_HOST`/`MCP_SERVER_PORT`. `backend/onyx/mcp_server/api.py` builds a `FastMCP` instance with `auth=OnyxTokenVerifier()` and imports exactly two things: `from onyx.mcp_server.tools import search` and `from onyx.mcp_server.resources import indexed_sources`. `backend/onyx/mcp_server/tools/__init__.py` contains only `from onyx.mcp_server.tools import search` — i.e. **one tool module, no CLI reflection and no ORM reflection**, the exact failure mode that sinks ArchiveBox.
  - The documented tool surface is three query capabilities: *"Search your knowledge base"*, *"Search the web"*, *"Fetch full page content"*, with *"All of your existing Onyx permissions and access controls are enforced automatically"* (https://docs.onyx.app/overview/onyx_anywhere/mcp_server).
  - Authentication is a **per-consumer bearer token** (`Authorization: Bearer`, Personal Access Token or API key). `backend/onyx/mcp_server/auth.py` shows `OnyxTokenVerifier.verify_token` delegating to the Onyx API's `/me` endpoint and, on success, returning `AccessToken(..., scopes=["mcp:use"], ...)`.
  - The Enterprise half is where the MCP *client* machinery lives (`mcp.json`, `mcp_registry`, `mcp_servers`) — that is Onyx's ability to *call* external MCP servers, which footprint does not need. The root LICENSE grants "MIT Expat" to everything outside `ee/` and names `backend/ee/LICENSE`, `web/src/app/ee/LICENSE` and `web/src/ee/LICENSE` as the Enterprise carve-outs.

  **What footprint should take from it, and what it should not.** Take: one tool module per capability, no reflection, a per-consumer bearer credential, and authorisation enforced below the tool layer. Do **not** take the rest: the transport is **HTTP-only** (no stdio — bad for a desktop app, and it contradicts the spec's own "bind only to localhost" advice unless you configure it), **one of the three tools is open-world egress** ("Search the web", "Fetch full page content" — footprint would be handing Hermes a general internet fetch, which no recall interface needs), and the scope is a single coarse `mcp:use` with **no read/write differentiation**. Onyx can push authorisation down to its data layer because it has per-document ACLs; footprint is single-user with no ACLs to push down to, so footprint must get the restriction from the *tool surface* instead. That is precisely why the two controls are complementary rather than alternatives: a per-consumer key scopes **which consumer**, an allow-list plus a read-only DB role scopes **which operations**. footprint needs both.

The lesson generalises and is the strongest argument for how footprint should build its surface: **shipping an MCP server does not mean shipping a read-only one.** Three of these projects are permissively or partially permissively licensed and all three hand the agent more capability than a recall interface needs. The archive-platform survey's recommendation is the right one and I adopt it here: enforce read-only with an **explicit tool allow-list plus a read-only database role** — two independent controls — and not by convention, not by annotation, and never by dynamically reflecting a CLI or an ORM into tool definitions.

---

## 4. Closest fits and the gap

Ranked by how much of footprint's surface each already covers.

1. **`basicmachines-co/basic-memory`** — closest on *function*: local, single-user, plain-text corpus with a full-text + semantic + graph index and structured filtering, exposed over MCP, with honest tool annotations. **Gap:** AGPL-3.0 (distribution blocker), open-core, write-oriented (the agent authors the corpus rather than reading a captured one), no encryption at rest, and no verified timeline/append-only evidence model.
2. **`doobidoo/mcp-memory-service`** — closest on *architecture*: exactly the SQLite + `sqlite-vec` + MCP-in-one-process design footprint would choose, with hybrid search and time-range filtering. **Gap:** it is a memory store the consumer writes into, unencrypted, and its HTTP transport defaults to `0.0.0.0:8000`. No read-only mode, no evidence semantics.
3. **`StevenStavrakis/obsidian-mcp`** — closest on *boundary design*: allow-list at launch, per-call path containment, no network listener, bounded results, MCP `2026-07-28`. **Gap:** Obsidian-specific, Markdown not SQLite, read-write, no semantic search, no timeline.
4. **`chroma-mcp` / `mcp-server-qdrant`** — closest on *retrieval tooling*: clean semantic-search tools over a vector store. **Gap:** no timeline, no encryption, read-write, and either a slowing project or an extra daemon.
5. **Onyx** (`onyx-dot-app/onyx`) — closest on the *read-only surface property itself*, and the only project in either survey that ships one deliberately: one tool module, three query tools, per-consumer bearer token, no reflection (§3.10). **Gap:** it is a Postgres/Vespa-scale team deployment, not a single-user local install; HTTP-only, no stdio; one of its three tools is open-world internet egress; its read-only guarantee comes from per-document ACLs at the data layer, which footprint does not have; and the surrounding product is open-core. Its MCP server is MIT, so its *design* is free to copy.

**The single most important gap.** Nothing in this ecosystem combines the three properties that define footprint's surface: **(a) read-only** over a corpus it did not author, **(b) encrypted at the application level** so it survives an unencrypted host disk, and **(c) append-only with verifiable timestamps**. Every candidate either owns and mutates its data, or delegates encryption to the OS, or both. Adopting any of them wholesale would mean adopting the wrong data-flow direction and the wrong at-rest posture. What footprint should take from this survey is a small set of patterns plus two libraries:

- **ADOPT** `sqlite-vec` (Apache-2.0) as the vector index inside the same encrypted SQLite file.
- **ADOPT** SQLCipher Community Edition (BSD-style, with an in-app attribution screen) or `SQLite3MultipleCiphers` (MIT) for encryption at rest.
- **ADOPT-AS-PATTERN** the `obsidian-mcp` containment model for the server process, and the official `filesystem` server's allow-listed-directories idea.
- **ADOPT-AS-PATTERN** basic-memory's honest `readOnlyHint` annotations and frontmatter/schema filtering — while remembering that annotations are *hints*, so the real enforcement is that footprint simply does not register a write tool at all.
- **ADOPT** the MCP protocol itself, over **stdio**, pinned to a specific revision.
- **ADOPT-AS-PATTERN — the Onyx lesson, inverted:** a **non-mutating** surface is not a **confined** one. Onyx's read-shaped surface still exposes "Search the web" and "Fetch full page content", so it is read-only in the write sense and unbounded in the *egress* sense. For a participant-only archive whose entire premise is a closed corpus, **Hermes should have no web-search tool and no arbitrary-URL fetch tool at all**; every tool should resolve against the local corpus and nothing else. Read-only and no-egress are two separate requirements, and the second is the one an adopter is most likely to inherit by accident.

---

## 5. Open questions and things I could not verify

**Protocol facts I could not read directly (fetch returned empty on these specific pages):**

- The `2026-07-28` **authorization** page (`…/basic/authorization/index.mdx`, and the rendered `modelcontextprotocol.io/specification/2026-07-28/basic/authorization`). I used the `2025-06-18` revision for the normative statements. **Unverified:** whether resource indicators (RFC 8707) / audience binding, or Client ID Metadata Documents, became normative in the newest revision. A search result suggested a `2025-11-25` revision added "CIMD" and a "step-up flow" — treat as unconfirmed.
- The **tools / annotations** page (`…/server/tools.mdx`). My annotation claims come from the MCP organisation's own blog post and one secondary article that quotes the spec. Verify the exact sentence at https://modelcontextprotocol.io/specification/2026-07-28/server/tools before relying on it in a design document.
- The **official filesystem server README** — empty on three attempts (raw + jsDelivr + cache-busted). The allow-list/roots/symlink specifics are therefore second-hand from the servers README one-liner and the containment pattern observed in `obsidian-mcp`. Read the file before copying its semantics.
- **MCP registry** server count and current stability wording — the registry page is JavaScript-rendered and returned only "Loading servers…". Preview status since 2025-09-08 comes from a search result, not from the registry itself.

**Licences I did not verify:**

- The `modelcontextprotocol/modelcontextprotocol` **specification repo** licence (the `servers` repo is MIT/Apache-2.0 mixed; the spec repo is a different repo and I did not read its LICENSE).
- The official **SDK** licences (TypeScript, Python, Go, Rust, Java, Kotlin, C#, PHP, Ruby, Swift). All are listed in the servers README; none was checked. If footprint builds on an SDK, read that SDK's LICENSE.
- `MarkusPfundstein/mcp-obsidian` activity date — licence verified as MIT, recency not re-verified.
- `tursodatabase/libsql` encryption-at-rest capability.
- **Which Onyx connectors *beyond a known-fifteen set* are Enterprise-gated.** Narrowed from "which connectors are EE-gated" after the archive-platform survey enumerated fifteen. I re-probed the **seven that matter to footprint** — `gmail`, `outlook`, `imap`, `slack`, `teams`, `zoom`, `fireflies` — and confirm each is **present in the MIT tree** (`backend/onyx/connectors/<name>/connector.py` resolves) and **absent under `ee/`** (all seven `backend/ee/onyx/connectors/<name>/connector.py` return 404). Two of them (`zoom`, `fireflies`) returned real source text; the other five returned an empty body, which is this tool's known non-404 failure mode rather than a missing file. So **every footprint-relevant capture path — email, chat, meeting transcripts — is MIT-licensed code**, which is the favourable direction for us. I did **not** re-probe the remaining eight (`discord`, `google_drive`, `confluence`, `jira`, `notion`, `github`, `sharepoint`, `salesforce`), so the teammate's result stands on their evidence for those.
- **Correction to a method claim: directory existence cannot be probed this way.** The archive-platform survey recorded that `backend/ee/onyx/connectors/` and `backend/ee/onyx/external_permissions/` "do exist (HTTP 200)". My probe returns **404 for both** — and, decisively, also 404 for `backend/onyx/connectors/`, a directory that **provably exists** because `backend/onyx/connectors/gmail/connector.py` returns 200. `raw.githubusercontent.com` serves files, not directory listings, so a directory path 404s regardless of whether the directory is there. A bogus control (`backend/onyx/connectors/__nonexistent_dir__/`) also 404s, so the test cannot distinguish the two cases. **Resolved via a method that does work:** the GitHub **contents API** and **git trees API** both list directories (`api.github.com/repos/onyx-dot-app/onyx/git/trees/main:backend/ee/onyx/connectors`), and a bogus-path control returns `{"message":"Not Found","status":"404"}` there, so they genuinely discriminate. Results:
  - `backend/ee/onyx/connectors/` **exists but contains no connectors at all** — exactly three files, `capability_applicability.py` (1,134 B), `capability_checks.py` (4,328 B) and `perm_sync_valid.py` (4,614 B), with `"truncated": false` on the listing. They are capability/permission-sync **gating helpers**, not connector implementations. So the archive-platform survey's caveat that "Onyx ships *some* EE-only connectors, I just didn't identify which" is **not supported**: on this evidence the EE connector directory holds no connectors.
  - `backend/ee/onyx/external_permissions/` **does** contain per-connector subdirectories — `box`, `canvas`, `confluence`, `github`, `gmail`, `google_drive` and more. This is the Enterprise half that matters, and it confirms the important half of that caveat in a sharper form: the **per-connector ACL/permission-sync implementations** are Enterprise-licensed. Note that `gmail` appears in **both** trees — `backend/onyx/connectors/gmail/connector.py` is MIT, while `backend/ee/onyx/external_permissions/gmail/` is not. Ingest is MIT; the enforcement layer that sits on top of it is not. For footprint that is the favourable way round, because footprint is single-user and has no per-document ACLs to sync.
  - **The Enterprise enforcement is broader than ACL bookkeeping, and I read the file that proves it.** `backend/ee/onyx/external_permissions/post_query_censoring.py` exists and imports `get_all_censoring_enabled_sources` and `get_source_perm_sync_config`, operates on `InferenceChunk` and `User`, and defines `_get_all_censoring_enabled_sources()`. So **post-retrieval censoring of returned result chunks is Enterprise code**, not just the ACL sync that decides who may see a document. That is the strongest form of the "enforcement is not portable" point: the licence boundary sits on *filtering the results that come back from a query*, which is precisely the job footprint's read-only surface has to do itself. The same directory also holds `perm_sync_types.py` (importing `DocExternalAccess`/`ElementExternalAccess`/`NodeExternalAccess`) and `sync_params.py`.
  - The file-level findings above are unaffected either way, since 404-vs-not-404 is reliable for files.
- **Onyx's exact per-tool inventory comes from its documentation, not its source.** I verified the module structure directly (`tools/__init__.py` imports only `search`; `api.py` imports only `tools.search` and `resources.indexed_sources`), but `tools/search.py` itself would not render for this tool, so the three capability names are quoted from https://docs.onyx.app/overview/onyx_anywhere/mcp_server rather than read from the decorators. **Consequence: that list can silently age.** Neither survey read the tool bodies, so treat "three query capabilities" as docs-derived, not exhaustive — a future Onyx release could add a mutating or egress tool and this description would not change. Re-check it against `backend/onyx/mcp_server/tools/` before building anything on it.

**Substantive unknowns:**

- **The transcript schema must not carry one authoritative `language` field.** The transcription survey (`docs/research/oss/transcription-malay.md`) found that Whisper-family models condition the decoder on a **single** language token per 30-second window, so intra-sentence Malay/English switching is a known accuracy cliff, and Hermes' local faster-whisper path takes one language hint defaulting to `en`. Any read-only surface footprint exposes over transcripts should therefore carry **per-segment** language (or an explicit `unknown`/`mixed` value) rather than a document-level language, so that a wrong hint degrades visibly instead of silently and a future code-switching fix has somewhere to land. This is a schema decision for the interface, not just a transcription-quality issue.
- **No accuracy benchmark exists** for any of these tools on **Malay/English code-switching** retrieval. I found no evaluation of embedding or reranking models on Malaysian code-switched text. This is a genuine unknown for footprint's search quality, not just a gap in my survey.
- **No candidate documents an append-only or tamper-evident model** (hash chaining, signed timestamps). Nothing in this survey is prior art for footprint's evidence constraint.
- **No candidate documents an MCP-level read-only enforcement mechanism** beyond "don't expose write tools". If Hermes is launched over stdio it inherits footprint's process identity, so the only real boundary is the tool set footprint registers plus OS-level file permissions.
- **Khoj's MCP status may have changed** since issue #1364 was opened on 2026-07-01; I checked the agents documentation on 2026-09-25 and it still did not mention MCP, but I did not search the full docs tree.
- **Encryption-absence is an absence of evidence.** I searched the storage documentation of the most storage-explicit candidates; I did not read every repository's source. A project could encrypt and simply not document it.
