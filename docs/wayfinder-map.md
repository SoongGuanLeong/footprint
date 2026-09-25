# Wayfinder Map: Personal Work Footprint Archive

> **Mirror of GitHub issue #1** ([SoongGuanLeong/footprint#1](https://github.com/SoongGuanLeong/footprint/issues/1)) — the issue is canonical.
> Synced 2026-09-25 (map **closed** — all six children closed, way to build clear). Edit the issue, then re-sync this file. Do not edit here.

> **This issue is the canonical map.** Mirrored in the repo at `docs/wayfinder-map.md`. Edit this issue, then re-sync that file — not the reverse.

> **Reached 2026-09-25.** The way to build is clear and no architecture-level decision remains open. See [Handoff](#handoff).

## Destination

Spec for a **distributable, participant-only Personal Work Footprint archive** — a product that other people install on their own local machine, capturing chats, meeting recordings/transcripts, and emails where the user is a participant, recipient, or attendee. Single-user, **one store per install**. Storage: local store encrypted **at the application level**; the host's disk encryption is not assumed. Inference and transcription are **local-only by default** — no content leaves the device on the default path, embeddings included. Exposes a **read-only, confined** surface that an external agent consumes (Hermes is a separate project and a consumer, not an engine). Stack: commonly used solutions in Malaysia. Also produces the Hermes suitability verdict and the Malaysia PDPA 2010 legal boundary memo. Map done when the way to build is clear and no open decisions remain.

> **Revised 2026-09-25** during the #5 grilling (Q1, Q4, Q6, Q7). Previously read "personal ... local encrypted single-user store". See the decision record comment on #5 for the reasoning.

## Notes

- Domain: personal data capture in employment context, Malaysia PDPA 2010, company policy.
- Skills per ticket: see ticket Type. Research tickets run via /research subagents. Grilling tickets run via /grilling + /domain-modeling.
- Glossary: **Personal Work Footprint** = all messages, transcripts, emails where you are participant. **Participant-Only Boundary** = only data your account could lawfully open.
- Purpose: personal recall + evidence. Not covert surveillance.
- Hermes target: https://github.com/NousResearch/hermes-agent
- Tracker: GitHub Issues. See `docs/agents/issue-tracker.md` Wayfinding operations.

## Decisions so far

<!-- one line per closed ticket, gist + link -->

- **#6 Spec scope and handoff shape** — the spec is a **tracked repo file** (overriding the earlier "specs live as issues" convention), with **10 sections plus an appendix** and boundary rules deliberately ordered *before* architecture; "way clear" is a **checklist plus a source-of-truth check**; the two spikes are `/to-tickets` output rather than map tickets; research is referenced and only its rules extracted. [decision record](https://github.com/SoongGuanLeong/footprint/issues/6#issuecomment-5830632332)
- **#7 Company policy checklist** — an **audit instrument** (clauses to look for, 20 questions for IT/HR, evidence to keep, a three-condition go/no-go gate, re-check triggers), distinct from #3's §10 which states what policy must cover. Generic by design, since each installer has a different employer. [checklist](https://github.com/SoongGuanLeong/footprint/blob/main/docs/policy-checklist.md)
- **#5 Architecture and storage decision** — distributable participant-only archive, one store per install, **application-level encryption** (SQLCipher + sqlite-vec, so the FTS index and embeddings sit inside the same cipher boundary) since the host's disk encryption is not assumed; **local-only** inference and transcription by default, embeddings included; **per-user daemon + CLI** with a **read-only, confined MCP surface over stdio** that fails closed when locked; **Windows first**, macOS second, Linux third; recall primary with evidence as an invariant. [decision record](https://github.com/SoongGuanLeong/footprint/issues/5#issuecomment-5829911933)
- **#2 Malaysia workplace stack survey** — participant-only capture is achievable via delegated per-user OAuth (Teams 1:1/group chat, M365 mail, Gmail, Google Chat, Slack, Zoom, Google Meet metadata); Teams channels, the Teams transcript/recording APIs, Teams Export, Gmail domain-wide delegation and `Mail.Read.All` sit outside the boundary; personal WhatsApp is a genuine gap (export-only). [memo](https://github.com/SoongGuanLeong/footprint/blob/main/docs/research/malaysia-workplace-stack.md)
- **#3 PDPA 2010 boundary** — participant-only is necessary but **not sufficient**: PDPA applies in full, there is no blanket employment exemption, no two-party-consent statute and no CMA s.234 participant carve-out. Defensible boundary = participant-only + overt + purpose-limited to own recall + no third-party disclosure + sensitive data excluded + local encrypted store in Malaysia + retention-limited + employer policy/DLP respected. [memo](https://github.com/SoongGuanLeong/footprint/blob/main/docs/research/pdpa-2010-boundary.md)
- **#4 Hermes agent audit** — **use, don't fork, build the store**: Hermes meets local run, transcription and extensibility, only *operates* (does not archive) connectors, and ships no at-rest encryption (plaintext SQLite). Pair it with a separate encrypted archive store, a participant-only capture filter, and own corpus ETL/index. [memo](https://github.com/SoongGuanLeong/footprint/blob/main/docs/research/hermes-agent-audit.md)

## Handoff

The map's job was to clear the way to build, and it is clear: no architecture-level question remains open, and both planning artifacts exist.

- **Spec** — `docs/spec.md`, a tracked repo file per the #6 decision, discussed on [#8](https://github.com/SoongGuanLeong/footprint/issues/8). Eleven sections plus two appendices: the ten decided in #6, plus a User stories section the spec template requires.
- **Build tickets** — [#9](https://github.com/SoongGuanLeong/footprint/issues/9)–[#23](https://github.com/SoongGuanLeong/footprint/issues/23), fifteen tracer-bullet tickets with native blocking edges. Frontier at close: #9, #10, #11, #12.
- **ADRs** — five. Four are [#12](https://github.com/SoongGuanLeong/footprint/issues/12); the external pin choice is an acceptance criterion of [#14](https://github.com/SoongGuanLeong/footprint/issues/14), because it cannot be decided before the pin is chosen.

### The five former gaps, and where they now live

| Formerly not specified | Now |
|---|---|
| Data retention duration and deletion policy | Spec §8 — keep everything by default, pruning is an explicit user action, deletion appends a tombstone. Ticket #17. |
| Search + recall UX | Spec §9 — full-text and semantic search, timeline, person-scoped retrieval, coverage. Tickets #16 and #18. |
| Transcription accuracy for Malay/English code-switching | Spec §2 — promoted to a **gating spike** that decides whether meetings ships in v1 or v1.5. Ticket #9. |
| Integration with company DLP / audit logging | Spec INV-11 plus the [policy checklist](https://github.com/SoongGuanLeong/footprint/blob/main/docs/policy-checklist.md) — the product surfaces the checklist rather than pretending to enforce it. |
| Export and portability | Spec §9, with open question A-12. Ticket #19. |

### What closing the map does not mean

Twelve open questions remain, each with an explicit trigger in the spec's Appendix A. They are **deferred by design, not unresolved** — each names the event that resolves it. The two spikes (#9, #10) are empirical work gating a scope choice, which is why #6 put them in the build tickets rather than the map.

## Out of scope

- Company-wide surveillance or data where user was not participant.
- Multi-user or team deployments: one store per install, one user per store.
- Real-time covert recording of others without consent or access.
- Full build implementation — planned in #9–#23 and tracked there, not here.
- Formal legal advice: memo is research, not counsel.

## Child tickets

- [x] #2 01 - Malaysia workplace stack survey (research) - closed
- [x] #3 02 - PDPA Malaysia 2010 boundary (research) - closed
- [x] #4 03 - Hermes agent capability audit (research) - closed
- [x] #5 04 - Architecture and storage decision (grilling) - closed
- [x] #6 05 - Spec scope and handoff shape (grilling) - closed
- [x] #7 06 - Company policy checklist (task) - closed

