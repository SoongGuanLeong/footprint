# Per-source capture and export tooling: an OSS survey

*Research memo for project "footprint". Scope: the tools that actually get data OUT of each source (email, chat, meetings), and the credential model each one requires.*

## 1. The question, and the short answer

**Question.** For a participant-only personal work archive (single user, one local store, encrypted at rest, distributable, delegated per-user OAuth only), which existing free/OSS tools can capture and export data from email, chat, and meetings — and which of them require tenant/workspace admin rights and are therefore outside the boundary by definition?

**Short answer.** The per-source tooling is mature and good, but it is a *toolbox, not a product*. For email, `mbsync` + `notmuch`/`mu` + `lieer`/GYB give a solid, fully delegated, single-user capture path. For chat, `slackdump` is the strongest single tool in this entire survey (no admin needed, SQLite archive, incremental, and it already ships a local MCP server) — but it is **AGPL-3.0**. WhatsApp (KnugiHK, MIT) and Signal (signalbackup-tools, GPL-3.0) are participant-only and depend on the user extracting their own device backup. For meetings, local transcription is a solved problem (whisper.cpp, Buzz, Vibe, Meetily — all MIT), and capture is solved on the local-audio side.

The two hard gaps are: (a) **Microsoft Teams** — there is no participant-only OSS path; the only supported route needs tenant admin consent; and (b) **no single project does the whole job**. The closest whole-product fit is **Timelinize** (a local SQLite personal timeline that imports email, Telegram, SMS, photos, location and social DMs), but it is **AGPL-3.0**, explicitly alpha, and does not cover Slack/Teams/WhatsApp, live email sync, or meeting transcription.

Nothing in this survey provides application-level encryption at rest, append-only verifiable timestamps, semantic search, or a read-only MCP surface. Those are footprint's own build.

## 2. Candidate table

Licenses below were read from the repository's license metadata (GitHub API `license.spdx_id`) and, where the metadata said `NOASSERTION`, from the actual `LICENSE`/`COPYING` file. Where I could not read a file directly, the row says so.

### Email

| Candidate | License | What it covers | What it does NOT cover | Activity | Verdict |
|---|---|---|---|---|---|
| **isync / mbsync** | GPL-2.0 (mirror metadata; upstream on SourceForge) | IMAP↔Maildir (and IMAP↔IMAP) two-way sync; XOAUTH2 for Gmail/O365 | No indexing/search; no Gmail label semantics beyond flags; config-file driven | Upstream release 1.5.0 (Dec 2024); GitHub mirror stale (2017) | **ADOPT** (IMAP capture) |
| **OfflineIMAP** | GPL-2.0 (README "GNU General Public License v2.") | IMAP↔Maildir sync (Python 2) | Superseded; OAuth2 support is poor/deprecated | Last push 2023-06-13; repo self-labels LEGACY | **REJECT** (superseded by mbsync) |
| **offlineimap3** | unverified (no `LICENSE` at repo root; `COPYING` not retrievable this session) | Python-3 continuation of OfflineIMAP | Same as above | Pushed 2026-06-15 | **REJECT** (mbsync is the maintained choice) |
| **mu** (djcb/mu) | GPL-3.0 | Maildir indexer/searcher + Emacs client; local full-text search | Does not fetch mail (pairs with mbsync) | Active; pushed 2026-09-23 | **ADOPT-AS-PATTERN** (index/search design) |
| **notmuch** | GPL-3.0-or-later (COPYING read) | Maildir indexing/search (Xapian), tagging, `libnotmuch` C library | Does not fetch mail | Active; pushed 2026-08-24 | **ADOPT-AS-PATTERN** |
| **lieer** | GPL-3.0-or-later (LICENSE.md read) | Two-way Gmail↔notmuch sync via Gmail API + the user's own OAuth | Gmail only | Pushed 2026-04-21 | **ADOPT** (Gmail path) |
| **Got Your Back (GYB)** | Apache-2.0 | Backs up Gmail messages to local disk via Gmail API over HTTPS, using the user's own OAuth | Backup, not continuous sync; Gmail only | Pushed 2026-09-11 | **ADOPT** (Gmail backup) |
| **GAM** | Apache-2.0 | Google Workspace *domain administration* | Requires domain admin — tenant scope | Active | **REJECT** (admin/tenant scope) |
| **Google Takeout** | proprietary (Google), free | User-initiated export of the user's own account (Mbox for mail) | Manual, periodic; not an API | n/a | **ADOPT-AS-PATTERN** |
| **Google Workspace Data Export** | proprietary (Google) | Admin-run bulk export of an org | Requires admin | n/a | **REJECT** (admin/tenant scope) |
| **piler / MailPiler** | GPL-3.0 (LICENSE read: "version 3 of the License") | Enterprise email archiving; ingest via SMTP journaling/BCC or IMAP; web full-text search | Org-wide/team product; heavy server stack; not participant-only | Pushed 2026-09-07 | **REJECT** (enterprise/team scope); ingest design **ADOPT-AS-PATTERN** |
| **DavMail** | GPL-2.0 (README read) | Gateway: Exchange/O365 (EWS) → IMAP/SMTP/CalDAV/CardDAV/LDAP, using the user's own account credentials | Only relevant to Exchange/365; adds a Java gateway process | Active; pushed 2026-09-24 | **WATCH** (needed only for Exchange/365 users) |

