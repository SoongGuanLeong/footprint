# OSS archive / personal-knowledge / search platforms — survey against the footprint bar

**Status:** research memo, persisted
**Date:** 2026-09-25
**Author:** oss-archive-platforms (research teammate)
**Scope:** survey of free / open-source platforms that already do some part of "capture many
sources into one searchable local store", assessed against what *footprint* needs.
**Method:** primary sources only — each project's own repository, README, `LICENSE` file and
documentation site. Licenses were read from the raw `LICENSE` file or the repo's own license
metadata, not from blog posts. Every claim below that I could not verify is marked as such.
Release data was read from each repo's `/releases/latest` on 2026-09-25.

---

## 1. The question, and the short answer

**Question.** Does an existing free/OSS project already do most of what footprint needs — a
*single-user*, participant-only personal work archive that ingests **email + chat + meeting
recordings/transcripts**, stores them **encrypted at the application level**, searches them
**semantically and by timeline**, and exposes a **read-only surface an external agent (Hermes)
can query**?

**Short answer: no. No single project does this, and the gap is not marginal — it is structural.**

The market splits cleanly into two halves, and nobody bridges them:

- **Capture-breadth products** (Onyx, AnythingLLM, Open WebUI) ingest many sources including
  email and chat — but they are multi-user/org products, store data in the clear, and are built
  to be deployed as a service. They answer "search my company's knowledge", not "archive what I
  personally participated in".
- **Local-archive products** (Khoj, Karakeep, ArchiveBox, Trilium, Memos, piler, notmuch, mu)
  are single-user-first and some have real encryption or real agent surfaces — but each covers
  exactly one content type: bookmarks, web pages, notes, or email. None of them ingests meeting
  media, and piler is the only one that even attempts archival rigour for email — and its
  cryptography does not hold up on inspection (§3.2).

Three specific things nobody in this survey combines:

1. **Participant-scoped multi-source capture.** Onyx comes closest (Gmail via delegated OAuth,
   Slack, Discord, Fireflies meeting transcripts) but its Gmail connector also ships a
   *service-account* mode that is tenant-wide — exactly the pattern footprint forbids.
2. **Application-level encryption of the store.** Only piler (email only — and weakly, §3.2) and
   Trilium (notes only, per-note) do anything at this layer. Paperless-ngx states in its own README that data is
   "stored in clear text without encryption". Khoj, Karakeep, ArchiveBox, Onyx, Memos and
   AnythingLLM document no app-level encryption at all.
3. **A read-only agent surface over an encrypted local store.** Karakeep and ArchiveBox ship MCP
   servers; Onyx now ships one too. But Karakeep's is over bookmarks, ArchiveBox's exposes
   *destructive* tools as well as read tools, and none of the three sits on an encrypted store.

**Practical consequence for footprint:** the recommendation is not "adopt X". It is
**ADOPT-AS-PATTERN across four projects** — Khoj for semantic recall over a local LLM, Karakeep
for the single-user-self-host-plus-MCP delivery shape, ArchiveBox for durable artifact provenance,
and Memos for the chronological timeline UX. **piler** was a fifth until its cryptography was
read: it keeps only its *feature checklist* (legal hold, retention, dedup, audit-log search), not
its implementation — see §3.2. The capture layer and the encrypted store have to be built;
nothing in this survey supplies either.

---

## 2. Comparison table

Legend for the license column: **AGPL-3.0** and **GPL-3.0** are strong copyleft — a network-use
or linking constraint for a distributable product. **BUSL**, **Open WebUI License** and
**Onyx Enterprise License** are *source-available but not open source*; see §5.

| Candidate | License (verified) | Covers | Does NOT cover | Last release (2026-09-25) | Verdict |
|---|---|---|---|---|---|
| **Onyx** (ex-Danswer) | MIT Expat outside `ee/`; **Onyx Enterprise License** inside `ee/` (open core) | 56 official connectors incl. Gmail (OAuth **and** service-account), Google Drive, Slack (indexed + federated), Discord, **Fireflies** (meeting transcripts), Jira, Notion, Linear, Confluence; semantic + keyword search; local LLMs; **MCP client *and* MCP server** (the server is MIT-side, the client registry is `ee/`); REST API; RBAC/multi-user | App-level encryption; single-user simplicity (multi-service deploy, though now `onyx-cli deploy install` with Lite/Standard); WhatsApp/Teams/WeChat; local device capture; EE-gated features | v4.8.1, 2026-09-24 | **ADOPT-AS-PATTERN** (closest whole-product shape, wrong product type) |
| **Khoj** | **AGPL-3.0** | Docs/PDF/Markdown/Org, Notion, GitHub, web search; semantic + keyword search; chat, agents, automations; local LLMs (Ollama, LM Studio, LiteLLM); REST API; Obsidian/Emacs/desktop/WhatsApp clients | **Email, chat history, meeting media/transcription — none**; no app-level encryption; **no MCP server found**; cloud edition is multi-user | 2.0.0-beta.28, 2026-03-26 | **ADOPT-AS-PATTERN** (semantic recall + local LLM) |
| **Karakeep** (ex-Hoarder) | **AGPL-3.0** | Links, pages, PDFs, images, text notes; OCR; LLM auto-tagging; full-text + semantic search; RSS/CLI/SingleFile; REST API; **official MCP server**; local + cloud AI providers; single-user self-host first | Email, chat, meeting media; app-level encryption (not documented; could not verify); multi-source capture beyond URLs/files | v0.33.2, 2026-08-11 | **ADOPT-AS-PATTERN** (single-user + MCP delivery shape) |
| **ArchiveBox** | **MIT** | Web/URL preservation into durable static formats (WARC/HTML/PDF/screenshot); CLI + Python API + alpha REST; LDAP/multi-user; **MCP server** (protocol 2025-11-25) | Email, chat, meeting media; semantic search (full-text only); app-level encryption | v0.9.51, 2026-09-23 | **ADOPT-AS-PATTERN** (provenance + durable artifact format) |
| **piler** (MailPiler) | **GPL-3.0-only** | Email only, but feature-rich: built-in SMTP, archival rules, retention, **legal hold**, dedup, compression, message encryption (**weak — see §3.2**), digital fingerprinting and verification, full-text + expert search, tags, audit logs, AD/LDAP, SSO, Google Workspace & Office 365, EML/Maildir/mbox | Chat, meetings, semantic search, MCP/agent API; **crypto is static-IV CBC with no MAC** | piler-1.4.9, 2026-06-17 | **REJECT** as a reference implementation; keep only as a **feature checklist** |
| **Paperless-ngx** | **GPL-3.0** | Scanned/PDF documents, OCR, **IMAP email ingestion** of attachments, Email OAuth, cron-driven mail task, full-text search, REST API, permissions, local-LLM/AI features | Chat, meetings/transcription; **encryption — README states data is stored "in clear text without encryption"**; document-centric not communication-centric | v3.2.1, 2026-09-20 | **REJECT** as a base; **ADOPT-AS-PATTERN** for IMAP+OAuth ingestion |
| **AnythingLLM** | **MIT** | Document RAG (PDF/docs), **built-in audio transcription**, agents, **MCP compatibility**, local + cloud LLMs, desktop single-user app | Email/chat/meeting connectors; app-level encryption; multi-user is Docker-only | v1.16.2, 2026-09-22 | **ADOPT-AS-PATTERN** (bundled local transcription + MCP client) |
| **TriliumNext Trilium** | **AGPL-3.0** | Notes with **per-note encryption** ("protected notes"), ETAPI REST API, self-hosted sync server, single-user-first | Email, chat, meeting media; semantic search; MCP | v0.105.0, 2026-08-19 | **ADOPT-AS-PATTERN** (single-user-first store + selective encryption + API) |
| **Memos** | **MIT** | Short-form notes into a **chronological Markdown timeline**, REST API, SQLite, zero telemetry, single-user friendly | Email, chat, meeting media; encryption; MCP (community only) | v0.31.0, 2026-09-19 | **ADOPT-AS-PATTERN** (timeline UX) |
| **SurfSense** | **Apache-2.0** except `surfsense_backend/app/proprietary/` = **BUSL 1.1** (open core) | Local-first research over your own documents, local models, source-cited answers, MCP server for its hosted scraper API, keys in OS keychain | Email/chat/meeting ingestion; plugins gated behind a paid licence | v2.0.2, 2026-09-21 | **WATCH** (BUSL dir + open-core gating) |
| **Docspell** | **AGPL-3.0** | DMS: unifies scanner + **email** files, ML tagging, full-text search, REST API, multi-user | Chat, meetings; encryption (not documented); stale | v0.43.0, 2025-03-15 | **REJECT** (stale, wrong shape) |
| **Mayan EDMS** | **Apache-2.0** | Heavy document management: workflows, OCR, metadata, REST API, multi-user | Email/chat/meeting ingestion; encryption; far too heavy | no `/releases/latest` redirect (releases page only) | **REJECT** |
| **Reor** | **AGPL-3.0** | Local AI note-taking, semantic search over notes, local LLM | Email/chat/meeting; MCP; encryption | v-0.2.32, 2025-04-05 | **REJECT** (stale, out of scope) |
| **Open WebUI** | **"Open WebUI License"** — source-available, branding-preservation clause (not OSI open source) | Chat UI, RAG knowledge bases, MCP/MCPO/OpenAPI tool servers (**as client**), optional SQLite encryption, LDAP/SSO/SCIM, enterprise tier | Email/chat/meeting ingestion (the 45+-source `oikb` connector project is separate); no MCP *server* for its KB | v0.11.4, 2026-09-21 | **REJECT** (license incompatible with a distributable product) |
| **Logseq** | **AGPL-3.0** | Local-first outliner/notes, plugins, Markdown/org | Any ingestion; encryption; MCP | 2.0.1, 2026-07-13 | **REJECT** |
| **Joplin** | **AGPL-3.0-or-later**, with **per-directory overrides** (`packages/server` carries its own `LICENSE.md`) | Notes, plugin ecosystem, **E2EE for sync**, Web Clipper API | Local DB is **not** encrypted (only sync is E2EE); no ingestion; no MCP | v3.7.18, 2026-09-11 | **REJECT** as a base |
| **Anytype** | **unverified** — no `LICENSE`/`LICENSE.md`/`COPYING` at `anytypeio/anytype-ts` root | Local-first encrypted object store, own sync protocol (`any-sync`) | No ingestion; **license not verifiable** | — | **REJECT** (license unverifiable) |
| **SilverBullet** | **MIT** | Local-first Markdown notes, Lua scripting, self-hosted | Ingestion, encryption, MCP, semantic search | (no release data captured) | **REJECT** |
| **Docmost** | **AGPL-3.0** core + **Docmost Enterprise** license in `packages/ee/` | Team wiki/collaborative docs | Everything footprint needs; it is a team product | v0.96.0, 2026-09-08 | **REJECT** (multi-user team product) |
| **Recoll** | **unverified** (could not fetch `COPYING` from the mirror I tried; conventionally GPL-2.0+) | Desktop full-text index over local files **including Maildir/IMAP**, Xapian-backed | No capture, no encryption, no agent surface; not a store | — | **ADOPT-AS-PATTERN** (local full-text index design) |
| **notmuch** | **GPL-3.0-or-later** | Maildir index + search over Xapian, tagging, CLI + library, fast | Email only; no capture; no encryption; no agent surface | (no release data captured) | **ADOPT-AS-PATTERN** |
| **mu** | **GPL-3.0** | Maildir index + search, CLI | Email only | v1.14.3, 2026-08-09 | **ADOPT-AS-PATTERN** |
| **PrivateGPT** | **Apache-2.0** | Local RAG over local files, local LLMs | Email/chat/meeting; encryption; MCP | v1.0.1, 2026-06-18 | **REJECT** (out of scope) |
| **Quivr** | **Apache-2.0** | RAG framework, multi-user | Everything footprint needs; effectively dormant/renamed (repo redirects to The-Vibe-Company/quivr) | 2025-02-04 | **REJECT** (dormant) |
| **wallabag** | **MIT** | Read-later article archive | Email/chat/meeting; encryption; MCP | 2.6.14, 2025-10-07 | **REJECT** (out of scope) |
| **SiYuan** | **AGPL-3.0** | Local-first notes, E2EE sync | Ingestion; MCP | v3.8.5, 2026-09-22 | **REJECT** |
| **Immich** | **AGPL-3.0** | Photos/video self-host, ML search, mobile backup | Out of scope — listed only as a pattern for local media + ML search + mobile capture | v3.2.2, 2026-09-15 | **WATCH** (pattern only) |

