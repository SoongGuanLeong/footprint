# OSS Reuse Survey — Is there an existing free/OSS solution for footprint?

*Research memo for wayfinder ticket #5 (Architecture and storage decision). Written 2026-09-25.*
*Synthesises five sub-surveys: [archive-platforms](oss/archive-platforms.md), [capture-tooling](oss/capture-tooling.md), [encryption-store](oss/encryption-store.md), [transcription-malay](oss/transcription-malay.md), [agent-interface](oss/agent-interface.md). Licences below were re-verified against each repository's GitHub licence metadata before being written down here.*

---

## Summary: the question and the answer

**Question.** Does a free/OSS project already exist that footprint can adopt instead of building?

**Answer. No. There is no whole-product fit, and the gap is structural rather than marginal.**

The market splits cleanly in two, and footprint sits in the seam:

- **Multi-user capture-breadth products** (Onyx, and to a lesser degree MailPiler and Paperless-ngx) can pull email, chat and meeting data from many sources, but they are organisation products by design, ship no application-level encryption, and target a server, not a person's laptop.
- **Single-user local tools** (Khoj, Karakeep, ArchiveBox, Trilium, msgvault) get the delivery shape right — local, self-hosted, single-user — but each covers one content type, or covers many and leaves the store in plaintext.

Three specific capabilities exist **nowhere together**:

1. **Participant-scoped multi-source capture** — reading only what the user's own account could lawfully open, via delegated per-user OAuth, with no tenant-wide admin grant.
2. **Application-level encryption at rest** — including the search index and the embeddings, not just the main database. This matters because the reference host has no full-disk encryption.
3. **A read-only agent surface** — and this is the sharpest negative result: shipping an MCP server does not mean shipping a read-only one. Of the projects surveyed that expose MCP, **three hand the agent more capability than a recall interface needs** (ArchiveBox exposes shell and crawl, Karakeep ships create-* write tools, mcp-memory-service is a write-oriented store). MCP has **no protocol-level read-only enforcement**: authorization is optional and HTTP-only, stdio servers are explicitly instructed *not* to implement it and to take credentials from the environment, and `readOnlyHint` is an explicitly untrusted hint. The boundary is entirely footprint's to build.

The closest single project is **msgvault** — see below — and it is the cleanest possible illustration of the gap: near-perfect shape, plaintext store.

---

## Recommended adoptions

These are the components worth taking, as code.

| Component | Candidate | Licence | Verdict |
|---|---|---|---|
| Store encryption | **SQLCipher** | BSD-3-Clause | **ADOPT** — page-level AES-256-CBC + HMAC-SHA512, PBKDF2; codec sits below the page cache, so the FTS5 index is inside the same cipher boundary |
| Store encryption (alt) | **SQLite3MultipleCiphers** | MIT | **ADOPT** as the alternative — same role, MIT, no attribution clause |
| Vector index | **sqlite-vec** | Apache-2.0 (dual MIT) | **ADOPT** — vectors live in ordinary SQLite tables, so embeddings are encrypted by the same boundary; brute-force KNN is fine at personal scale |
| Agent surface | **MCP over stdio** | spec | **ADOPT**, pinned to a revision; latest 2026-07-28 |
| STT engine | **faster-whisper** / **whisper.cpp** | MIT (code) / MIT | **ADOPT** as engine and runtime |
| VAD | **Silero VAD** | MIT | **ADOPT** |
| Gmail capture | **Got Your Back (GYB)** | Apache-2.0 | **ADOPT** — backs up the user's own mailbox with the user's own OAuth, no admin rights |
| IMAP capture | **isync/mbsync** + **notmuch** | GPL-2.0 / GPL-3.0-or-later | **ADOPT** as separate processes only (see licence discipline) |
| WhatsApp capture | **WhatsApp-Chat-Exporter** (KnugiHK) | MIT | **ADOPT** — backup-file input, no API needed |
| Slack capture | **slackdump** | AGPL-3.0 | **ADOPT ONLY AS A SEPARATE UNMODIFIED PROCESS** — see licence discipline |
| Timestamping | **OpenTimestamps** / **RFC 3161** | — | **BUILD** — no candidate in any of the five surveys integrates either |