### Chat

| Candidate | License | What it covers | What it does NOT cover | Activity | Verdict |
|---|---|---|---|---|---|
| **slackdump** (rusq) | **AGPL-3.0** | Archives Slack private/public messages, threads, files, users, emojis **without admin privileges**; outputs Slack Export, "dump", SQLite archive (json+gz), static HTML; incremental; **ships a local MCP server (STDIO/HTTP)** | Slack only | Very active; pushed 2026-09-25 | **ADOPT** (best-in-class) — **AGPL-3.0 constraint, see §5** |
| **WhatsApp-Chat-Exporter** (KnugiHK) | MIT | Parses WhatsApp chat databases from the user's own Android/iOS backups; HTML/JSON output; includes media | No live sync (must extract a device backup); details of format flags not re-verified this session | Pushed 2026-08-12 | **ADOPT** (best license here) |
| **signalbackup-tools** (bepaald) | GPL-3.0 | Decrypts the user's own Signal backup (30-digit passphrase) and exports to HTML/CSV/JSON/text with media | Signal only; requires the backup file + passphrase | Pushed 2026-08-22 | **ADOPT** |
| **Telegram Desktop export** (official client) | proprietary but free (Telegram) | Built-in "Export Telegram data": JSON/HTML + media, from the user's own account | Manual; not scriptable; not OSS | n/a | **ADOPT-AS-PATTERN** |
| **telegram-export** (expectocode) | MPL-2.0 | Telethon-based chat export | **Archived 2019** | Archived | **REJECT** (unmaintained) |
| **Telethon** | MIT | Python MTProto client — lets you build a participant-only exporter | You write the exporter | Active | **ADOPT** (library) |
| **Element / element-web** | **AGPL-3.0** | Client-side "Export chat" to HTML/JSON/txt from the user's own rooms | Matrix only; E2EE export limitations | Very active | **ADOPT-AS-PATTERN** (AGPL if you link the code) |
| **matrix-archive** (osteele) | MIT | Export a Matrix room archive | Stale (2021), 46 stars; E2EE handling unclear | Pushed 2021-06-17 | **WATCH** |
| **DiscordChatExporter** (Tyrrrz) | MIT | Exports Discord channels/DMs to HTML/JSON/CSV/TXT with media | Needs a bot token (must be in the server) or a **user token (self-bot — violates Discord ToS)** | Active; pushed 2026-09-01 | **WATCH** (ToS/legal caveat) |
| **teams-chat-export** (cvhyatt-code) | MIT | Exports a Teams chat to text/JSON/images via Graph | 0 stars, unproven; single-author | Pushed 2026-09-22 | **WATCH** |
| **teams-chat-extract** (xprtyg33k) | **NONE (no license file)** | Teams chat export via Graph | No license = all rights reserved, unusable | Pushed 2026-06-12 | **REJECT** (unlicensed) |
| **Teams Export APIs** (Microsoft) | proprietary | Compliance/eDiscovery export of Teams content | **Requires application permissions + admin consent** | n/a | **REJECT** (admin/tenant scope) |

### Meetings