---

## 3. Per-candidate detail

### 3.1 The five focus candidates

#### Onyx (formerly Danswer) — the closest whole-product shape, and still the wrong product

- Repo: https://github.com/onyx-dot-app/onyx — LICENSE: https://github.com/onyx-dot-app/onyx/blob/main/LICENSE
- Docs: https://docs.onyx.app/ — MCP: https://docs.onyx.app/admins/actions/mcp

**License.** The root `LICENSE` is explicit and mixed:

> "All content that resides under 'ee' directories of this repository is licensed under the Onyx
> Enterprise License. Each ee directory contains an identical copy of this license at its root:
> `backend/ee/LICENSE`, `web/src/app/ee/LICENSE`, `web/src/ee/LICENSE` … Content outside of the
> above mentioned directories or restrictions above is available under the 'MIT Expat' license."

So Onyx is **open core**: MIT for the bulk, proprietary for everything under `ee/`. This matters
for footprint because the interesting enterprise-shaped features (RBAC, SSO, some connectors,
multi-tenant admin) live behind that boundary. **I did not enumerate exactly which features are
EE-gated in this pass** — see §5.

**What it covers.** Onyx is the broadest capture surface in this survey. Its docs sitemap lists
**56 official connectors**, and the ones that matter for footprint are all present:

- `connectors/official/gmail/oauth` — **delegated per-user OAuth**, exactly the authorisation
  model footprint requires;
- `connectors/official/gmail/service_account` — **tenant-wide service-account access**, exactly
  the model footprint forbids. The fact that both exist side by side is a useful warning: an
  org-search product will happily let an admin index the whole tenant.
- `connectors/official/slack/slack_indexed` and `slack_federated` — chat ingestion, two modes;
- `connectors/official/discord`, `confluence`, `jira`, `notion`, `linear`, `google_drive`,
  `hubspot`, `zendesk`, `fireflies`;
- `connectors/official/fireflies` — **meeting transcripts**, i.e. the meeting-media leg that
  almost every other candidate lacks.

It also supports local/offline LLMs, has a REST API, and — importantly for Hermes — it is both an
MCP client and an MCP server. From the MCP docs page:

> "You can configure Onyx to be an MCP client and allow your Agents to retrieve data or perform
> operations." … "Onyx can also act as an MCP server! See the Onyx MCP Server guide to connect
> Claude, Cursor, and other AI tools to your Onyx knowledge base."

The MCP **server** is the half worth copying, and it is read-shaped in the sense that matters most
— it mutates nothing. But be precise about what it exposes: its docs list three capabilities,
"Search your knowledge base", "Search the web", "Fetch full page content", and **two of the three
are open-world egress** — arbitrary web search and retrieval of complete text from any URL. So it
is a *non-mutating* surface, not a *confined* one. It also states that "All of your existing Onyx
permissions and access controls are enforced automatically" — which is possible only because Onyx
has per-document ACLs to enforce against (a coarse single `mcp:use` scope plus data-layer
checks). footprint is single-user with no ACLs, so it can borrow the shape but not the enforcement
mechanism. Clients authenticate with a Personal Access Token or API Key in an
`Authorization: Bearer` header, over HTTP (not stdio). The code matches the docs: the server lives
at `backend/onyx/mcp_server/` (`mcp_server_main.py`, `api.py`) and imports a single
`onyx.mcp_server.tools.search` module. **That path is outside `ee/`, so the MCP server is MIT
Expat** — while Onyx's MCP *client*/actions registry (`mcp.json`, `mcp_registry`,
`mcp_servers` under `backend/ee/`) is the proprietary half. So the part footprint wants to study
is legally reusable, and the part behind the paywall is the part footprint does not want.