**Licence discipline, stated plainly.** AGPL-3.0 is the dominant hazard in this space (Khoj, Karakeep, basic-memory, Timelinize, slackdump, Joplin, Logseq, Trilium, Docspell, Reor, Docmost). Because footprint is a product you distribute, an AGPL dependency that you link, bundle, or host triggers network copyleft. The safe pattern is to **invoke AGPL tools as separate unmodified CLI processes across a process boundary**, never as a linked library, and to state that constraint in the spec. Where a permissive alternative exists in the same role, prefer it — which is why SQLite3MultipleCiphers (MIT) is listed beside SQLCipher (BSD-3).

---

## Adopt-as-pattern (copy the design, do not depend on the code)

| Project | Licence | What to take |
|---|---|---|
| **Onyx** (ex-Danswer) | MIT Expat outside `ee/`, Onyx Enterprise License inside `ee/` (open core). Verified at file level: MIT = all seven footprint-relevant connectors (gmail, outlook, imap, slack, teams, zoom, fireflies), the MCP server, and the search/indexing core. Enterprise = the MCP client/actions registry, per-connector permission sync, and **post-retrieval result censoring** | Connector architecture across 56 sources; delegated vs tenant-wide OAuth as a first-class distinction. The single-tool-module MCP server is the **positive precedent** for a non-mutating surface. **Reject as a base**: multi-user, no encryption. Note the licence boundary sits exactly where footprint must do its own work — result censoring is the job the read-only surface performs |
| **Karakeep** (ex-Hoarder) | AGPL-3.0 | The delivery shape — single-user self-host + official MCP server + documented REST API + local model providers + OCR |
| **ArchiveBox** | MIT | Durable-provenance artifacts and the "readable in 50–100 years without the tool" goal. Also the cautionary case for MCP write exposure |
| **obsidian-mcp** (StevenStavrakis) | MIT | **Best local containment design in the survey**: vaults allow-listed at process start, per-call path containment, symlinks blocked, never listens on a network interface, stdout reserved for the protocol |
| **piler / MailPiler** | GPL-3.0-only | **Feature checklist only** — legal hold, retention rules, dedup, fingerprinting, audit-log search. **Do not copy its implementation** (see rejects) |
| **Khoj** | AGPL-3.0 | Semantic recall over a local corpus with local LLMs. Note: no MCP server found by two independent surveys; the request is an open issue |
| **Cryptomator** | GPL-3.0 | The key hierarchy: scrypt → KEK → RFC 3394 key-wrap. Do not link |
| **Standard Notes** | AGPL-3.0 | Key hierarchy: Argon2id → root key → per-purpose item keys |

---

## Rejects, with reasons

| Candidate | Reason |
|---|---|
| **msgvault** | The closest whole-product fit in the entire survey — MIT, local-first archive of email, chat, meetings, calendars and contacts, with keyword + semantic + hybrid search, attachment extraction, a web UI, a TUI, an HTTP API **and an MCP server**. Its own `SECURITY.md` states: "The SQLite database is not encrypted… OAuth tokens are stored as plaintext JSON files… Mitigation: Rely on OS-level full-disk encryption." It is also self-described **alpha**, and its optional enrichment features send selected data to configured endpoints by default. **This is precisely the assumption footprint cannot make**, which makes it the strongest evidence that app-level encryption of the store *and the index* is footprint's actual differentiator — not capture breadth |
| **piler / MailPiler** (as implementation) | Its "message encryption" is AES-256-CBC with a **static key and IV read from the config file** — deterministic, so shared prefixes leak — and **no MAC, so no tamper detection**, with legacy Blowfish-CBC for older records. An archive whose integrity cannot be verified contradicts the append-only / verifiable-artifact requirement. Its fingerprinting feature is not a substitute for authenticated encryption |
| **Paperless-ngx** | Its own README: "should never be run on an untrusted host because information is stored in clear text without encryption." (Its IMAP + OAuth ingestion design is worth copying.) |
| **encfs** | Dormant; unauthenticated crypto; 2014 audit; four CVEs in Debian as of 2026; the author recommends gocryptfs instead |
| **LUKS, VeraCrypt** | Require root. Also unusable as a per-install product default |
| **Anytype** | **Not open source** — "Any Source Available License 1.0", source-available |
| **Open WebUI** | **Not open source** — "Open WebUI License" (BSD-3 plus a branding-preservation clause above 50 users), non-OSI |
| **NeMo Sortformer diarization, Meta MMS-1B-All, SeamlessM4T-v2** | CC-BY-NC-4.0 — **non-commercial**. Fatal for a distributed product |
| **DuckDB encrypted mode** | Not a reject so much as deferred: DuckDB itself says it "does not yet meet the official NIST requirements" |
| **DiscordChatExporter** (user-token mode) | MIT, but the user-token mode is a self-bot and violates Discord's ToS |
| **GAM, Google Workspace Data Export, piler, Zoom S2S downloaders** | All require tenant/workspace admin scope — outside the participant-only boundary by definition |