| Candidate | License | What it covers | What it does NOT cover | Activity | Verdict |
|---|---|---|---|---|---|
| **whisper.cpp** (`examples/stream`) | MIT | Real-time microphone transcription; sliding-window + VAD; fully local | ASR engine only — no capture UI, no diarisation, no store | Very active; pushed 2026-09-24 | **ADOPT** (transcription engine) |
| **Buzz** (chidiwilliams) | MIT | Offline transcription/translation of audio, Whisper-based, GUI+CLI | File-based; not a live meeting capturer | Pushed 2026-09-23 | **ADOPT** (transcription) |
| **Vibe** (thewh1teagle) | MIT | Offline transcription desktop app, cross-platform | Transcription only | Pushed 2026-09-05 | **ADOPT** (transcription) |
| **Meetily** (Zackriya-Solutions/meeting-minutes) | MIT | Local capture + real-time transcription (Whisper/Parakeet), local summarisation via Ollama; "no data ever leaves your computer" | Open-core: a paid **Meetily PRO** tier exists for "enhanced accuracy, advanced exports, custom summary workflows, team-ready features" | Very active; pushed 2026-09-15 | **ADOPT-AS-PATTERN / WATCH** (open-core) |
| **Hyprnote → "anarlog"** (fastrepl/hyprnote) | MIT (community app) | Local-first meeting notepad, system-audio capture + transcription | "Source-visible enterprise components are commercially licensed" (open-core); team has pivoted to a new product (char.com) | Pushed 2026-09-25 | **WATCH** (open-core + pivot risk) |
| **Amurex** (thepersonalaicompany) | **AGPL-3.0** | Browser-extension meeting copilot | Stale; extension/browser scope | Last push 2025-05-27 | **WATCH** (AGPL + stale) |
| **Zoom recording downloaders** (georgeb3 et al.) | MIT (that fork) | Bulk-download Zoom cloud recordings | Several use **Server-to-Server OAuth = account-level/admin** — outside the boundary; 0 stars | Pushed 2025-12-30 | **REJECT** (admin credential model) |
| **Chrome-recording-transcription-extension** (recallai) | **NONE** | Records/scrapes Meet captions locally | No license = unusable | Pushed 2025-11-11 | **REJECT** (unlicensed) |
| **GMeetSummarizer** | **NONE** | Transcribe Meet recordings from Drive links | No license; 1 star | Pushed 2025-06-17 | **REJECT** (unlicensed) |

### Whole-product candidates

| Candidate | License | What it covers | What it does NOT cover | Activity | Verdict |
|---|---|---|---|---|---|
| **Timelinize** (timelinize/timelinize) | **AGPL-3.0** | A single local SQLite personal timeline. Importers include: Emails, Telegram, Android SMS, Apple Messages, Facebook, Facebook Messenger, Instagram, Instagram DMs, X/Twitter (+DMs), Google Voice, Google Location History, GPX/GeoJSON/KML/NMEA, Google Photos, Apple Photos, media, Strava, contacts/calendars/vCard, iPhone/iCloud backups. Local-only. | **Slack, Teams, WhatsApp, Discord**; live email sync (import-based); meeting capture/transcription; app-level encryption at rest; semantic search; MCP surface; verifiable timestamps. Self-described **alpha**, "may not be backwards-compatible" | Pushed 2026-05-22 | **WATCH / ADOPT-AS-PATTERN** (closest fit; AGPL is the blocker) |

## 3. Per-candidate detail with citations

### Email

**isync / mbsync** — upstream https://isync.sourceforge.io/ ; GitHub mirror https://github.com/notetiene/isync (mirror metadata reports `GPL-2.0`; the mirror's last push is 2017-09-05, so use the upstream SourceForge tree, not the mirror). `mbsync` synchronises Maildir and IMAP4 mailboxes both ways and supports OAuth2 (XOAUTH2) for Gmail and O365. It authenticates as the user — no admin involvement. It does no indexing; the canonical pairing is mbsync → notmuch or mbsync → mu. **GPL-2.0** is copyleft; invoking the `mbsync` binary as a separate process is fine, but do not statically link or vendor it into a differently-licensed binary without a licence review.

**OfflineIMAP** — https://github.com/OfflineIMAP/offlineimap ; the README's License section reads "GNU General Public License v2." The repo description itself says "[LEGACY: move to offlineimap3]" and it is Python 2. Python-2 is a hard blocker for a 2026 distributable. `offlineimap3` (https://github.com/offlineimap/offlineimap3) is the Python-3 continuation and was pushed 2026-06-15, but I could not retrieve a `LICENSE`/`COPYING` file for it this session, so its exact SPDX id is unverified. Either way, mbsync is the better-maintained option.