Deployment has also gotten lighter: `uv tool install onyx-cli && onyx-cli deploy install` offers
**Lite** and **Standard** modes and installs Docker if needed. That is a meaningful improvement
over the old "stand up Postgres + Vespa + Redis + model servers by hand" story.

**What it does not cover, against the footprint bar.**

- **Encryption.** No application-level encryption is documented. Onyx assumes a trusted host.
  On a plain ext4 root with no LUKS, that fails footprint's hard constraint outright.
- **Single-user.** Onyx is a multi-user org product: admin panel, users, permissions, group
  sync. A single-user install is possible but is a degenerate case, not the design centre.
- **Participant-only by construction.** Connector-level scoping exists, but the product's default
  posture is "index the tenant". Footprint's invariant is the opposite.
- **Chat coverage is Western-enterprise.** Slack, Discord, Teams-adjacent. Not WhatsApp, not
  WeChat, not Google Chat in the Malaysian workplace sense (see the workplace-stack memo).

**Verdict: ADOPT-AS-PATTERN, REJECT as a base.** Copy: the connector registry shape, the
delegated-OAuth connector model, the dual MCP client/server design, and the Lite-vs-Standard
deployment split. Do not depend on the code: open-core licence boundary, org-scale footprint,
no encryption.

#### Khoj — best semantic recall over a local LLM, wrong content

- Repo: https://github.com/khoj-ai/khoj — LICENSE: AGPL-3.0 (read from
  https://raw.githubusercontent.com/khoj-ai/khoj/HEAD/LICENSE)
- Docs: https://docs.khoj.dev/

**License: AGPL-3.0.** Strong copyleft with a network clause. For footprint this is a real
constraint: if footprint links Khoj code into a distributed binary, or offers a modified Khoj as
a network service, the AGPL obligations attach. ADOPT-AS-PATTERN sidesteps this entirely.

**What it covers.** The docs sitemap is the honest inventory of what Khoj ingests. Its
`data-sources` section contains exactly three entries — `github_integration`,
`notion_integration`, and `share_your_data` — plus local files. Its `features` section covers
`search`, `chat`, `agents`, `automations`, `online_search`, `voice-chat`, `code_execution`,
`image_generation`. Local model support is documented under `advanced/ollama`,
`advanced/lmstudio`, `advanced/litellm`, `advanced/use-openai-proxy`. Clients: desktop, web,
Obsidian, Emacs, WhatsApp.

That gives Khoj a genuinely good **semantic search + local-LLM + query-filter** story — the part
of footprint that is hardest to build well — plus a documented REST API for an external agent.

**What it does not cover.**

- **Email: no.** There is no IMAP/Gmail data source in the docs sitemap. Khoj's "email" surface
  is transactional (magic-link login), not archival.
- **Chat: no.** The WhatsApp *client* lets you talk to Khoj; it does not ingest your WhatsApp
  history.
- **Meeting media: no.** `features/voice-chat` is speech-to-text for *input*, not meeting
  transcription of a recording.
- **Encryption: no** app-level encryption documented.
- **MCP: not found.** No MCP page in the docs sitemap, and `mcp/README.md` /
  `src/khoj/mcp/__init__.py` are both 404. That is *absence of evidence*, not proof — I could not
  fully rule out a third-party MCP server. Khoj's own agent surface is its REST API.
- **Release cadence is odd:** the repo was pushed 2026-08-02 but the latest *release* is
  `2.0.0-beta.28` dated 2026-03-26. Self-hosters are running beta tags.

**Verdict: ADOPT-AS-PATTERN.** The semantic index + local-LLM + query-filter design is worth
copying closely. AGPL-3.0 rules out depending on the code in a distributable product.

#### Karakeep (formerly Hoarder) — the right delivery shape, the wrong content

- Repo: https://github.com/karakeep-app/karakeep — LICENSE: AGPL-3.0
- Docs: https://docs.karakeep.app/ — MCP: https://docs.karakeep.app/next/integrations/mcp/

**License: AGPL-3.0**, owned by Localhost Labs Ltd (per README). Same distribution caveat as
Khoj. There is also a paid managed cloud (`cloud.karakeep.app`) — the classic self-host-first
project with a hosted tier, but the OSS edition is not feature-crippled in the parts we care
about.

**What it covers.** Karakeep is the closest thing in this survey to footprint's *delivery
shape*: a **single-user-first self-hosted app** with an **official MCP server**, a fully
documented REST API, OCR, LLM auto-tagging, full-text plus semantic search, and pluggable AI
providers including local models. Its docs integrations are exactly the set an agent-facing
product needs: `command-line`, `mcp`, `rss-feeds`, `singlefile`, `agentic-skills`.

The MCP docs page is explicit about both the surface and the direction:

> "Karakeep comes with a Model Context Protocol server that can allow LLMs (like claude and codex)
> to interact with your karakeep data."

with tools `search-bookmarks`, `get-bookmark`, `get-bookmark-content` (read) and
`create-*` (write), shipped as `ghcr.io/karakeep-app/karakeep-mcp:latest`. Note the read/write
mix: like ArchiveBox, the MCP surface is not read-only by default. Footprint would need to
restrict the exposed tool set.

**What it does not cover.**

- **Content type.** Bookmarks, links, pages, PDFs, images, notes. No email, no chat, no meeting
  media. Karakeep's own README explains it grew out of a Pocket replacement use case.
- **Encryption.** No app-level encryption is documented. I tried to verify against
  `docs/administration/security-considerations` but the page body is client-rendered and the
  markdown source path guesses all 404'd. **Treat Karakeep's encryption posture as unverified;
  the safe assumption is that it relies on the host.**

**Verdict: ADOPT-AS-PATTERN.** Copy the packaging and the agent surface — single-user self-host,
official MCP server as a first-class artifact, documented REST API, local-model providers. Not
the content model, and not the licence if you intend to link.

#### ArchiveBox — provenance and durability, and a cautionary MCP story

- Repo: https://github.com/ArchiveBox/ArchiveBox — LICENSE: **MIT**
- Docs: https://docs.archivebox.io/ — MCP module: `archivebox/mcp/`

**License: MIT.** This is the most permissive licence among the serious candidates, and it is the
only MIT candidate with both a durable on-disk artifact format *and* an MCP server. If footprint
needs to reuse code rather than copy a design, this is the one candidate where that is legally
trivial.

**What it covers.** ArchiveBox preserves web content into static, replayable formats and states
its design goal as the archive remaining viewable "in 50 – 100 years without needing to run
ArchiveBox". That is directly relevant to footprint's evidence constraint: **append-only,
verifiable, replayable artifacts** are the same instinct. Its interfaces are CLI, Python API,
alpha REST, a browser extension, and a desktop app; it also supports LDAP and multiple users.

It now ships an MCP server. The module docstring is precise:

> "Model Context Protocol (MCP) server for ArchiveBox. … Provides a JSON-RPC 2.0 interface over
> stdio for AI agents to control ArchiveBox."

and the server source declares:

```python
PROTOCOL_VERSION = "2025-11-25"
PUBLIC_TOOLS = ("add", "search", "crawl", "snapshot", "archiveresult", "shell")
ACTION_TOOLS = {"crawl", "snapshot", "archiveresult"}
READ_ONLY_ACTIONS = {"help", "list", "search", "status", "version"}
DESTRUCTIVE_ACTIONS = {"delete", "remove"}
```

**This is the single most useful negative finding in the survey.** ArchiveBox has the vocabulary
for read-only vs destructive MCP actions, and still exposes `add`, `crawl`, `snapshot` and
`shell` to the agent. An agent that can call `shell` against an archive is not a read-only
surface. Footprint's Hermes surface must be built as an explicit allow-list of read tools, and
ArchiveBox demonstrates how easily that boundary erodes when you generate tools from a CLI.

**What it does not cover.**

- Email, chat, meeting media — none. It is a web archiver.
- **Semantic search** — it has a full-text search module (`archivebox.search`) but no vector
  layer, so it cannot answer "find the meeting where we discussed the same thing as this email".
- **Encryption** — none documented; the archive is a plain directory tree by design (that is the
  point of the durability goal). On a non-LUKS host this fails footprint's constraint.