---

## WATCH

- **Mesolitica Malaysian Whisper** (`mesolitica/Malaysian-whisper-large-v3-turbo-v3`) — best technical fit, trained on ms+en+zh+ta including Manglish, non-gated, word timestamps. **Licence undeclared** — no licence tag, no LICENSE file; only the surrounding `malaya-speech` *code* is MIT. Legally blocked until the author declares a weight licence.
- **Qwen3-ASR-1.7B** — Apache-2.0, non-gated, lists Malay among 30 languages plus language ID and a forced aligner, but **no published code-switching benchmark**. Worth a spike.
- **Timelinize** — local SQLite personal timeline importing email/Telegram/SMS/photos/location/social DMs, but AGPL-3.0, self-described alpha, no Slack/Teams/WhatsApp, no live email sync, no meeting transcription.
- **DuckDB v1.4+ encryption** — AES-GCM-256/CTR-256 and it encrypts WAL and temp files, which closes real leak holes; re-evaluate when it leaves experimental.

---

## The gap footprint must build

1. **Participant-only capture filter.** Nothing in the ecosystem has this concept. Existing tools either take everything their credentials allow or rely on the source's own access control. This is footprint's core, not a plugin.
2. **A read-only *and* confined agent surface.** Two independent controls — not convention, not annotations: register **no write tools at all**, plus a **read-only database role**. Both are needed because they cover different axes: a per-consumer credential scopes *which* consumer, the allow-list plus DB role scopes *which operations*.
   
   **Non-mutating is not the same as confined**, and this is the trap to avoid: the best available precedent (Onyx) exposes three tools, of which two are open-world egress — "Search the web" and "Fetch full page content". A surface can be read-only and still be an unbounded data-egress path. For a participant-only archive over a closed corpus, Hermes should expose **no web-search tool and no arbitrary-URL fetch tool at all**; every tool must resolve against the local corpus and nothing else. This also bears on #3's no-third-party-disclosure condition.
   
   Enforcement cannot be borrowed: Onyx delegates authorisation to its data layer because it has per-document ACLs, and its token verifier returns a single coarse scope with no read/write split. footprint is single-user with no ACL layer, so take the *shape* (small fixed tool set, per-consumer credential, no CLI/ORM reflection, allow-list at launch, per-call path containment, no network listener, stdout reserved for the protocol) and build the enforcement.