**mu** — https://github.com/djcb/mu , **GPL-3.0** (repo license metadata). Maildir indexer/searcher + Emacs client + Guile bindings. Pushed 2026-09-23. Useful as a search/index design reference and as an optional local index; it does not fetch mail.

**notmuch** — https://github.com/notmuch/notmuch (mirror of http://git.notmuchmail.org/git/notmuch). The `COPYING` file states "either version 3 of the License, or (at your option) any later version" → **GPL-3.0-or-later**. Xapian-backed indexing/tagging plus a C library. Pushed 2026-08-24.

**lieer** — https://github.com/gauteh/lieer , `LICENSE.md`: "either version 3 of the License, or (at your option) any later version" → **GPL-3.0-or-later** (GitHub metadata says `NOASSERTION` because the file is a custom-prefixed GPL). Two-way Gmail↔notmuch sync using the Gmail API and the user's own OAuth. Pushed 2026-04-21.

**Got Your Back (GYB)** — https://github.com/GAM-team/got-your-back , **Apache-2.0**. README: "a command line tool for backing up your Gmail messages to your local computer. It uses Gmail's API over HTTPS." Authenticates as a single user's own Google account; no admin. Pushed 2026-09-11.

**GAM** — https://github.com/GAM-team/GAM , **Apache-2.0**, "command line management for Google Workspace." This is a domain-administration tool. Anything it does beyond the user's own account needs admin — **outside the participant-only boundary**.