**Verdict: ADOPT-AS-PATTERN.** Copy the durable-artifact/static-export philosophy, the
"archive must outlive the tool" test, and the read-only-action taxonomy — then implement the
taxonomy more strictly than ArchiveBox does.

#### Paperless-ngx — reject, and the README tells you why

- Repo: https://github.com/paperless-ngx/paperless-ngx — LICENSE: **GPL-3.0**
- Docs: https://docs.paperless-ngx.com/

**License: GPL-3.0.** Copyleft; a linking/distribution constraint.

**What it covers.** Paperless-ngx is a strong document management system with a genuinely useful
ingestion path for footprint's email leg. Its configuration reference documents a full IMAP
consumer: `PAPERLESS_EMAIL_HOST`, `PAPERLESS_EMAIL_PORT`, `PAPERLESS_EMAIL_HOST_USER`,
`PAPERLESS_EMAIL_HOST_PASSWORD`, `PAPERLESS_EMAIL_USE_TLS`, `PAPERLESS_EMAIL_USE_SSL`,
`PAPERLESS_EMAIL_TASK_CRON`, plus a dedicated **"Email OAuth"** section and
`PAPERLESS_EMAIL_CERTIFICATE_LOCATION` for self-signed local IMAP servers. It also has
`PAPERLESS_EMAIL_GNUPG_HOME` (PGP-related handling of mail), OCR, full-text search, a REST API,
and per-user permissions.

**Why it fails the bar.** The project says so itself, in its own README:

> "**Paperless-ngx should never be run on an untrusted host** because information is stored in
> clear text without encryption. No guarantees are made regarding security (but we do try!) and
> you use the app at your own risk."

That is a direct, primary-source statement that the store is unencrypted. Combined with GPL-3.0
and a document-centric (not communication-centric) data model — no chat, no meeting media, no
transcription — Paperless-ngx cannot be footprint's base.

**Verdict: REJECT as a base; ADOPT-AS-PATTERN for the IMAP + delegated-OAuth ingestion design**
and for the honesty of its security documentation. That README sentence is a model of how to
state a limitation plainly.

### 3.2 The rest, in brief

**piler (MailPiler)** — **GPL-3.0-only** — https://github.com/jsuto/piler, https://www.mailpiler.org/.
The strongest *email-archival* reference in the survey. Its README feature list reads like a
specification for footprint's email leg: built-in SMTP, archival rules, retention rules,
**legal hold**, message and attachment **deduplication**, message compression, **message
encryption**, **digital fingerprinting and verification**, full-text and expert search, saved
searches, tagging, view/export/restore, bulk import/export, access control, AD/LDAP, IMAP/POP3
auth, SSO, Google Workspace and Office 365 support, 2FA, **audit logs** and search within them,
and recognised formats EML/Maildir/mbox. "Digital fingerprinting and verification" plus "legal
hold" plus "audit logs" is precisely the evidence-and-append-only vocabulary footprint needs.
**Correction (resolved by oss-encryption-store, 2026-09-25).** I originally flagged piler's
encryption as unverified because its wiki page 404s. That question is now closed, and the answer
is a downgrade. The features are in the **OSS** tree (not gated behind Enterprise):
`src/cfg.c` declares the `encrypt_messages` and `iv` config keys and the encrypt/decrypt path
lives in `src/archive.c`. But the crypto is weak:

- `src/archive.c` selects between `EVP_aes_256_cbc()` for newer records and `EVP_bf_cbc()`
  (**Blowfish-CBC**) for older ones — a legacy cipher;
- the key and IV come straight from configuration (`src/cfg.h` has `unsigned char key[KEYLEN]`
  and `unsigned char iv[MAXVAL]`; `src/cfg.c` parses a plain `iv` string) — i.e. a **static IV**,
  so encryption is deterministic and shared prefixes leak;
- CBC **with no MAC**, so there is **no tamper detection**.

That last point is disqualifying for footprint: an archive whose integrity cannot be verified
contradicts the append-only / verifiable-artifact requirement outright. Piler's "digital
fingerprinting and verification" is not a substitute for authenticated encryption.

Also on licence: piler's `LICENSE` says "version 3 of the License" — **GPL-3.0-only**, not
"or later". GitHub reports `NOASSERTION` only because of the custom preamble.

**Verdict: REJECT as a reference implementation.** Keep piler as a **feature checklist** — legal
hold, retention rules, deduplication, audit-log search and fingerprinting are all things
footprint's evidence model will want; its *cryptography* is the part not to copy.

**AnythingLLM** — MIT — https://github.com/Mintplex-Labs/anything-llm,
https://docs.anythingllm.com/. MIT, MCP-compatible, agents, local and cloud LLMs, and notably
**built-in audio/video transcription** ("AnythingLLM Built-in (default)"). Multi-user support is
explicitly *Docker only*; the desktop app is the single-user path. It does not ingest email, chat
or meeting platforms — you feed it documents. **ADOPT-AS-PATTERN** for bundling a local
transcription model into a desktop single-user app, and for MCP client integration. The MIT
licence makes it the most reusable code of the "AI app" candidates.

**TriliumNext Trilium** — AGPL-3.0 — https://github.com/TriliumNext/Trilium. Single-user-first
personal knowledge base with **per-note encryption** ("protected notes" with per-note
granularity, per its README), a self-hosted sync server, and the ETAPI REST API. The per-item
encryption model is worth studying for footprint's selective-sensitivity story (PDPA excludes
sensitive personal data). No ingestion of communications, no semantic search, no MCP.
**ADOPT-AS-PATTERN.**

**Memos** — MIT — https://github.com/usememos/memos. The README positions it as "an open-source,
self-hosted home for short-form thinking… flow into a chronological Markdown timeline", with
"zero telemetry" and an MIT licence. That is footprint's *timeline* requirement in its simplest
form: a chronological spine over a plain local store. No ingestion, no encryption, no MCP.
**ADOPT-AS-PATTERN** for the timeline UX and for the "keep control / no telemetry" posture.

**SurfSense** — https://github.com/MODSetter/SurfSense. **Read the LICENSE carefully**: it is
Apache-2.0 *except* `surfsense_backend/app/proprietary/`, which is **Business Source License
1.1**. The README also says "A licence adds plugins and priority support and gates nothing else".
So: open core, with a BUSL island in the backend. Its local-first framing and OS-keychain secret
storage are appealing, but a BUSL directory in the core backend is a distribution hazard.
**WATCH.**

**Open WebUI** — https://github.com/open-webui/open-webui. The README is explicit that the
current codebase includes components "licensed under the Open WebUI License with an additional
requirement to preserve the 'Open WebUI' branding". That is **source-available, not OSI open
source**, and it is a branding/redistribution constraint. It also has an Enterprise tier, and its
45+-source knowledge-base connectors live in a *separate* project (`open-webui/oikb`). Its MCP
support is client-side (MCP/MCPO/OpenAPI tool servers); it consumes MCP servers, it does not
expose its knowledge base as one. **REJECT** on licence grounds for a distributable product —
and it is a good reminder to check the LICENSE file rather than the badge.

**Docspell** — AGPL-3.0 — https://github.com/eikek/docspell. A DMS that "can unify your files
from scanners, emails" with ML tagging and full-text search. Last release v0.43.0 (2025-03-15) —
stale. Email ingestion without email-archive rigour, no chat/meetings, no documented encryption,
AGPL. **REJECT** (stale, wrong shape).

**Mayan EDMS** — Apache-2.0 — https://gitlab.com/mayan-edms/mayan-edms. A heavyweight DMS with
workflows, OCR and metadata; the repo has no `/releases/latest` redirect (releases page only).
Nothing about it addresses participant-only capture, encryption, or an agent surface. **REJECT.**

**Reor** — AGPL-3.0 — https://github.com/reorproject/reor. Local AI note-taking with semantic
search over notes. Last release v-0.2.32 (2025-04-05) — stale. **REJECT.**

**Logseq** — AGPL-3.0 — https://github.com/logseq/logseq. Local-first outliner. No ingestion,
no encryption, no agent surface. **REJECT.**

**Joplin** — https://github.com/laurent22/joplin. The root LICENSE is
**AGPL-3.0-or-later**, but with an important wrinkle stated in the file itself:

> "All code in this repository is licensed under the AGPL-3.0-or-later License **unless a
> directory contains a LICENSE or LICENSE.md file**, in which case that file applies to the code
> in that sub-directory. For example, packages/server contains a LICENSE.md file…"

So Joplin is not uniformly AGPL — per-directory licences apply, and **I did not verify what
`packages/server`'s licence is**. Also: Joplin's E2EE applies to *sync*, not to the local
database, which is the opposite of footprint's requirement (encrypt the local store; the host may
have no LUKS). **REJECT as a base**; the sync-E2EE design is not what footprint needs.

**Anytype** — https://github.com/anytypeio/anytype-ts. I could **not find a LICENSE file** at the
repository root (LICENSE, LICENSE.md, LICENSE.txt, COPYING all 404). Its architecture — a
local-first, encrypted object store with its own sync protocol (`any-sync`) — is genuinely
interesting for footprint, but a candidate whose licence cannot be verified from primary sources
cannot be adopted or depended on. **REJECT (license unverifiable)**; revisit only if someone can
point at the actual licence text.

**SilverBullet** — MIT — https://github.com/silverbulletmd/silverbullet. Local-first Markdown
notes with Lua scripting. MIT is friendly, but there is no ingestion, encryption or agent
surface. **REJECT** (pattern value only: a single-binary, no-sudo, local-first deployment story).

**Docmost** — AGPL-3.0 core + Docmost Enterprise license in `packages/ee/` — team wiki.
**REJECT** — multi-user team product, opposite of footprint's premise.

**Recoll** — desktop full-text search over local files including Maildir/IMAP, Xapian-backed.
I could not fetch its `COPYING` (the GitHub mirror I tried 404s; the canonical repo is on
Framagit), so **its licence is unverified here** — conventionally GPL-2.0+. **ADOPT-AS-PATTERN**
for the "index heterogeneous local sources into one queryable full-text index" design, which is
exactly footprint's search layer minus the vector half.

**notmuch** — **GPL-3.0-or-later** (read from `COPYING`) — Maildir indexing and search over
Xapian, tag-based, fast, with a library and CLI. Email-only, no encryption, no agent surface.
**ADOPT-AS-PATTERN** for the tag-plus-query model and for its speed on large mailboxes.

**mu** — GPL-3.0 — https://github.com/djcb/mu. Same niche as notmuch (Maildir index + search),
last release v1.14.3 (2026-08-09). **ADOPT-AS-PATTERN.**

**PrivateGPT** — Apache-2.0 — local RAG over local files with local LLMs. No email/chat/meeting
ingestion, no encryption, no MCP. **REJECT** (out of scope).

**Quivr** — Apache-2.0, but the repo now redirects to `The-Vibe-Company/quivr` and the last
release was 2025-02-04. **REJECT** (dormant/renamed).

**wallabag** — MIT — read-later article archive, last release 2.6.14 (2025-10-07). **REJECT**
(out of scope).

**SiYuan** — AGPL-3.0 — local-first notes with E2EE sync. **REJECT** (out of scope).

**Immich** — AGPL-3.0 — self-hosted photos/video with ML search and mobile background backup.
Out of scope as a base, but worth noting as the best-in-class pattern for "local media store +
ML search + mobile capture client", which is roughly the shape footprint's meeting-recording leg
needs. **WATCH** (pattern only).

---

## 4. Closest fits and the gap

**There is no single project to adopt.** Ranked by how much of the bar they clear:

1. **Onyx** clears the *capture* bar best (delegated Gmail OAuth, Slack, Discord, Fireflies
   meeting transcripts, 56 connectors) and now clears the *agent surface* bar (MCP server). It
   fails **encryption** (none), **single-user** (org product), and **participant-only by
   construction** (it also ships tenant-wide service-account Gmail).
   *Gap it leaves:* it is the wrong kind of product — an org knowledge search, not a personal
   archive — and it cannot satisfy the encrypted-at-rest constraint on a non-LUKS host.

2. **Karakeep** clears the *delivery shape* bar best: single-user-first, self-hosted, official
   MCP server, REST API, local-model providers, OCR, timeline-ish browsing. It fails on
   **content** (bookmarks/links only), **encryption** (undocumented/unverified), and its MCP
   surface includes write tools.
   *Gap it leaves:* nothing to capture email, chat or meeting media into.

3. **piler** has the best *email feature checklist*: legal hold, retention, dedup,
   fingerprinting, audit-log search. But its encryption is static-IV CBC with no MAC (see §3.2),
   so it **no longer counts as clearing the email-rigour bar** — it is a feature checklist, not a
   reference implementation. It fails everything else too: email only, no semantic search, no
   agent surface, GPL-3.0-only.
   *Gap it leaves:* it is a mail archiver, not a personal recall product — and its integrity model
   is not one to copy.

4. **ArchiveBox** clears the *provenance* bar best: MIT, durable static artifacts designed to
   outlive the tool, plus an MCP server. It fails **semantic search**, **encryption**, and
   **content breadth** (web only), and its MCP server exposes destructive and `shell` tools.
   *Gap it leaves:* it preserves pages, not communications, and its agent surface is not
   read-only.

5. **Khoj** clears the *semantic recall over a local LLM* bar best. It fails **all ingestion**
   (no email, chat or meeting media), **encryption**, and **MCP** (none found).
   *Gap it leaves:* it can answer questions over documents you hand it, but it cannot capture
   anything.

**The composite gap, stated once.** No candidate combines (a) participant-scoped capture across
email + chat + meeting media, (b) an application-level encrypted single-user store, and (c) a
read-only agent surface. (a) exists only in multi-user org products with no encryption; (b)
exists only in single-content-type tools (piler for mail, Trilium for notes); (c) exists in
projects whose stores are unencrypted and whose MCP surfaces are not read-only. **Footprint's
capture layer and encrypted store must be built.** The five patterns above are the parts worth
copying.

A secondary, sharper observation for whoever designs the Hermes surface: **two of the three
projects that ship an MCP server ship a write-capable one** (Karakeep with `create-*`,
ArchiveBox with `add`/`crawl`/`snapshot`/`shell`). Footprint's read-only requirement is
therefore *not* the industry default, and should be enforced by an explicit tool allow-list plus
a read-only database role — not by convention.