3. **Microsoft Teams participant-only capture.** There is no OSS path. Microsoft's only supported bulk export is the Teams Export API, which requires application permissions and tenant admin consent — outside the boundary by definition. Graph's delegated `Chat.Read` *could* read a user's own chats, but no mature OSS tool does it (one MIT tool has zero stars; another has no licence at all). **This is a build item, and it is the largest single unknown in the capture layer.**
4. **Integrity that survives the encryption decision.** Three verified precedents, and none of them is directly reusable:
   - **restic is the anti-pattern.** Its design doc confirms the config *and* the index and snapshot files are encrypted, integrity is a Poly1305-AES MAC, and the storage ID is the SHA-256 of the **encrypted** bytes. So verification requires the key and the identifier is meaningless without it — exactly wrong for evidence that must outlive the key.
   - **WARC 1.1 is the structural precedent, but plaintext.** It defines per-record, algorithm-agile digests (`WARC-Block-Digest`, `WARC-Payload-Digest`, with no algorithm mandated) plus a separate CDX/CDXJ index. That is container + independent manifest, done right.
   - **Maildir + notmuch/mu supplies the derivability property**: the index is rebuildable from the store alone, and therefore disposable.
   
   Consequence: the manifest must be checkable **without the key**, must be **derivable from the store alone**, and the dependency must run strictly **store → manifest, never manifest → store** — otherwise the store is not append-only regardless of its schema.
   
   **But "verifiable without the key" collides with "no disclosure".** An unkeyed digest manifest is itself a leak: it exposes record counts, sizes and timestamps, and an unkeyed digest is a confirmation oracle — anyone who can guess a record's content can hash the guess and confirm that record exists, without the key. This is a published attack class (Halevi, Harnik, Pinkas and Shulman-Peleg, *Proofs of Ownership in Remote Storage Systems*, CCS 2011, eprint.iacr.org/2011/207).
   
   **The structure that resolves the false choice is a transparency log.** RFC 6962 (Certificate Transparency) defines publicly auditable, append-only Merkle-tree logs with signed tree heads, and Google's Trillian generalises this to arbitrary data while letting the application choose the leaf value (default Merkle hash `SHA-256(0x00 | leaf.LeafValue)`). Hashing the **ciphertext** into the leaf gives key-free verifiability with no plaintext oracle, plus inclusion and consistency proofs.
   
   **And the structure alone is not enough.** A signed tree head is only as trustworthy as the independence of its signer. CT works because many mutually-distrusting parties run monitors and gossip (RFC 6962 s5.3, s7.3); a single-user local archive has neither, so the same key that writes the archive can rewrite history and sign a fresh head, undetectably. The Merkle structure buys **efficient proofs relative to a tree head**, not append-only-ness. **Something external must pin that head** before "append-only" is a guarantee rather than a claim.
   
   Four constraints follow, all of which are build work:
   - **Pin externally, and treat the pin as load-bearing, not optional.** This is what makes the property meaningful against the archive's own writer — and in a workplace dispute the owner is exactly the party whose evidence must be credible to someone else.
   - **The pinned structure must be keyed or ciphertext-derived**, enforced by construction rather than review. A public, permanent pin over an *unkeyed plaintext* digest would be the most durable confirmation oracle conceivable: permanent, public, unretractable. The keyed HMAC is therefore a **precondition** for a public pin, not an optional extra.
   - **Stamp at capture time, not at manifest-build time.** Both protocols bound existence from *above* only — OTS says "existed prior to some point in time", RFC 3161 says "existed before a particular time" — so no pin can attest *when* a record was captured. For a dispute turning on "when did I learn X", the capture timestamp stays self-asserted.
   - **Store the complete proof, never the pending one.** Incomplete OTS proofs need a remote calendar to verify, and the default calendars are donation-funded; once a stamp confirms, upgrade it (`ots upgrade`) so a volunteer-uptime dependency becomes a one-time operation.
   
   **The pin choice is a genuine three-way trade with no default.** OpenTimestamps wins on trust model (no trusted third party) but requires a **local Bitcoin Core node to verify** and pends for hours; RFC 3161 returns immediately and has a protocol-level answer to certificate expiry and revocation, but substitutes a TSA as a trusted third party, and its tighter bound is bought with that same trust substitution (its `genTime` is the TSA's assertion about its own clock). Carrying **both** is the current suggestion — each covers the other's weakness — but the cost is the **union** of verification requirements (a Bitcoin node *and* a TSA chain *and* historical revocation data), so the ADR must state whether it optimises survivability or verifier burden, and must bind both anchors to the **same manifest root**.
   
   **Dedup needs a separate identity hash.** A Trillian leaf's Merkle hash changes when its timestamp changes, so dedup cannot key on it. footprint needs an independent identity hash so a re-submitted record is **rejected** rather than silently creating a second timeline entry — the same requirement as piler's dedup feature and the append-only constraint.
   
   Unsolved and worth stating: CA-root/trust-anchor distrust and hash-algorithm longevity affect both pins, and no project in any of the five surveys integrates external timestamping at all.
5. **Append-only storage.** No candidate documents append-only or tamper-evident storage at all.

**Note for #5's resolution.** The evidence and integrity material (gap items 4 and 5) is not really OSS survey content — it is design content, and it is where this survey overflows. Both the encryption and the archive-platform surveys independently ended up carrying it under "open questions" headings, and it accounts for roughly a fifth of the encryption memo. When #5 resolves, this is the part that wants writing up as ADRs rather than living in a literature review: app-level encryption of store *and* index, the AGPL process-boundary rule, the one-way store → manifest dependency, and the pin choice.

---

## Transcription: the honest state of the art

No OSS component is simultaneously (a) licensed for redistribution, (b) accurate on intra-sentence Malay/English switching, and (c) benchmarked on Malaysian speech. Each candidate satisfies at most two.

The only code-switching WER evidence found is research-only (AsyncSwitch, arXiv 2506.14190): off-the-shelf Whisper averages ≈21% WER on code-switched audio and ≈30% on monolingual Malay, with the best adaptation at ≈17% and no released checkpoint. Whisper is monolingual by design — one language token conditions the decoder per 30-second window — and a code-switching paper states outright that "Whisper was never intended to be a solution to code switching."

Two consequences for the design:

- **Test on your own audio.** No benchmark exists for this, so the choice must be settled by a spike on real Malaysian meetings: plain `large-v3-turbo` vs Mesolitica turbo-v3 vs Qwen3-ASR.
- **The transcript schema must carry per-segment language, not one document-level language field.** A single language field would hide exactly the failure mode that is known to exist.

Hardware is not the constraint: `large-v3` is ~3GB and `turbo` ~1.7GB in FP16, so an RTX 3050 8GB runs either. Defer diarization to v2 — the useful models are either gated (pyannote) or non-commercial (NeMo Sortformer).

---

## Encryption and the unattended-capture problem

**Recommended shape:** SQLCipher (or SQLite3MultipleCiphers) holding content, FTS5 index **and** sqlite-vec embeddings in one encrypted file, with a key hierarchy of Argon2id → root key → per-purpose content/blob/index keys, in the style of Standard Notes.

**The unsolved part, stated plainly:** a process that decrypts to capture must hold the key in memory. There is no clean answer. The best achievable is a wrapped master key in the OS credential store (libsecret / Keychain) so capture can run after login, which sets the security floor at the OS login session. That is a real weakening and belongs in an ADR, not a footnote.

**Known caveat:** the Python wheels for SQLCipher (`sqlcipher3-binary`) are Linux-x86_64 only — no macOS/arm64 — so a build or vendor step is required for a cross-platform product.

---

## Open questions

1. **Encryption vs durability.** ArchiveBox's goal of being readable in 50–100 years without the tool is in direct tension with store-level encryption. Which wins? Needs an ADR.
2. **Verifiable manifest.** Structure is settled (transparency log over ciphertext, one-way dependency, identity hash for dedup); the **pin choice is not** — OTS vs RFC 3161 vs both, and which of survivability or verifier burden wins. ADR item.
3. **The unattended key.** Is an OS-credential-store-wrapped key acceptable, or must capture require an interactive unlock each session?
4. **Teams.** Build the delegated `Chat.Read` capture path, or declare Teams out of v1 scope?
5. **Mesolitica's licence.** Blocked until declared. Worth asking the author.
6. **macOS packaging.** Does installing a macFUSE-based vault require admin on all supported macOS versions? (Unverified.)
7. **Unverified across the surveys:** exhaustive SQLCipher CVE check; whether Onyx's MCP server lives under `ee/`; libSQL's crypto; the MCP spec-repo and SDK licences; whether msgvault transcribes audio.

---

## Sources

Primary memos, each with its own citation list:

- [archive-platforms.md](oss/archive-platforms.md) — 28 candidates
- [capture-tooling.md](oss/capture-tooling.md)
- [encryption-store.md](oss/encryption-store.md)
- [transcription-malay.md](oss/transcription-malay.md)
- [agent-interface.md](oss/agent-interface.md)

Repository licences re-verified 2026-09-25 via the GitHub licence API for: msgvault, slackdump, Karakeep, Onyx, sqlite-vec, SQLCipher, whisper.cpp, mcp-memory-service, basic-memory, obsidian-mcp, SQLite3MultipleCiphers, Khoj, ArchiveBox, Got Your Back, Paperless-ngx, Timelinize, WhisperX, faster-whisper, WhatsApp-Chat-Exporter.