**Google Takeout / Workspace Data Export** — Takeout (https://takeout.google.com) is a user-initiated export of the user's own account and is the delegated, participant-only pattern; it is not an API and is manual/periodic. The Workspace *Data Export* tool is admin-only. **Takeout = pattern; Data Export = reject.**

**piler / MailPiler** — https://github.com/jsuto/piler , https://www.mailpiler.org/ . The `LICENSE` file: "This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, version 3 of the License." → **GPL-3.0** (version 3 exactly). Piler is an *enterprise* email-archiving appliance: ingest by SMTP journaling/BCC or IMAP, MySQL + Sphinx + Redis stack, web search UI. It is designed for org-wide journaling, i.e. a team product, not a single participant's local store. Its **ingest + full-text search design is worth copying**; the product is not a fit.

**DavMail** — https://github.com/mguessan/davmail , README: "DavMail is licensed under the GNU General Public License v2.0." It is a gateway that exposes Exchange/O365 (EWS) as IMAP/SMTP/CalDAV/CardDAV/LDAP, using the user's own account credentials/OAuth. Pushed 2026-09-24, 760 stars. Only relevant if footprint must support Exchange/365-hosted mail; it adds a Java daemon to the install.

### Chat

**slackdump** — https://github.com/rusq/slackdump , **AGPL-3.0** (repo license metadata). README: "archive your private and public Slack messages, users, channels, files and emojis. Generate Slack Export **without admin privileges**." It explicitly supports "create a Slack Export archive without admin access" and "create incremental Slack archives." Outputs: Slackdump Archive (SQLite, or json+gz), Standard/Mattermost Slack Export, static HTML (`slackdump convert -f html`), and it "displays images" (media). It also "Local MCP Server to use with Opencode, Claude, or any other AI tool supporting mcp over STDIO or HTTP." Pushed 2026-09-25, 2,796 stars. **This is the single best-designed capture tool in the survey and the closest thing to footprint's own read-only MCP idea.** The licence is the problem — see §5.

**WhatsApp-Chat-Exporter (KnugiHK)** — https://github.com/KnugiHK/Whatsapp-Chat-Exporter , **MIT**. Repo description: "A customizable, cross-platform tool for parsing WhatsApp chat databases from Android and iOS/iPadOS backups." The repo root contains `LICENSE`, `Whatsapp_Chat_Exporter/`, `pyproject.toml`, `tests/`. It works from the user's own device backup — participant-only, no admin, no server. Pushed 2026-08-12, 1,238 stars. *I was unable to re-fetch the README text this session (repeated fetch failures), so the exact output-format flags and media handling are not re-verified here; the licence (MIT) and the input model (device backup) are verified.*

**signalbackup-tools (bepaald)** — https://github.com/bepaald/signalbackup-tools , **GPL-3.0**. Works on the user's own Signal backup file (decrypting with the user's 30-digit passphrase) and can export to HTML/CSV/JSON/text including media. Pushed 2026-08-22, 1,410 stars. Participant-only.

**Telegram** — The Telegram Desktop client has a built-in "Export Telegram data" (JSON/HTML + media) that runs entirely from the user's own account. For automation, **Telethon** (MIT) is the maintained MTProto library and lets you write a participant-only exporter; `expectocode/telegram-export` (MPL-2.0) is **archived (2019)** and should not be depended on. Timelinize also has a Telegram importer.

**Matrix / Element** — `element-web` (https://github.com/element-hq/element-web) is **AGPL-3.0**. The client's "Export chat" feature (HTML/JSON/txt) is participant-only and client-side; the Element docs page "Exporting Messages" documents it. `osteele/matrix-archive` (MIT) is stale (2021, 46 stars) and its E2EE handling is unclear.

**Discord** — `Tyrrrz/DiscordChatExporter` (https://github.com/Tyrrrz/DiscordChatExporter) is **MIT** and exports to HTML/JSON/CSV/TXT with media. The catch is credentials: a **bot token** requires the bot to be added to the server (often needs server-admin action), and a **user token** is a self-bot, which violates Discord's Terms of Service. Flag as a legal/ToS risk even though the licence is permissive.

**Microsoft Teams** — this is the weak spot. Microsoft's supported bulk export (https://learn.microsoft.com/en-us/microsoftteams/export-teams-content) is the **Teams Export API**, which requires *application* permissions and admin consent — tenant scope, outside the boundary. Graph's delegated `Chat.Read` scope can read the signed-in user's own chats, but there is no mature OSS tool that does it: `cvhyatt-code/teams-chat-export` is MIT but has 0 stars and is unproven; `xprtyg33k/teams-chat-extract` has **no licence file at all** (all rights reserved) and cannot be used. **No ADOPT candidate exists.**

### Meetings

**whisper.cpp** — https://github.com/ggml-org/whisper.cpp , **MIT**. `examples/stream/README.md`: "a naive example of performing real-time inference on audio from your microphone. The `whisper-stream` tool samples the audio every half a second and runs the transcription continuously," with a sliding-window + VAD mode (`--step 0 -vth 0.6`). Fully local. Pushed 2026-09-24. The Whisper model weights are also MIT (OpenAI). Whisper is multilingual and nominally supports Malay, but see §6 on code-switching.

**Buzz** — https://github.com/chidiwilliams/buzz , **MIT**, "transcribes and translates audio offline on your personal computer." Pushed 2026-09-23, 21,670 stars. File-based, not a live capturer.

**Vibe** — https://github.com/thewh1teagle/vibe , **MIT**, "Transcribe on your own!", cross-platform desktop. Pushed 2026-09-05, 7,600 stars.

**Meetily** — https://github.com/Zackriya-Solutions/meeting-minutes , **MIT**, 31,095 stars, pushed 2026-09-15. README: "runs entirely on your local machine. It captures your meetings, transcribes them in real-time... all without sending [data]"; "Local Transcription: Transcribe meetings entirely on your device using Whisper or Parakeet models. No cloud required"; summarisation via Ollama (local) or optional cloud providers. **Open-core flag:** the README advertises a paid **Meetily PRO** tier ("enhanced accuracy, advanced exports, custom summary workflows, team-ready features"). The capture/transcribe core is MIT, but confirm which features are gated before depending on it.

**Hyprnote / "anarlog"** — https://github.com/fastrepl/hyprnote , **MIT** for the community app. The README carries two important notes: "The team is now building char (char.com)" (pivot risk) and "Source-visible enterprise components are commercially licensed" (open-core). Local-first meeting notepad. Pushed 2026-09-25, 9,393 stars. **WATCH.**

**Amurex** — https://github.com/thepersonalaicompany/amurex , **AGPL-3.0**, 2,874 stars, but **last push 2025-05-27** — roughly 16 months stale as of 2026-09-25. Browser-extension scope. **WATCH.**

**Zoom / Google Meet recording fetchers** — the Zoom cloud-recording downloaders (e.g. `georgeb3/zoom-recording-downloader`, MIT) largely use **Server-to-Server OAuth**, which is an account-level/admin credential — outside the boundary. Zoom's *user-level* OAuth can list the user's own recordings, but the OSS tools in this space are thin and mostly admin-oriented. For Google Meet, recordings land in the *organizer's* Drive; the only local OSS options found (`recallai/Chrome-recording-transcription-extension`, `ilya-kolchinsky/GMeetSummarizer`) have **no licence file**, so they are unusable. **This is a genuine gap.**

### Whole-product candidates

**Timelinize** — https://github.com/timelinize/timelinize , https://timelinize.com , **AGPL-3.0** (repo license metadata). Description: "Store your data from all your accounts and devices in a single cohesive timeline on your own computer." Go, 3,675 stars, pushed 2026-05-22. The data-preparation docs enumerate its importers: Facebook, Instagram, Strava, X (Twitter), Apple Contacts, calendars, contact lists, **Emails**, vCard, Google Location History, GeoJSON/GPX/KML-GX/NMEA, Google Photos, Apple Photos, media, **Android text messages**, **Apple Messages**, **Telegram**, Google Voice, iPhone, iCloud. It is local-only and single-user — architecturally the same shape footprint wants. But the docs carry an **alpha notice**: "Timelinize is unfinished software. New versions may not be backwards-compatible, so expect that you will have to reset with new timelines when you upgrade." It does **not** cover Slack, Teams, WhatsApp or Discord; its email support is import-based (Takeout/mbox) rather than live IMAP sync; and it has no meeting capture/transcription, no application-level encryption at rest, no semantic search, no MCP surface, and no verifiable-timestamp mechanism.

Adjacent products worth knowing but out of scope: ArchiveBox (web page archiving), paperless-ngx (document management), Hoarder/Karakeep (bookmarks). None cover chat/meeting capture.

## 4. Closest fits and the gap

**No single OSS project does most of footprint's job.** The field is a set of excellent per-source extractors with no integration layer, no unified store, no encryption story, and no read-only query surface.

The closest candidates and the specific gap each leaves:

- **Timelinize** (AGPL-3.0) — closest whole-product *shape*: one local SQLite timeline, single-user, multi-source import. Gap: no Slack/Teams/WhatsApp/Discord, no live email sync, no meeting capture/transcription, alpha churn, and AGPL is disqualifying for a product you intend to distribute under your own terms (see §5).
- **slackdump** (AGPL-3.0) — closest to the *capture + MCP* half. It proves the "archive to SQLite, expose over MCP, no admin required" pattern end-to-end. Gap: Slack only; AGPL-3.0.
- **Meetily / anarlog / whisper.cpp** (MIT) — closest to the *meeting* half. Gap: meetings only, no email/chat, no unified store, and the first two are open-core.
- **mbsync + notmuch/mu + lieer/GYB** (GPL/Apache) — the *email* half is essentially solved and fully delegated. Gap: it is a set of CLI tools, not a component you can embed, and GPL/copyleft applies to the GPL pieces.
- **piler** (GPL-3.0) — closest to the *archive + search* half, but it is an enterprise team appliance, which is the opposite of footprint's single-user model.

What **none** of them provide, and footprint must build: application-level encryption at rest (they all assume the host protects the data), append-only verifiable timestamps, semantic search, a stable read-only MCP surface (slackdump is the only one with any MCP, and it is per-source and AGPL), and a participant-only Teams path.

## 5. Licence flags (read this before depending on anything)

**AGPL-3.0 — highest risk.** `slackdump`, `element-web`, `Amurex`, `Timelinize`. AGPL's network clause is triggered if you *modify* the software and let users interact with it over a network, or if footprint is a derivative work. **Calling an unmodified `slackdump` binary as a subprocess is the safe pattern** (separate process, mere aggregation), but if footprint bundles slackdump, modifies it, or embeds it as a library while offering a hosted service, AGPL obligations attach. Get a licence review before shipping any AGPL component inside a distributable product. Timelinize being AGPL-3.0 is the main reason it is WATCH rather than ADOPT.

**GPL-2.0 / GPL-3.0 — copyleft.** `isync`, `OfflineIMAP`, `davmail` (GPL-2.0); `mu`, `notmuch`, `lieer`, `signalbackup-tools`, `piler` (GPL-3.0). Fine as separate CLI processes; do not link or vendor into a differently-licensed binary without review. GPL-3.0 adds anti-tivoization/patent terms that GPL-2.0 lacks.

**Permissive — safe to embed.** MIT: WhatsApp-Chat-Exporter, DiscordChatExporter, whisper.cpp, Buzz, Vibe, Meetily, Hyprnote/anarlog, teams-chat-export, matrix-archive, Zoom downloaders. Apache-2.0: GYB, GAM. MPL-2.0: telegram-export (archived).

**No licence at all — do not use.** `xprtyg33k/teams-chat-extract`, `recallai/Chrome-recording-transcription-extension`, `ilya-kolchinsky/GMeetSummarizer`. No licence file means all rights reserved, regardless of the code being public.

**Open-core (feature you need may be paid).** Meetily (Meetily PRO: "enhanced accuracy, advanced exports, custom summary workflows, team-ready features"), Hyprnote/anarlog ("Source-visible enterprise components are commercially licensed"). Verify the specific feature you need is in the MIT core before depending on it.

**Source-available but NOT open source.** None of the verified candidates are BUSL/Elastic/SSPL. The proprietary items in scope (Google Takeout/Data Export, Telegram Desktop, Teams Export APIs, Zoom cloud) are closed-source services, not source-available projects.

**Proprietary service dependency (not a licence issue, a design issue).** Anything routed through a cloud API (Meetily's optional Claude/Groq/OpenRouter providers, Zoom/Teams/Meet cloud APIs) breaks the local-only, single-user, encrypted-at-rest promise and is out of scope for the capture path.

## 6. Open questions and things I could not verify

1. **Malay/English code-switching accuracy.** No candidate in this survey publishes a Malay or code-switching benchmark. Whisper is multilingual and nominally supports Malay (`ms`), and whisper.cpp/Buzz/Vibe/Meetily can all run multilingual models, but **I found no primary-source accuracy number for Malaysian code-switched speech.** This is a real evaluation you must run yourself against your own audio. Treat "supports Malay" and "handles code-switching well" as different claims; only the first is documented.
2. **WhatsApp-Chat-Exporter output details.** Repeated fetches of its README failed this session. Verified: MIT licence, parses Android/iOS backups, repo structure. Not re-verified: exact output-format flags, media handling, and whether root is required for any extraction step.
3. **offlineimap3 licence.** No `LICENSE`/`COPYING` retrievable this session; SPDX id unverified. (The original OfflineIMAP is GPL-2.0 per its README.) Moot if you pick mbsync.
4. **isync licence from the primary upstream.** The only GitHub mirror (`notetiene/isync`) is stale (2017) and reports GPL-2.0. Upstream lives on SourceForge (https://isync.sourceforge.io/); I did not read the upstream COPYING directly. Treat as GPL-2.0-or-later pending confirmation.
5. **Teams participant-only viability.** Whether Graph's delegated `Chat.Read` + `/me/chats` can reconstruct a full, media-complete Teams chat export in practice — and whether the user's tenant permits the needed delegated scopes — is untested here. There is no OSS tool to lean on, so this needs a spike.
6. **Timelinize importer credential models.** I confirmed the importer *list* and that it is local-only, but did not audit each importer (e.g. does its email importer read mbox/Takeout only, or can it do live IMAP?). Its alpha notice means any integration would be brittle.
7. **Encryption, append-only, and timestamps.** No candidate provides application-level encryption at rest, an append-only/verifiable-timestamp log, or a tamper-evident chain. These are unaddressed by the entire survey and must be built into footprint.
8. **Meeting capture scope.** This memo covers *capture and transcription* tools; it does not evaluate diarisation quality, system-audio capture on Ubuntu 26.04 (PipeWire/PulseAudio loopback), or macOS audio-permission constraints — all of which are separate, load-bearing questions for a distributable product.

---

*Method note: licences were read from GitHub's repo license metadata (`license.spdx_id`) and, where that returned `NOASSERTION`, from the repository's own `LICENSE`/`COPYING` file. Activity is the last `pushed_at` date as observed on 2026-09-25. Star counts are as observed on that date. No repository was cloned, installed, or modified; the footprint repo was not touched.*