**One candidate is meaningfully better than the others, and it is the closest thing to a
precedent — but with an important qualification.** Onyx's MCP server is non-mutating: its docs
describe exactly three capabilities ("Search your knowledge base", "Search the web", "Fetch full
page content"), authentication is a Personal Access Token or API Key as a bearer header, transport
is HTTP, and the implementation confirms a small fixed surface —
`backend/onyx/mcp_server/api.py` imports a single `onyx.mcp_server.tools.search` module plus an
`indexed_sources` resource module, rather than reflecting a CLI or ORM (its
`tools/__init__.py` contains nothing but that one import). It is MIT-licensed code, outside
`ee/`, so it can be read and copied freely.

**The qualification: "read-only" is not the same as "confined".** Two of Onyx's three tools are
open-world egress — arbitrary web search, and retrieval of complete text from any URL. A consumer
of that server gets a general internet fetch, not merely a query over the local corpus. For
footprint this is the wrong default on two counts: it is an unbounded data-egress path, and it is
irrelevant to a participant-only archive whose whole premise is that the corpus is closed. So
Onyx is the precedent for *shape* (a small fixed tool set, per-consumer credentials so revocation
is one operation, no CLI/ORM reflection) but not for *scope* — footprint's Hermes surface should be
strictly confined to the local corpus, with no web-search or arbitrary-URL tool at all.

**And Onyx can push authorisation into the data layer only because it has per-document ACLs to
enforce.** Its token verifier delegates to the Onyx API's `/me` endpoint and returns a coarse
single `mcp:use` scope with no read/write split; the real enforcement happens against document
permissions. footprint is single-user and has no ACLs, so it **cannot borrow the enforcement
mechanism — only the shape**. There is a concrete file path for this split: **gmail appears in
both of Onyx's trees** — `backend/onyx/connectors/gmail/connector.py` is MIT, while
`backend/ee/onyx/external_permissions/gmail/` is Enterprise-licensed (alongside per-connector sync
for slack, teams, outlook, sharepoint, confluence, github, google_drive, box, canvas, jira and
salesforce, plus `post_query_censoring.py`). **Ingest is MIT; the enforcement layered on top of it
is not.** That is precisely why the two controls are complementary rather than
alternatives: per-consumer credentials scope *which consumer* is asking (borrowed from Onyx, and
from Karakeep's bearer key), while an explicit tool allow-list plus a read-only database role
scope *which operations* are possible (which footprint must supply itself, because it has no ACL
layer to lean on).

---

## 5. Open questions and things I could not verify

**Licence items I could not close:**

- **Anytype (`anytypeio/anytype-ts`)**: no `LICENSE`, `LICENSE.md`, `LICENSE.txt` or
  `COPYING` at the repository root (all 404). Its licence is **unverified**. Do not depend on it.
- **Recoll**: could not fetch `COPYING` from the GitHub mirror I tried (404); the canonical repo
  is on Framagit. **Unverified**; conventionally GPL-2.0+.
- **Joplin**: root is AGPL-3.0-or-later, but the file itself declares **per-directory overrides**
  and names `packages/server` as having its own `LICENSE.md`. I did not read that file, so
  Joplin's server-side licence is **unverified**.
- **Onyx EE boundary — the MCP question is RESOLVED; the rest is not.** I confirmed the *rule*
  (everything under `ee/` is Onyx Enterprise License). oss-agent-interface then raised the sharp
  follow-up: is the MCP *server* code inside `ee/`? It is **not**. Verified from the repository:
  `backend/onyx/mcp_server_main.py` and `backend/onyx/mcp_server/api.py` both exist outside any
  `ee/` directory, and `api.py` imports `from onyx.mcp_server.tools import search` — so **the Onyx
  MCP server is MIT Expat**, and an adopter can consume it without touching the proprietary
  boundary. The MCP *client* / actions registry, by contrast, does sit in `backend/ee/`
  (`mcp.json`, `mcp_registry`, `mcp_servers`), so Onyx's ability to *call* external MCP servers
  is the Enterprise-licensed half.
  **Connector gating — now checked, for the footprint-relevant set (2026-09-25).** Two distinct
  methods were used, and they must not be conflated (this section was corrected on 2026-09-25
  after oss-agent-interface challenged the method):
  - **File-level** (valid): I probed each connector's source path in both trees —
    `backend/onyx/connectors/<name>/connector.py` versus
    `backend/ee/onyx/connectors/<name>/connector.py` — via HTTP status on
    raw.githubusercontent, which serves files and is the right tool for this.
  - **Directory-level**: I used the `github.com/.../tree/main/<path>` HTML page, **not**
    raw.githubusercontent and not the contents API. An earlier version of this section said
    "via HTTP status on raw.githubusercontent" immediately before the directory claims, which
    implied a method that cannot work — raw.githubusercontent serves files, not directory
    listings, so a directory path 404s whether or not the directory exists. The tree page *does*
    discriminate: I verified with a bogus-path control that a real directory returns 200 and a
    nonexistent one returns 404, in both the OSS and EE trees.
  - **Better method for directory listings** (oss-agent-interface's suggestion): the git trees API
    — `api.github.com/repos/<owner>/<repo>/git/trees/main:<path>` — returns name/type/size and
    distinguishes real from bogus paths. **Caveat on the API, corrected 2026-09-25 after
    oss-agent-interface pushed back:** the unauthenticated ceiling is **60 requests/hour *per
    source IP***, and the sandbox's egress address is **not stable** — I observed an error naming
    IPv4 `180.75.252.103` and, in a later check, an egress IPv6 of `2001:e68:5458:...`. So quota
    availability varies between runs and between teammates' fetch paths: I hit `remaining: 0, used:
    60` (with five consecutive 403s) at one point, while oss-agent-interface made ~40 calls with
    zero 403s on their path. **Neither "the API is rate-limited here" nor "the API works here" is
    the right generalisation.** The accurate practice is: the API works, but check
    `api.github.com/rate_limit` before a large batch, and expect bursts to 403 even with quota
    left (GitHub's secondary/abuse limiter). Do not trust the `used` counter to the digit — it
    read stale in one of our fetches.
    For that reason I could not reproduce their directory listings via the API myself and instead
    corroborated them from the tree-page payload.
  **All fifteen tested connectors are OSS/MIT — every one returns 200 in the MIT tree and 404 in
  `ee/`:** gmail, outlook, imap, slack, teams, zoom, fireflies, discord, google_drive,
  confluence, jira, notion, github, sharepoint, salesforce. That covers every capture path
  footprint cares about — **email (Gmail, Outlook, IMAP), chat (Slack, Teams, Discord) and meeting
  transcripts (Zoom, Fireflies) are all MIT-licensed code.**
  **Caveat 1 is WITHDRAWN.** I had written that because `backend/ee/onyx/connectors/` exists,
  "Onyx clearly ships *some* EE-only connectors — I simply did not identify which". That was an
  unwarranted inference: directory existence says nothing about contents. oss-agent-interface
  listed the directory via the git trees API and it contains **no connectors at all** — exactly
  three gating helpers (`capability_applicability.py`, `capability_checks.py`,
  `perm_sync_valid.py`), with `"truncated": false`. I corroborated this from the tree-page
  payload: the helper filenames appear while `gmail`, `fireflies` and `zoom` do not appear at all.
  So the correct statement is **"the EE connector directory exists but contains only
  capability/permission gating helpers"** — not "Onyx has EE-only connectors". I make no claim
  either way about whether EE-only connectors exist elsewhere.
  **Caveat 2 is CONFIRMED and sharpened.** `backend/ee/onyx/external_permissions/` exists and
  contains **per-connector permission-sync implementations** — visible in the listing: box,
  canvas, confluence, github, **gmail**, google_drive, jira, outlook, salesforce, sharepoint,
  slack, teams — alongside `perm_sync_types.py`, `sync_params.py` and `post_query_censoring.py`.
  So it is not merely that "permission sync is partly EE": the per-connector sync for the very
  connectors footprint cares about is Enterprise-licensed, and `post_query_censoring.py` suggests
  even result-level filtering is EE. **gmail therefore appears in both trees** —
  `backend/onyx/connectors/gmail/connector.py` is MIT while
  `backend/ee/onyx/external_permissions/gmail/` is not. That is the file-path version of the
  earlier claim that Onyx's authorisation model is a shape to copy rather than a mechanism to
  inherit: **ingest is MIT, enforcement on top of it is Enterprise.** For footprint the split
  falls the favourable way, since a single-user archive has no per-document ACLs to sync.
- **piler Enterprise vs OSS — RESOLVED (2026-09-25).** oss-encryption-store verified from
  source that the features I cited (`encrypt_messages`, `iv` config keys in `src/cfg.c`; the
  encrypt/decrypt path in `src/archive.c`) are in the **OSS** tree, not gated behind Enterprise.
  The same pass found the crypto weak — static IV, CBC without a MAC, legacy Blowfish for older
  records — which is why piler is now **REJECT as a reference implementation**. See §3.2.

**Technical items I could not verify:**

- **Karakeep encryption**: the docs security page is client-rendered and the markdown source
  paths I guessed 404'd. No app-level encryption is *documented*, but I cannot state definitively
  that none exists.
- **Khoj MCP — corroborated negative.** No MCP page in the docs sitemap, and the two repo paths I
  guessed 404'd. oss-agent-interface reached the same conclusion **independently**, and additionally
  found that `docs.khoj.dev/features/agents/` does not mention MCP (checked 2026-09-25) and that an
  open issue (#1364, 2026-07-01) is a user requesting exactly that endpoint. Two independent surveys
  agree: **no Khoj MCP server found.** That is still absence of evidence rather than proof, but it
  is stronger than one survey's silence.
- **piler encryption algorithm / key management — RESOLVED (2026-09-25).** Originally
  unverified because `/wiki/current:encryption` 404s. oss-encryption-store read the source:
  `EVP_aes_256_cbc()` for newer records and `EVP_bf_cbc()` (Blowfish-CBC) for older ones, with key
  and IV taken from configuration (`src/cfg.h`, `src/cfg.c`) — a **static IV** and **no MAC**.
  Deterministic encryption with no tamper detection. See §3.2 for why that disqualifies it.
- **Mayan EDMS**: no `/releases/latest` redirect, so its release cadence is unverified; the
  project is primarily hosted on GitLab.
- **The "encrypted store + separate independently verifiable manifest" split is implemented by
  nobody in this survey — but the *shape* has a verified precedent.** oss-encryption-store raised
  this as the sharpest open design question (an archive meant to be readable without the tool is in
  tension with a store that is opaque without the key). Cross-survey findings, with each item's
  evidential status marked:
  - **restic (BSD-2-Clause) — verified negative, and the anti-pattern to name in the ADR.**
    oss-encryption-store confirmed my reading against `doc/design.rst` with line references:
    lines 50-61 (every file except those under `keys/` is AES-256-CTR with a Poly1305-AES MAC,
    `IV || CIPHERTEXT || MAC`, random IV per file, 32-byte overhead), line 64 (the config file is
    encrypted this way), line 228 ("Individual files for the index, locks or snapshots are
    encrypted"), and lines 22/33/42-43 (a storage ID is "the SHA-256 hash of the content stored
    in" the file — i.e. of the **encrypted** bytes). The sharpest statement of the failure is
    theirs, and it is the sentence for the ADR: it is not merely that verification requires the
    key, but that **the identifier itself is meaningless without it** — the evidence cannot be
    checked by anyone who lacks the very secret the evidence is supposed to outlive.
  - **WARC + a separate plaintext index (CDX/CDXJ) — VERIFIED structural precedent, unencrypted.**
    I originally offered this as a lead because I could not fetch the spec; oss-encryption-store
    then verified it against the WARC 1.1 specification. `WARC-Block-Digest` is "an optional
    parameter indicating the algorithm name and calculated value of a digest applied to the full
    block of the record", formatted `algorithm:digest-value` (the spec's own example is
    `sha1:AB2CD3EF4GH5IJ6KL7MN8OPQ`), and the spec explicitly says "No particular algorithm is
    recommended" — so the format is **algorithm-agile by design**, which matters for a long-lived
    archive because algorithm migration is anticipated rather than bolted on.
    `WARC-Payload-Digest` is a separate payload digest, "not necessarily equivalent to the record
    block", and CDX/CDXJ is the separate plaintext index. So: plaintext tool-independent container
    + per-record algorithm-agile digests + separate index. **The caveat stands exactly as framed —
    nobody encrypts the container**, so footprint would adopt the shape and add the encryption.
    Note the attribution: this is verified by oss-encryption-store, not by me; my own fetch of the
    spec failed, which is why the item moved from lead to finding on their evidence.
  - **Maildir + a separate rebuildable index (notmuch/mu) — same split, no crypto.** The
    transferable property: the index is derivable from the store alone and disposable, so it can
    never become a second source of truth that drifts.
  - **Mailpile — DO NOT CITE.** The one project I know of that paired an encrypted mail store with
    a searchable index. Unverifiable: COPYING 404s and there is no `/releases/latest`.
  - **External timestamping — greenfield.** No candidate integrates RFC 3161 or OpenTimestamps in
    any form; there is no precedent to borrow.
  - **The reason I would not adopt the WARC shape uncritically, and the direction now proposed:**
    an unencrypted per-record manifest is itself a metadata leak and a hash is a **confirmation
    oracle** — it reveals record count, sizes and timestamps, and lets anyone who can *guess* a
    record's content hash the guess and check membership, without the key. For a participant-only
    archive of emails and meeting transcripts under PDPA that is a real disclosure, not a
    theoretical one. I framed this as a three-way choice (unkeyed plaintext digests / digests keyed
    by a separate manifest key / digests over the ciphertext). **oss-encryption-store found the
    prior art that dissolves the choice, and I independently spot-checked their citations:**

    - **Transparency logs are the missing structure.** RFC 6962 (Certificate Transparency) defines
      "publicly auditable, append-only, untrusted" logs built on Merkle Trees (§3.4) with a Signed
      Tree Head (§3.5) and independent Monitors (§5.3); I confirmed those section headings and that
      sentence verbatim in the RFC. Google's Trillian generalises this to "transparent, append-only
      logging of arbitrary data", and — the decisive detail — **the application chooses the leaf
      contents**: "The default Merkle hash for a Trillian Log leaf is `SHA-256(0x00 | leaf.LeafValue)`"
      (confirmed on Trillian's Transparent Logging page). So hashing the **ciphertext** into the
      leaf gives key-free public verifiability with no plaintext oracle — my option 3 — *plus* the
      append-only proof structure (inclusion and consistency proofs against a signed tree head)
      that option 1 was implicitly borrowing from WARC.
    - **The oracle is a named, published attack class**, so the ADR can cite it rather than assert
      it: Halevi, Harnik, Pinkas and Shulman-Peleg, "Proofs of Ownership in Remote Storage
      Systems" (CCS 2011, eprint.iacr.org/2011/207) — "an attacker who knows the hash signature of
      a file can convince the storage service that it owns that file", gaining "access to
      potentially huge files of other users based on a very small amount of side information". I
      confirmed both quotations on the eprint abstract page.
    - **Proposed resolution (a proposal, not a finding):** combine ciphertext-digest leaves in a
      signed Merkle tree with an *optional* keyed HMAC over the plaintext stored under a manifest
      key escrowed **separately** from the data key. That yields three separable capabilities:
      verify the archive is intact and append-only (no key at all); verify a record decrypts to the
      expected plaintext (manifest key); read the content (data key). The capability separation is
      the actual design decision — option 2 is only an improvement if the manifest key genuinely
      lives apart from the data key, otherwise it is just a slower oracle.
    - **One caveat I raised, which the ADR must handle:** a signed tree head is only as trustworthy
      as the independence of its signer. CT works because many mutually-distrusting parties run
      monitors and gossip about what they see; a single-user local archive has no monitors, and the
      same key that writes the archive can rewrite history and re-sign a fresh tree head. So the
      Merkle structure gives *efficient proofs relative to a tree head*, while something external
      must **pin** that tree head before "append-only" becomes a guarantee rather than a claim.
      This is why the external-timestamping gap below is load-bearing rather than optional: it is
      what makes the append-only property meaningful against the archive's own writer — including
      the archive's owner, who in a workplace dispute is exactly the party whose evidence has to
      be credible.
      **Which pin, verified (oss-encryption-store, spot-checked by me):** the two candidates have
      different trust models, so this is not a coin flip. RFC 3161 line 43 states "A TSA may be
      operated as a Trusted Third Party" — one trusted party substituted for another. OpenTimestamps
      instead describes "a set of operations for creating provable timestamps and later independently
      verifying them", Bitcoin-anchored: "Anyone could realize a timestamp with the permissionless
      blockchain by paying the transaction fees", with calendar servers that are "free to use" and
      need "no registration or api key" — so the calendars are an *availability* dependency, not a
      trust dependency.
      **Correction (oss-encryption-store, spot-checked by me): this is a trade, not a clear win for
      OpenTimestamps.** I first recorded OTS as "the better-aligned default"; that was overstated.
      RFC 3161 has **protocol-level long-term-validation semantics that OTS has no analogue for**.
      I confirmed all three passages in the RFC: its stated purpose includes "An example of how to
      prove that a digital signature was generated during the validity period of a public key
      certificate" (line ~51); "The TSA's role is to time-stamp a datum to establish evidence
      indicating that a datum existed before a particular time. This can then be used, for example,
      to verify that a digital signature was applied to a message before the corresponding
      certificate was revoked thus allowing a revoked public key certificate to be used for
      verifying signatures created prior to the time of revocation" (lines 73-77); and its
      six-step verification procedure requires that "The date/time indicated by the TSA MUST be
      within the validity period of the signer's certificate" and that "Should the certificate be
      revoked, then the date/time of revocation shall be later than the date/time indicated by the
      TSA" (lines ~1130-1156). So an expired or revoked TSA certificate does not by itself break an
      old token.       **The trade is three-dimensional, not two.** OpenTimestamps wins on trust model; RFC 3161 wins
      on long-horizon revocation handling and on latency; and OTS carries a **verification-tooling
      cost RFC 3161 does not** — checking an OTS proof needs Bitcoin Core, whereas checking a 3161
      token needs the certificate chain. Which of the three matters most is an ADR decision.
      **But the latency advantage is not free, and this is easy to misread as a win for RFC 3161**
      (caveat raised by oss-encryption-store, spot-checked by me). The TSA's tightness is its own
      assertion: "genTime is the time at which the time-stamp token has been created by the TSA"
      (line ~478), and the token's own `accuracy` field "represents the time deviation around the
      UTC time contained in GeneralizedTime" — a SEQUENCE of seconds/millis/micros in which "If
      either seconds, millis or micros is missing, then a value of zero MUST be taken for the
      missing field" (lines ~527-544). So the tighter capture-time bound and the trust substitution
      are **the same purchase**: "tighter" is the TSA's claim about its own clock, self-reported
      with a self-declared accuracy, not an independent measurement. The latency axis must not be
      read as a free advantage for RFC 3161 when it is bought with precisely the property that made
      OpenTimestamps preferable on the trust axis.
      **But the asymmetry is smaller than it first appears, and this bounds how much RFC 3161
      actually retires.** Step 5 of that same procedure requires that "The revocation information
      about that certificate, at the date/time of the Time-Stamping operation, MUST be retrieved."
      The RFC *mandates retrieving* historical revocation data; it cannot guarantee that a CRL or
      OCSP response from years earlier is still obtainable and interpretable. So RFC 3161's answer
      to revocation is protocol-level but its *inputs* are infrastructure-level — the same class of
      dependency as OTS's calendars, one layer down. What remains genuinely unsolved for **both**
      is trust-anchor/CA-root distrust (a root pulled from trust stores decades later is not covered
      by the RFC's provision) and hash-algorithm longevity. And whether real-world TSAs actually
      ship the long-term-validation material the procedure depends on is a **deployment question
      that neither survey checked** — flagged, not assumed.
      **These three are not equally actionable, and a flat list would mislead whoever writes the
      ADR** (framing from oss-encryption-store): CA-root/trust-anchor distrust and hash-algorithm
      longevity are **unfalsifiable until decades pass** — no work we can do now closes them, so
      they are **risk-acceptance decisions, not research tasks**. TSA long-term-validation-material
      availability is different in kind: it is **checkable today** against a real TSA's published
      practice and what it actually returns, which makes it the only one of the three that is a
      **task**, and the only one that could still change the pin recommendation. It should be
      labelled that way so it gets picked up rather than sitting beside two items nobody can close.
      **Two limits I would add, because both bear on whether this actually satisfies the evidence
      constraint:**
      (i) *Neither pin can attest capture time.* Both protocols define themselves the same way —
      OpenTimestamps: "A timestamp proves that some data existed prior to some point in time";
      RFC 3161: "existed before a particular time". So a pin gives an **upper bound on existence**,
      never the moment of capture. For a workplace dispute where the question is "when did I learn
      X", the capture timestamp remains **self-asserted**. The architectural consequence is that
      the stamp must be taken **at capture time**, not at manifest-build time, and the stamping
      interval should be kept short — otherwise the archive proves only "this record existed before
      some later point", which is weaker than it sounds.
      (ii) *If the pin is public and permanent, the pinned value must never be an unkeyed plaintext
      digest.* Publishing a Merkle root over ciphertext leaves (or over keyed HMACs) is fine. But
      publishing an unkeyed digest of plaintext into the Bitcoin blockchain would be the world's
      most durable **confirmation oracle** — permanent, public, and checkable by anyone who can
      guess a record's content, with no ability to retract. The leaf design and the pin choice are
      therefore coupled, not independent decisions.
      (iii) *OpenTimestamps' operational reality, verified from the official client README* — this
      closes the latency/format lead oss-encryption-store had flagged as unverified, and it adds a
      constraint neither of us had raised:
      - **Latency is hours, not seconds.** "It takes a few hours for the timestamp to get confirmed
        by the Bitcoin [blockchain]", and until then the calendars report "Pending confirmation in
        Bitcoin blockchain". So stamping at capture time yields a proof whose bound is hours
        *after* capture — which is still far tighter than stamping at manifest-build time, and
        reinforces that the stamp belongs in the capture path.
      - **Verification requires a Bitcoin node.** "While OpenTimestamps can *create* timestamps
        without a local Bitcoin node, to *verify* timestamps you need a local Bitcoin Core node (a
        pruned node is fine)." This is a real adoption barrier for footprint's evidence claim: an
        external verifier — a colleague, a lawyer, a court, or a future owner of the machine —
        must run Bitcoin Core to check the archive, which is a much heavier requirement than
        "verify a hash". If third-party verifiability is part of the point, this needs to be an
        explicit, documented cost rather than a surprise.
      - **Proofs come in two states, and only one survives the calendars.** "Incomplete timestamps
        are ones that require the assistance of a remote calendar to verify; the calendar provides
        the path to the Bitcoin block header." So an un-upgraded proof depends on a calendar that
        may not exist decades later, whereas an upgraded (complete) proof is self-contained.
        **Design rule: once a stamp confirms, upgrade it and store the complete proof, never the
        pending one** — otherwise the long-term verifiability of the archive is silently hostage to
        the uptime of donation-funded calendar servers.
      - Confirming the semantics in (i), the tool's own success output reads "Bitcoin block 358391
        attests existence **as of** 2015-05-28" — an existence bound, not a capture attestation.

    **No candidate in this survey implements any of this**; both surveys found only option 1
    (unencrypted, WARC-style) or the restic pattern (keyed identifier).
- **Paperless-ngx `PAPERLESS_EMAIL_GNUPG_HOME`**: I saw the setting name in the configuration
  reference but did not read its semantics. If footprint ever wants to preserve PGP-signed or
  PGP-encrypted mail, this is worth reading.

**Benchmarks that do not exist (stated plainly rather than asserted):**

- **No candidate publishes Malay/English code-switching transcription accuracy.** Onyx's
  Fireflies connector and AnythingLLM's built-in transcription both rest on third-party speech
  models, and none of the projects surveyed publishes a benchmark for Malay, let alone
  code-switched Malay/English. **I found no evidence either way**, so no candidate can be
  credited or penalised on this axis. This is being covered separately by the
  transcription-Malay workstream.
- **No candidate publishes a "participant-only capture" guarantee.** Onyx is the only project
  whose connectors even distinguish delegated per-user OAuth from tenant-wide service-account
  access; the rest simply index whatever credential they are given.

**Not covered in this pass (deliberately):**

- Encryption primitives (SQLCipher, gocryptfs, age, Cryptomator) — the oss-encryption-store
  workstream owns this. This memo only reports which candidates have *app-level* encryption:
  piler (email), Trilium (per-note), Open WebUI (optional SQLite), SurfSense (keys in OS keychain
  only). Everything else relies on the host.
- Capture tooling for specific sources (Teams/Zoom/WhatsApp/Outlook) — the oss-capture-tooling
  workstream owns this.

---

## Appendix: licence legend used in this memo

| Label | Meaning for a distributable product |
|---|---|
| **MIT** | Permissive. Reuse code freely with attribution. |
| **Apache-2.0** | Permissive, with an explicit patent grant. Reuse freely. |
| **AGPL-3.0** | Strong copyleft **plus a network clause**. Linking into a distributed binary, or offering a modified version as a network service, triggers source-disclosure obligations. Treat as "design reference, do not link". |
| **GPL-3.0** | Strong copyleft on distribution. Linking into a distributed binary triggers obligations. Treat as "design reference, do not link". |
| **BUSL 1.1** | Source-available, **not open source**. Production use is typically restricted until a change date. A hazard inside a core directory (SurfSense). |
| **Onyx Enterprise License** | Proprietary licence for `ee/` directories. Open core. |
| **Open WebUI License** | Source-available with a branding-preservation requirement. **Not OSI open source.** |
| **Docmost Enterprise License** | Proprietary licence for `packages/ee/`. Open core. |
| **Unverified** | Could not read licence text from primary sources. Do not depend on. |
