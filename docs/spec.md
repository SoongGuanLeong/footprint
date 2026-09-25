# Personal Work Footprint Archive — v1 Specification

**Status:** buildable spec, ready for ticket breakdown.
**Date:** 2026-09-25.
**Canonical location:** this file. Per the [#6](https://github.com/SoongGuanLeong/footprint/issues/6) decision, the spec is a **tracked repo file** because it is a build input that needs diff history next to the code it describes. The issue tracker is the discussion surface, not the home of the spec.
**Source of truth check (per #6, Q17 item 5):** every decision in the [#5 decision record](https://github.com/SoongGuanLeong/footprint/issues/5#issuecomment-5829911933) (Q1–Q14) is reflected below, every still-open item is carried in [Appendix A](#appendix-a--open-questions-with-triggers) with a stated trigger, and nothing here contradicts a memo in `docs/research/`.

## Glossary

Terms used precisely throughout. Where a term is contested in the ecosystem, the definition here wins.

- **Participant-only** — capture is limited to communications the user was a genuine party to: a party to the conversation, a named recipient, or an invited attendee. Not "content the user could technically reach".
- **Record** — one captured unit: an email, a chat message, or a meeting. The atomic thing the archive stores and cites.
- **Canonical data** — the captured original bytes plus their capture metadata. Never mutated. Inside the manifest.
- **Derived data** — anything computed from canonical data: extracted text, transcripts, summaries, FTS entries, embeddings. Disposable and rebuildable from canonical data alone. **Outside** the manifest.
- **Store** — the single encrypted SQLite file holding canonical and derived data.
- **Manifest** — the append-only transparency log over ciphertext-derived leaf digests. Verifiable without a key.
- **Exclusion** — a **pre-ingest** filter. An excluded record never enters the archive, so it never reaches the index, the embeddings, or the manifest.
- **Tombstone** — a **post-ingest** append recording that an included record was pruned. The manifest grows; it never shrinks.
- **Supersession** — how a correction is represented: a new version linked to the original, never an in-place mutation.
- **Agent surface** — the read-only, confined MCP interface an external agent consumes.
- **Locked** — the store's key is not held by the daemon. Capture and query both stop.

---

## Problem statement

A person's working life happens in systems they do not own. The agreements, the decisions, the "we said we'd do X in March" — all of it lives in an employer's tenant, under an employer's retention policy, indexed by an employer's search, and reachable only while the employment lasts and the licence is paid. Meeting transcripts expire in weeks. Deleted messages fall out of export windows in days. When the user needs to recall what was actually said, their own memory is the only index, and it is the least reliable one available.

The specific pain is not "I want to keep everything". It is:

- **Recall.** "What did we actually agree about the migration date?" — answerable today only by scrolling a chat client or searching a mailbox that may no longer exist.
- **Continuity across jobs.** The record of work done is the employer's asset, not the worker's. Leaving means losing the ability to answer questions about one's own history.
- **Self-protection.** Being able to check what was actually said, rather than what someone remembers being said.
- **Privacy.** Any tool that solves this by shipping the user's mail and meetings to a cloud service has traded one exposure for a worse one. The user's own machine must be sufficient.

And it is a **product** problem, not a personal script: the user is the first user, but the point is that other people install it on their own machines and get the same thing. So the design cannot assume the first user's environment — in particular it cannot assume the host has disk encryption, because the reference host does not.

## Solution

A local, single-user archive that:

1. **Captures only what the user was a participant in**, through each provider's own delegated per-user consent — never a tenant-wide credential.
2. **Encrypts at the application level**, in one file, so the content, the full-text index and the embeddings share a single cipher boundary. The host's disk encryption is not assumed.
3. **Keeps everything on the device by default.** Inference, transcription and embedding are local. Cloud inference is an opt-in mode, never the architecture.
4. **Exposes a read-only, confined interface** to a local AI agent over stdio, so the user can ask questions about their own history in natural language.
5. **Is append-only and tamper-evident**, so the record cannot be silently rewritten — while being positioned honestly as a recall tool that happens to be tamper-evident, not as an evidence or legal tool.
6. **Fails closed.** The archive is session-scoped: it locks, and while locked both capture and query stop.

---

## User stories

**Installation and first run**

1. As a person installing footprint on my own machine, I want a per-user install that needs no administrator rights, so that I can install it on a work laptop without an IT request.
2. As a person installing footprint, I want the installer to work without a code-signing warning escalating into a block, so that I can complete the install without disabling a security feature.
3. As a first-run user, I want to set a passphrase and be told plainly what it protects and what happens if I lose it, so that I am not surprised later.
4. As a first-run user, I want to choose which sources to connect, so that I am not forced to grant access I do not want to give.
5. As a first-run user, I want to see exactly which permissions each connector requests and why, so that I can make an informed consent decision.
6. As a first-run user, I want to connect a source through my own delegated consent without an administrator approving anything, so that the product works in a workplace where I am not an admin.
7. As a first-run user, I want to be told when a source cannot be connected because my administrator has blocked the app, so that I understand the failure is policy, not a bug.

**Unlock and locking**

8. As the archive owner, I want to unlock the archive once per session with an interactive action, so that the key exists only while I am present.
9. As the archive owner, I want the archive to lock automatically on session end, so that a stolen or shared machine does not expose it.
10. As the archive owner, I want the archive to lock after an idle timeout, so that an unattended unlocked daemon during a long lunch is not a standing exposure.
11. As the archive owner, I want capture and query to stop while locked rather than queue for later decryption, so that nothing needs the key outside a session.
12. As the archive owner, I want the archive to never auto-unlock on boot and never remember my key, so that the security floor stays at my OS login session.
13. As the archive owner, I want a wrong passphrase to fail closed and clearly, so that I am never left believing a locked archive is readable.
14. As the archive owner, I want no run-while-logged-out mode, so that the key cannot outlive the session that unlocked it.

**Capture — email**

15. As the archive owner, I want to capture my own Microsoft 365 mailbox through delegated read access, so that my mail is archived without an admin.
16. As the archive owner, I want to capture my own Gmail through delegated read access, so that the same works on Google Workspace.
17. As the archive owner, I want capture to run on a schedule without my intervention, so that the archive stays current.
18. As the archive owner, I want the archive to capture eagerly rather than backfill on demand, so that content expiring in provider retention windows is not lost.
19. As the archive owner, I want to see per-source coverage — what is captured, up to when, and what is missing — so that I know how much to trust an empty search result.
20. As the archive owner, I want an incremental capture that resumes after an interruption, so that a crash does not require a full re-fetch.

**Capture — chat and meetings**

21. As the archive owner, I want to capture my own Teams 1:1 and group chats, so that my chat history is archived.
22. As the archive owner, I want my own meeting recordings and transcripts captured from files I already have access to, so that I get meeting content without an admin-gated API.
23. As the archive owner, I want to import a WhatsApp chat export manually, so that my most-used informal channel is at least partially covered.
24. As the archive owner, I want the product to be honest that WhatsApp cannot be automated, so that I do not wait for a connector that will never exist.
25. As the archive owner, I want Slack and Google Chat support to be clearly labelled as dependent on my administrator not blocking the app, so that I can predict whether it will work for me.
26. As the archive owner, I want capture to never include content I was not a party to, so that the archive stays within the boundary that makes it defensible.

**Transcription and language**

27. As the archive owner, I want my meetings transcribed locally, so that audio never leaves my machine.
28. As the archive owner, I want each transcript segment tagged with its own language, so that a Malay/English code-switched meeting is represented accurately rather than flattened to one language.
29. As the archive owner, I want raw audio to be deletable after transcription, so that I keep the text without keeping the most sensitive artifact.
30. As the archive owner, I want raw meeting audio flagged sensitive by default and withheld from the agent surface, so that voice data is not exposed by default.
31. As the archive owner, I want to know when a transcript is unreliable, so that I do not treat a bad transcript as an authoritative record.

**Recall**

32. As the archive owner, I want full-text search across my mail, chats and transcripts, so that I can find a specific thing I remember saying.
33. As the archive owner, I want semantic search, so that I can find things by meaning when I do not remember the words.
34. As the archive owner, I want a chronological timeline, so that I can reconstruct the order of events.
35. As the archive owner, I want to see every record involving a given person, so that I can reconstruct a relationship's history.
36. As the archive owner, I want to retrieve a full meeting transcript, so that I can read what was actually said.
37. As the archive owner, I want search to return the current version of a record by default, so that corrections are reflected without destroying the original.
38. As the archive owner, I want sensitive-flagged records withheld from agent answers unless I explicitly include them, so that a disclosure mistake is not the default.
39. As the archive owner, I want to exclude a record, a thread, or a person, and have that exclusion purge the index and embeddings too, so that "excluded" means gone from search, not just hidden from a list.
40. As the archive owner, I want to prune a record and have a tombstone recorded, so that deletion does not break the archive's tamper-evidence.
41. As the archive owner, I want retention to default to keeping everything, so that nothing is destroyed silently by a default I did not choose.
42. As the archive owner, I want a web UI on localhost, so that I can browse and review without a terminal.

**Agent surface**

43. As the archive owner, I want to ask a local AI agent questions about my own history, so that I get answers rather than search results.
44. As the archive owner, I want the agent to have read-only access only, so that an agent mistake cannot corrupt the archive.
45. As the archive owner, I want the agent surface to expose no web search, no URL fetch and no filesystem path parameter, so that a read-shaped tool cannot become an egress path.
46. As the archive owner, I want the shipped agent configuration to be restrictive by default, so that the safe posture does not depend on me writing config correctly.
47. As the archive owner, I want the agent to fail clearly when the archive is locked, so that I know to unlock rather than believing I have no data.
48. As the archive owner, I want long transcripts paginated or served as resources, so that a multi-hour meeting is not silently truncated.
49. As the archive owner, I want a single tool call never to return more than the transport can carry, so that I am never shown a truncated answer as if it were complete.

**Trust, evidence and honesty**

50. As the archive owner, I want the archive to be append-only, so that the record cannot be quietly rewritten.
51. As the archive owner, I want to verify the archive's integrity without needing my passphrase, so that verification is possible in situations where the key is unavailable.
52. As the archive owner, I want each record stamped at capture time, so that timestamps reflect when I received it, not when I re-processed it.
53. As the archive owner, I want the archive's integrity anchored outside the archive, so that whoever holds the write key cannot rewrite history and re-sign it.
54. As the archive owner, I want the product to not call itself an evidence tool or a legal tool, so that I do not rely on it in a way it cannot support.
55. As the archive owner, I want the product to say "sensitive-data flagging with review", never "sensitive data excluded", so that the product does not make a compliance claim it cannot meet.

**Privacy and egress**

56. As the archive owner, I want nothing to leave my machine by default, so that the archive's privacy does not depend on a vendor's terms.
57. As the archive owner, I want embeddings computed locally, so that semantic search is not an egress path.
58. As the archive owner, I want cloud inference to be an explicit opt-in, so that I can choose convenience over privacy knowingly.
59. As the archive owner, I want any cloud mode to be labelled as risk-reducing, not risk-eliminating, so that I am not sold a false guarantee.
60. As the archive owner, I want no enrichment feature that sends selected content to configured endpoints, so that the msgvault failure mode is not reproduced.

**Operations, portability and honesty**

61. As the archive owner, I want to export my archive in a documented format, so that I am not locked into the tool.
62. As the archive owner, I want to know what travels with me if I leave a company, so that I understand the boundary of my own record.
63. As the archive owner, I want a company-policy checklist surfaced in the product, so that I can confirm my own use complies with my employer's rules rather than assuming it does.
64. As the archive owner, I want the product to tell me when a source cannot be captured rather than failing silently, so that a gap is visible.
65. As the archive owner, I want diagnostics that do not themselves leak content, so that I can get help without exposing my archive.
66. As the archive owner, I want to uninstall cleanly, so that removing the tool does not leave an orphaned daemon or scheduled task.
67. As a prospective installer, I want the same product on Windows, macOS and Linux, so that I can install it on the machine I actually use.
68. As a prospective installer, I want a release order I can plan around, so that I know when my platform is supported.

---

## 1. Purpose, non-goals and positioning

**Purpose.** Personal recall over the user's own working communications — mail, chat and meetings — on the user's own machine, with an AI agent able to answer questions over it.

**Positioning, stated deliberately because it constrains features.** footprint is a **personal recall tool that happens to be tamper-evident**. It is not an evidence tool, not a legal tool, not a compliance product, and not a surveillance system.

The reason this is a spec constraint and not marketing copy: the [PDPA memo](https://github.com/SoongGuanLeong/footprint/blob/main/docs/research/pdpa-2010-boundary.md) is explicit that the archive is single-user **by design**, not inaccessible **by right**, and that a "private dossier" framing is legally mistaken. Positioning therefore drives three concrete requirements:

- No feature may be justified by "it would hold up in a dispute". Features are justified by recall.
- The product must never make a compliance claim it cannot meet. See §3, INV-9.
- Evidence properties are **invariants the architecture must satisfy**, not features to market. See §8.

**Non-goals.**

- Company-wide or multi-person surveillance, in any form.
- Capturing content the user was not a participant in.
- Covert recording.
- Real-time monitoring or alerting on other people.
- Legal advice, or any output that reads as legal advice.
- Being a service to third parties (see INV-10 and §3).

## 2. Scope and phasing

### What v1 is

**Email is in v1 and never waits on anything.** Email is the unconditional first source because it exercises the whole architecture — application-level encryption of store, index and embeddings; the participant-only filter; the read-only confined agent surface; the manifest — with **no transcription dependency**. A disappointing v1 can therefore be attributed to the architecture rather than to transcripts. It also has the cleanest delegated capture path, is text-only so the schema stays simple, and carries the most defensible evidence value: written records where the user is a participant or recipient, with no consent question about recording another person.

**Meetings join v1 if and only if both spikes pass before feature freeze.** Either spike failing or being inconclusive moves meetings to **v1.5**, with the failure mode recorded rather than hidden. If transcription accuracy is the blocker, meetings may still ship with a reduced proposition — **audio, metadata and the user's own notes, rather than relying on transcripts**.

**Chat is v2 regardless**, and enterprise-only. Personal WhatsApp is a **permanent manual-import feature, not a connector** — there is no API for a user's own consumer WhatsApp history, only a per-chat media-stripped `.txt` export.

### The two spikes

The spikes are the **decision procedure** for the meetings scope, not a separate workstream, and they run from day one so they never block email.

1. **Transcription accuracy on real audio.** Compare plain `large-v3-turbo`, Mesolitica turbo-v3 and Qwen3-ASR on actual Malaysian meetings. No benchmark exists for Malay/English code-switching, so this must be empirical. Evidence to beat: off-the-shelf Whisper ≈21% WER code-switched and ≈30% monolingual Malay; the best published adaptation ≈17% with no released checkpoint; and the best technical fit (Mesolitica) has an **undeclared weight licence**.
2. **Teams participant-only capture against a real tenant.** The largest single unknown in the capture layer. No OSS path exists, Microsoft's only bulk export needs admin consent, and the route this spec relies on — the user's own recordings and transcripts as ordinary OneDrive/SharePoint files via delegated `Files.Read` / `Sites.Read.All` — **needs validation against a real tenant**. Teams is the likely meeting surface in a Malaysian M365 shop, so this gates meetings.

**What would override this:** a decision that meetings must ship in v1 **regardless of the spike outcomes**. That means shipping transcripts of unverified accuracy, and under the PDPA posture **a bad transcript is worse than no transcript** — it is a record that looks authoritative and is not.

### Platform scope

**All three platforms are in v1 scope. Release order Windows → macOS → Linux**, designed for three from day one with CI on all three. Sequencing is driven by distribution economics, not code, because the stack has **no FUSE dependency anywhere** — SQLCipher, sqlite-vec, llama.cpp/Ollama and Whisper are in-process libraries or ordinary userspace processes.

- **Windows first, and it is not close** — ~70.4% of Malaysian desktops against macOS 9.6% and Linux 3.2% (StatCounter 12-month average). Caveat recorded: these are consumer web-traffic figures, not enterprise, and **no credible Malaysia-specific enterprise desktop split exists in public sources**. Windows 10 is still ~14% of Malaysian desktops, so **target 10 as well as 11**.
- **macOS second — for reach, not ease.** It is the only platform with a mandatory non-bypassable recurring gate (US$99/yr, notarisation, Hardened Runtime, and a Mac in the build loop), and where the stack is weakest: CTranslate2 has **no GPU path on macOS**.
- **Linux last** despite being the development host — least reach, but no distribution gate and no recurring cost, so it rides along as a tarball or AppImage without a dedicated release effort.

## 3. Boundary rules as testable invariants

These are written as invariants rather than prose, because they are conditions the architecture **must satisfy** rather than properties it happens to have. They derive from the [PDPA memo's](https://github.com/SoongGuanLeong/footprint/blob/main/docs/research/pdpa-2010-boundary.md) boundary statement B1–B10. The memo is referenced, not reproduced: it is explicitly **not legal advice**, and reproducing it in a spec would invite it to be read as a legal position.

| # | Invariant | Enforcement | How it is tested |
|---|---|---|---|
| **INV-1** | **Genuine participation.** Every captured record is one the user was a party to, a named recipient of, or an invited attendee of. | Capture-time filter applied per source; connector scopes are chosen so the API cannot return more. | For each connector, a fixture set containing non-participant content (forwarded, overheard, third-party) produces zero records. |
| **INV-2** | **Overt collection.** Capture is never concealed from the other participants. | **Behavioural, not enforceable in code.** The product nudges: recording flows require an explicit acknowledgement, the overt posture is stated in the UI and docs, and no stealth mode is built. | Documented as a non-enforceable condition; asserted by the absence of any concealed-capture feature and by an acknowledgement gate on recording import. |
| **INV-3** | **Purpose limitation.** The archive is used for the user's own recall only — never to monitor, evaluate, rank or build a dossier on another person. | No feature exists that aggregates a person into a score, ranking or report; `list_people` returns records, never assessments. | No output type in the data model or surface can express a judgement about a person. |
| **INV-4** | **No third-party disclosure.** No sharing, publishing, forwarding or access-granting, except where compelled by law or required for a valid access/correction request. | The agent surface is read-only **and confined** (no web search, no URL fetch, no filesystem path parameter). No export-to-third-party integration. | Surface conformance test: the advertised tool set contains no egress-capable tool; a tool that can reach an arbitrary URL does not exist. |
| **INV-5** | **Sensitive personal data excluded or explicitly consented to.** | Sensitive-flagged records are withheld from the agent surface by default (§9). Manual exclusion is authoritative. | With a sensitive-flagged record present, agent answers exclude it unless the include flag is set. |
| **INV-6** | **Local, encrypted, single-user storage.** Store remains in Malaysia; encrypted at rest; access limited to the user; no cloud sync, no foreign backup, no cross-border transfer. | Application-level encryption of store **and** index; no sync or backup feature exists. | The raw store file contains no plaintext for any ingested content (T-3). |
| **INV-7** | **Retention limited to the purpose, with deletion honoured.** | Retention defaults to keep-everything with pruning an explicit user action; deletion appends a tombstone rather than mutating. | Pruning a record appends exactly one tombstone and removes it from search; no default expiry exists to fire. |
| **INV-8** | **Data-subject rights honoured.** Access and correction requests from other participants can be answered. | Search by person and by date range is supported, so the user can produce what is needed. | A person-scoped query returns the complete set for that person, excluding nothing that would be required to answer the request. |
| **INV-9** | **No overclaiming.** The product never states that sensitive data is excluded, that the archive is compliant, or that it is legal evidence. | Wording is fixed: "sensitive-data flagging with user review; the agent surface excludes flagged content by default". | Shipped strings are asserted in test; no claim-bearing string ships without review. |
| **INV-10** | **Not a service to others.** Each install is its own single-user archive. | **Refinement for a distributable product:** one store per install, no shared store, no hosted service, and **no data flowing between installs**. Being installable by others is not being a service to them. | The product has no server component and no inter-install channel; each install's store is independently encrypted with its own key. |
| **INV-11** | **Employer policy and DLP classification respected.** | The product surfaces the [company-policy checklist](https://github.com/SoongGuanLeong/footprint/blob/main/docs/policy-checklist.md) rather than pretending to enforce it. | The checklist ships in-product; the go/no-go gate is presented to the user before capture is enabled. |
| **INV-12** | **Covert archives are out of scope.** | Consequence of INV-2. No concealed-capture capability is built, ever. | Absence assertion, as INV-2. |

**Where the boundary is weakest, and what the spec does about it.** The conditions most likely to fail in practice are **INV-2 (overt collection)**, **INV-4 (no disclosure)** and **INV-11 (employer policy)**. Of these, **INV-2 cannot be engineered around** — it is a behavioural condition. The product's honest position is to nudge and document it, and to make INV-4 and INV-11 structural so that the two that *can* be enforced are enforced.

## 4. Architecture, storage and key lifecycle

### Components

- **`footprint` — the CLI.** Unlock prompt, capture control, import, exclusion, pruning, export, verification, and the MCP entry point. Required regardless of anything else: it is the debugging surface and the only place a passphrase is entered.
- **`footprintd` — the per-user daemon.** Holds the unlocked key, runs scheduled capture, manages local inference, and owns the store. Not a choice: the agent surface needs a process holding the key to forward to, inference needs managing, and capture runs on a schedule.
- **`footprint mcp` — the stdio MCP server.** Launched by the agent, forwards to the daemon over a local socket. **Never unlocks the store itself and never prompts for the key.** If the daemon is not running or the store is locked it **fails closed**. One key path, one unlock.
- **Capture workers** — in-daemon, scheduled per source.
- **Local inference** — LLM, speech-to-text and embeddings, all local by default.
- **Store** — one encrypted SQLite file.

**Why the MCP server must not hold a key.** A standalone MCP process that unlocked the store itself would create a second key path and a second place to get it wrong. Keeping the key in exactly one process is what makes the session-scoped posture enforceable rather than aspirational.

### Storage and the cipher boundary

**One file.** SQLCipher (BSD-3-Clause) with sqlite-vec (Apache-2.0), so that content, the FTS5 index and the embeddings all sit inside the **same cipher boundary**. This is the product's actual differentiator: the closest OSS product, msgvault, documents that its SQLite database is not encrypted and that its mitigation is to rely on OS-level full-disk encryption — the one assumption footprint cannot make, since the reference host has a plain ext4 root, no LUKS, and plain unencrypted swap.

**Inside the cipher boundary:** canonical record content, capture metadata, extracted text, transcripts and their per-segment language tags, summaries, FTS5 entries, embeddings, the sensitive-data flag, and OAuth tokens.

**Outside the cipher boundary, deliberately:** the manifest. It holds **ciphertext-derived digests**, so verification needs no key and reveals nothing about plaintext. This is a hard requirement, not an optimisation: an unkeyed plaintext digest manifest is itself a confirmation oracle (Halevi et al., CCS 2011) — anyone holding it could confirm whether a guessed message is in the archive.

**Dependency direction is one-way: store → manifest.** The manifest is derived from the store. Nothing in the store depends on the manifest, so the manifest can be rebuilt from the store, and a corrupt manifest can never make the archive unreadable. This is ADR material.

**Schema constraints.** Pin SQLite ≥ 3.42 for sqlite-vec, and prefer the `k =` form over `LIMIT` in vector queries. `sqlcipher3` 0.6.2 ships prebuilt wheels for Linux, macOS (universal2/x86_64/arm64) and Windows (win32/amd64/arm64), statically bundling SQLCipher 4.x, so **no build toolchain is needed on any platform**.

### Key hierarchy

```
passphrase ──Argon2id──▶ root key
                          ├──▶ store key      (SQLCipher; content, FTS5, embeddings)
                          └──▶ identity key   (keyed dedup hash — see §5)
```

- The **root key** is wrapped and stored in the OS credential store. The best achievable for unattended capture is a wrapped key in the OS credential store, which **sets the security floor at the OS login session**. The archive accepts that floor explicitly rather than pretending to exceed it. This is ADR material, and it is the reason the session rules below exist.
- The **store key** encrypts the file. SQLCipher's own page-level key derivation sits beneath it.
- The **identity key** is separate from the store key because the dedup identity hash must be stable across re-processing, while a manifest leaf's Merkle hash changes when its timestamp changes.

### Process topology and session rules

1. **Unlock requires an interactive action each session** — via the CLI at login. No "remember me", no persisted passphrase, no auto-unlock on boot.
2. **The daemon drops the key on session end, and on an idle timeout.** Both are needed: session end alone leaves an unattended unlocked daemon during a long lunch.
3. **Capture and query fail closed while locked.** No queue-and-decrypt-later behaviour, because that would require holding the key outside a session.
4. **Do not enable run-while-logged-out.** On Linux that means *do not use `loginctl enable-linger`* — it is user-grantable and therefore tempting, but it puts the key outside a session and contradicts rules 1–3.
5. **No remote-specific bypass or convenience mode.** A remote session is just a session.
6. **The application is not the mitigation for remote access.** On a host with no disk encryption and remote-access paths active, the real mitigations are application-level encryption and not leaving the daemon unlocked while unattended. Remote-access hardening is a host-level concern.

**Recorded non-technical risk:** using a remote-access tool from a work laptop may be flagged by the employer's DLP. That is an **employment risk, not a technical one**, and the mitigation is the overt, participant-only posture (§3), not anything in the software.

### Why the daemon is viable without admin

A daemon plus CLI is viable with **no admin on all three platforms, provided it only runs while the user is logged in.** That proviso is free here because the key is **session-scoped**. The same fact disqualifies the Windows Service route on **functional** grounds, not merely the admin one: a session-0 service cannot hold a key unlocked in the user's interactive session.

## 5. Data model

### Record types

- **Message** — one email or one chat message. Canonical fields: source, provider, conversation identifier, participants, author, sent time, received time, subject or thread, body, attachment references.
- **Meeting** — one meeting. Canonical fields: source, organiser, attendees, start/end, join link, and references to recordings, transcripts and the user's own notes.
- **Transcript segment** — a derived unit of a meeting, carrying `start`, `end`, `text`, `speaker` (where available) and **`language` per segment**.
- **Attachment** — canonical bytes plus a reference from the owning record.
- **Person** — a derived grouping over participants. Never a scored or ranked entity (INV-3).

### Canonical versus derived

The distinction is load-bearing and everything else follows from it:

- **Canonical** data is the captured original plus capture metadata. **Never mutated.** Inside the manifest.
- **Derived** data is everything computed from it: extracted text, transcripts, summaries, FTS entries, embeddings. **Disposable and rebuildable from canonical data alone.** **Outside the manifest.**

Consequence: re-indexing, re-embedding and re-summarising can happen freely without touching integrity. This is the Maildir + notmuch property identified in the OSS survey, and it is what lets recall be aggressive without endangering evidence.

### Supersession and versions

**Records are never mutated in place.** A re-transcription or a correction is a **new version that supersedes**, linked to the original. Search returns the **current** version by default. This gives good recall results while destroying nothing.

### Per-segment transcript language

Transcripts carry **per-segment** language, not a document-level field, because a Malaysian meeting code-switches mid-sentence. A document-level language field would misrepresent exactly the meetings this product targets.

### Exclusion versus tombstone

Two operations that look similar and must not be conflated:

- **Exclusion is a pre-ingest filter.** An excluded record **never enters the archive**, so it never reaches the index, the embeddings, or the manifest. Presence in the manifest would leak the record's existence even with the content gone.
- **Pruning is a post-ingest action on an included record.** It appends a **tombstone**, so the manifest grows and never shrinks, and the archive can still prove it was not tampered with.

Without that distinction the sensitive-data rule (§9) and the recall/evidence weighting (§8) would silently contradict each other.

### Deduplication

A **separate identity hash**, keyed with the identity key, is used for dedup — not the manifest leaf hash. A leaf's Merkle hash changes when its timestamp changes, so it cannot double as a stable identity.

### Sensitive flag

The sensitive flag is **metadata inside the encrypted store** and **must never reach the manifest in plaintext**. The flag itself is sensitive: knowing that a record was flagged discloses something about its content. This ties it to the ciphertext-derived leaf constraint in §8.

## 6. Capture

### The rule

Every connector uses **delegated per-user OAuth**. Never tenant-wide application permissions, never a service account, never an administrator grant. Where a platform requires an admin grant, the connector is **out of scope by definition** and must not be designed around.

### Recommended connectors — delegated, no admin consent required by the platform

| # | Source | Delegated path |
|---|---|---|
| 1 | **Microsoft Teams 1:1 and group chats** | Graph `/me/chats` and `/me/chats/{id}/messages`, delegated `Chat.Read`. Paginate; the delta endpoint is application-only. |
| 2 | **Microsoft 365 mail** | Graph `/me/messages`, delegated `Mail.ReadBasic` or `Mail.Read`. |
| 3 | **Teams meeting artefacts, via files** | Read the transcript or recording file from the user's own OneDrive, or a SharePoint site they belong to, with delegated `Files.Read` / `Files.Read.All` / `Sites.Read.All`. **Deliberately avoids the admin-gated transcript API.** Requires spike 2. |
| 4 | **Gmail** | Gmail API `users.messages.*` with `gmail.readonly`. Restricted-scope verification applies. |
| 5 | **Zoom** | User-managed OAuth app with `cloud_recording:read:list_recording_files`, `cloud_recording:read:list_user_recordings`, `cloud_recording:read:meeting_transcript`. |
| 6 | **Google Meet** | `meetings.space.readonly` for conference records, participants, transcript entries and smart notes. |

### Conditionally inside — no platform admin consent, but an administrator can block the app

| # | Source | Path | Caveat |
|---|---|---|---|
| 7 | **Google Chat** | `chat.messages.readonly`, `chat.spaces.readonly` with user credentials | Subject to Workspace Admin-console API controls |
| 8 | **Slack** | `conversations.history` with a user token and the `:history` scopes | Subject to the workspace "Approve apps" setting. **If built, `slackdump` is invoked as a separate unmodified process, never linked** — it is AGPL-3.0 and footprint is distributed (see the AGPL process-boundary ADR). |

The asymmetry is real and must be presented as such: neither platform has a per-permission admin-consent flag, so these verdicts rest on the OAuth model plus the fact that an administrator *can* intervene — not on an explicit "no admin consent required" statement.

### Manual import, inside the boundary

- **Personal WhatsApp** — on-device per-chat **Export chat** to `.txt`. The only mechanism Meta documents for a user's own history. Media arrives as sharesheet attachments that cannot be re-imported. **This is the single largest uncovered surface for a Malaysia-focused archive, and it is a permanent manual-import feature rather than a connector.** The product must say so rather than implying a connector is coming.
- **Telegram** — Desktop "Export Telegram data" to JSON or HTML, imported as a file.

### Explicitly outside the boundary — do not design around these

Teams channel messages (`ChannelMessage.Read.All`); the dedicated Teams transcript/recording APIs (`OnlineMeetingTranscript.Read.All`, `OnlineMeetingRecording.Read.All`); Teams Export APIs; Teams chat delta; Gmail domain-wide delegation; `Mail.Read.All`; WeCom session-content archiving; Zoom Server-to-Server OAuth. Each requires an administrator grant, an enterprise licence, or an application-level credential.

### Capture cadence — driven by provider retention ceilings

**Capture eagerly; never rely on backfill.** Provider retention windows are short and asymmetric, and an archive meant as a long-term record cannot recover what expired:

- Google Meet transcript entries expire **30 days** after the conference.
- Teams export reaches deleted messages for only **21 days**, and deleted users or teams for **30 days**.
- WhatsApp's business history sync covers **180 days** (and is not a substitute path anyway).

Consequence for the design: the daemon polls on a schedule short enough to stay inside the tightest window of each connected source, and an incremental capture that resumes after interruption is required, not a nicety. A source that has not been captured inside its window is reported as a **coverage gap** (§9, `coverage`) rather than silently missing.

### Credential handling

OAuth tokens live **inside the encrypted store**, never in plaintext files. This is a direct correction of the closest OSS product, which stores OAuth tokens as plaintext JSON.

## 7. Surface specification (MCP)

### Transport and revision

**MCP over stdio.** No listener, no port, no OAuth — the consumer implements stdio natively. **Target protocol revision `2025-11-25`.** The consumer's `auto` negotiation tries the classic `initialize` handshake first and only falls back to the `2026-07-28` `server/discover` probe on a modern-only signal, so targeting a handshake-era revision is the zero-friction path. `2026-07-28` also works but only via the discover fallback — unnecessary risk for no benefit.

### The tool set

**Small, fixed, and hand-written. Never reflected from a CLI or an ORM.** The ArchiveBox failure is the precedent: dynamic Click introspection exposed shell and crawl through what was meant to be a read surface.

| Tool | Purpose | Notes |
|---|---|---|
| `search` | Full-text and semantic search across records | Returns current versions by default; sensitive-flagged records excluded unless explicitly included |
| `get_record` | One record by identifier, with its metadata and version chain | Paginated |
| `timeline` | Chronological records over a window | Paginated |
| `list_people` | Participants seen in the archive, with record counts | **Returns records, never assessments** (INV-3) |
| `get_transcript` | A meeting transcript | **Must paginate** — see below |
| `coverage` | What is captured, up to when, and what is missing per source | Makes a gap visible rather than silently absent |

**Resources are used for large transcripts**, in addition to tools: a resource is the natural fit for a multi-hour transcript that cannot be a single tool result.

### Confinement rules

Read-only is **not** the same as confined, and both are required — they cover different axes:

- **Read-only** scopes *which operations*: no write tools registered, plus a **read-only database role**. Two controls, because the database role holds even if a tool is wrong.
- **Confined** scopes *which reach*: **no write tools, no SQL passthrough, no web-search tool, no arbitrary-URL fetch, no filesystem path parameter.** A non-mutating surface can still be an unbounded egress path — the lesson from Onyx, whose read-shaped MCP server nonetheless exposes web search and full-page fetch — which would collide with INV-4.

**The client is not part of the boundary.** The consumer does **not** filter by `readOnlyHint`; it captures the annotation only for approval gating on `trust: untrusted` servers and for retry safety, and its **default trust is `full`, where write-capable tools run with no approval at all**. Its own config reference calls the hint untrusted. So footprint's guarantees must be server-side and unconditional, and the shipped config below is what raises the client side to match.

### Pagination is a hard constraint

A **2,000,000-character cap** applies per tool result over stdio (not only HTTP), with head+tail truncation above it. A multi-hour meeting transcript returned as a single result **will** be truncated. Therefore:

- `get_transcript` and any large record retrieval **must paginate**, or return a handle the agent reads in chunks.
- A truncated result must never be presented as complete.

### Shipped configuration

**The agent configuration is a product deliverable, not documentation.** Leaving the user to write it would make the shipped default posture weaker than the design assumes.

```yaml
mcp_servers:
  footprint:
    command: "footprint"
    args: ["mcp"]
    env:
      FOOTPRINT_SOCKET: "${HOME}/.local/share/footprint/daemon.sock"
    cwd: "${HOME}"
    trust: untrusted
    tools:
      include: ["search", "get_record", "timeline", "list_people", "get_transcript", "coverage"]
    sampling:
      enabled: false
    elicitation:
      enabled: false
```

**Declare every needed path in `env:`.** The stdio child's environment is deliberately filtered to a safe baseline — `PATH`, `HOME`, `USER`, `LANG`, `LC_ALL`, `TERM`, `SHELL`, `TMPDIR`, plus `XDG_*` and the config's own `env` — so an undeclared socket path **fails at runtime after starting successfully**. That confusing failure mode is designed out by shipping the config above and validating the socket path at startup.

### Fail-closed behaviour

When the daemon is not running, or the store is locked, the MCP server returns a **clear, structured error** and never attempts to unlock, prompt for a key, or queue work. A locked store is an expected, designed state rather than an error condition.

## 8. Evidence model and retention

### Weighting

**Recall is the purpose. Evidence is an invariant it must not violate.** Where they conflict, **evidence wins on invariants** — append-only, no silent mutation, capture-time timestamps, the manifest — and **recall wins on features**.

The four conflicts, resolved:

- **Re-indexing** — the index, embeddings and summaries are derived data: disposable, rebuildable from the store alone, and outside the manifest (§5). Recall may rebuild freely.
- **Mutation** — never in place. A correction is a new version that supersedes (§5).
- **Pruning** — retention defaults to **keep everything**; pruning is an explicit user action and never a default expiry, because a default expiry would destroy evidence silently — the exact failure this weighting exists to prevent.
- **Deletion is itself append-only** — pruning appends a tombstone, so the manifest grows and never shrinks.

### Structure

- **A transparency log (RFC 6962 / Trillian) over ciphertext.** Leaves are **ciphertext-derived digests**, so verification needs **no key** and leaks nothing.
- **The external pin is load-bearing.** A single-user archive has **no independent monitors**, so the same key that writes the archive could otherwise rewrite history and re-sign a head. Pinning the head outside the archive is what makes tamper-evidence meaningful rather than decorative.
- **Stamp at capture time.** Both OpenTimestamps and RFC 3161 bound existence from above only, so the timestamp must be taken when the record is captured, not when it is processed.
- **Store the complete proof**, not just the root.
- **Separate identity hash for dedup** (§5), because a leaf's Merkle hash changes with its timestamp.

### Pin choice — open

**OpenTimestamps** wins on trust model but needs a local Bitcoin Core node to verify and pends for hours. **RFC 3161** returns immediately and handles revocation but substitutes a TSA. Carrying both costs the union of verification requirements. There is **no default**; see [Appendix A](#appendix-a--open-questions-with-triggers).

### What the product must not claim

Because recall is primary, the product **must not market itself as an evidence or legal tool**. The honest positioning is a personal recall tool that happens to be tamper-evident (see INV-9 and §1).

## 9. Recall UX

### v1 — minimal

The CLI plus the agent surface. Search, timeline, person-scoped retrieval, transcript retrieval and coverage are all reachable through the six MCP tools, so v1 needs no bespoke UI to be useful: the user asks their agent.

- **Full-text search** over mail, chat and transcripts.
- **Semantic search** using a **local** embedding model — constrained to local by the egress decision, so a cloud embedding API is not an option.
- **Timeline** over a window.
- **Person-scoped retrieval** — which also satisfies INV-8, since it is what lets the user answer an access request.
- **Transcript retrieval**, paginated.
- **Coverage reporting**, so an empty result can be distinguished from an uncaptured source.

### Sensitive-data behaviour in the surface

The restrictive default sits on the **agent surface**, not at ingest — that is where disclosure risk lives, because a mistake there is least recoverable. Sensitive-flagged records are **not returned to agents unless explicitly included**. Manual exclusion, this surface default, and the documentation ship in v1; the **assistive classifier is v2**, because flagging is cheap but a classifier that creates false confidence is not free, and v1 does not need it to be honest.

### v2 — localhost web UI

Served by the daemon. It becomes the natural home for unlock, which is what brings it forward rather than aesthetics. Four non-negotiable requirements:

1. **Bind `127.0.0.1` only.**
2. **Require a per-session token.**
3. **Validate `Host` and `Origin` headers to defeat DNS rebinding** — otherwise any page the user visits can reach localhost, which is why "it's only localhost" is not a security argument.
4. **Run the listener only while the UI is open.**

**The agent path deliberately stays stdio with no listener.** Two surfaces, two exposure models, chosen on purpose.

### Remote sessions

**A remote session is an ordinary session.** No remote-specific bypass or convenience mode. Whether to harden specifically against remote sessions — re-unlock on a detected remote session, or suppress the UI listener during one — is an open question in Appendix A; the recommendation is to accept them for v1.

## 10. Distribution

### Windows first

**v1 ships an unpackaged per-user install** — NSIS/Inno/WiX per-user, or a portable ZIP. Microsoft's guidance supports this, since unpackaged apps are "fully unrestricted" in process model and filesystem access.

**Auto-start: a per-user Task Scheduler task with an "At log on" trigger and the interactive logon type** — the only no-admin mechanism with crash supervision. Trap to design around: `TASK_LOGON_PASSWORD` or `TASK_LOGON_S4U` **silently will not launch for a non-admin**, because they require the "Log on as batch job" privilege, which is admin-granted and off by default.

**MSIX is not relied on for v1, though it is not disqualified by policy.** There is no Store clause against background, headless or CLI operation; `windows.startupTask` works for full-trust desktop-bridge apps; and long-running no-UI processes are explicitly not subject to UWP suspension. But two OS behaviours the documentation does not guarantee hit this design precisely:

1. **Whether a full-trust MSIX app's writes to a user-chosen store directory are virtualized per-package** (invisible to the unpackaged MCP agent) or pass through. Microsoft's docs **conflict**, and the only opt-out, `unvirtualizedResources`, requires Store approval that "in most cases won't be approved".
2. **Whether stdin is forwarded through an app execution alias.** stdout is documented; stdin is not, and a concrete counter-example exists of a packaged console app hanging in `fgets()` via its alias while the same executable works from its install path. **For an MCP stdio server, that is exactly the failure mode.**

A middle path worth evaluating later is a **sparse package** ("packaging with external location"), which keeps the per-user installer and registers a lightweight identity package to unlock `startupTask`. It does not resolve the stdin question, and non-Store distribution still needs a certificate.

### Code signing

Because v1 is unpackaged, the code-signing path is **the v1 plan, not a fallback** — MSIX would have got signing free from the Store.

**Azure Artifact Signing is geo-fenced away from Malaysia** (organisations USA/Canada/EU/UK; individuals USA/Canada only). That matters because it is the only option that is both cheap (~US$9.99/month) *and* exempt from the CA/Browser Forum hardware-crypto-module mandate.

| Route | Cost | Notes |
|---|---|---|
| **SSL.com IV** | US$129/yr | Issued to an Individual Applicant on a government photo ID |
| **Certum Standard** | ~€139–209/yr | Cloud signing included |
| **SSM sole proprietorship + OV** | RM30–60/yr + OV cost | Makes the OV route unambiguous; **a Sdn Bhd is not needed** (RM2,000–5,000+ in year one, and it buys nothing) |
| DigiCert / GlobalSign | — | **Require an organization** and will not issue to an individual |

Under CA/B Forum ballot CSC-31, certificates issued from 1 March 2026 cap at **460 days**, so this is permanently an annual cost. A hardware token or paid cloud HSM is also required on the OV route.

### SmartScreen — pre-purchase checklist item

**No certificate removes the first-download warning.** Since 2024 EV no longer bypasses SmartScreen — Microsoft's own words. A certificate removes "Unknown Publisher" and lets reputation accumulate across versions; **only the Microsoft Store gives zero warnings.**

- [ ] **Obtain written confirmation from the CA that an individual-validated (non-EV) certificate accumulates SmartScreen publisher reputation before paying.** The reasoning is high-confidence — Microsoft's Root Program lists one code-signing OID and reputation keys to the certificate identity — but **Microsoft does not document it**. This is a purchase precondition, not a design question, which is why it is a checklist item rather than an open question.
- [ ] Decide the CA and purchase, given CSC-31's 460-day cap and the annual renewal it implies.

### macOS (v2)

**A LaunchAgent in `~/Library/LaunchAgents`** — no admin, with `RunAtLoad` + `KeepAlive` for start-at-login and crash restart. **macOS 13+ requires the user to approve the background item** in System Settings > Login Items, and distribution forces Developer ID signing plus notarisation at US$99/yr with a Mac in the build loop.

### Linux

**`systemd --user`, no root**, with `Restart=always`. **Linger is deliberately not used** (§4, rule 4). Distributed as a tarball or AppImage.

### Uninstall

Removing footprint must remove the daemon's auto-start registration and the scheduled task as well as the binary. An orphaned Task Scheduler entry or LaunchAgent after uninstall is a bug, and it is tested (T-7).

---

## Testing decisions

### The three seams

**Three seams: two architectural, one test-only.** S1 and S2 sit at boundaries the product already has, and the prior art at the end of this section confirms them. It also forced the third — see "Why the store seam is test-only" below.

**S1 — the daemon process boundary, entered two ways.** The CLI for write and operational paths (capture, unlock, import, exclude, prune, export, verify) and **MCP stdio for read paths**. Tests drive the real daemon process. Evidence verification rides this seam by exposing a `verify` CLI verb, so the verifier needs no seam of its own.

**S2 — the connector adapter interface**, with **fake or recorded providers** standing in for real tenant APIs. This seam is unavoidable: the network is outside the product's control, and no test suite can be built on live Teams, Gmail or Slack tenants.

**S3 — a test-only store seam.** A `Store` interface whose only production implementation is SQLCipher-on-disk, and whose test surface is "open the real file with the real key" and "scan the real bytes". **Test-only by design: one production implementation, no second implementation to justify an abstraction.**

**Why the store seam is test-only, and why it is not a data-model seam.** A production store abstraction for *data-model* tests was rejected, and that reasoning still holds: it would couple tests to internals and force the spec to pin module boundaries that should stay free to move. S3 exists for a different reason, and it was the research that forced it. The spec originally claimed "the raw file contains no plaintext" was reachable black-box because it is a property of the *file*. **That claim was wrong.** S1 can *trigger* writes but cannot *observe* the file, and the property is asserted by scanning bytes on disk — the main file, every `-wal` / `-shm` / `-journal` sidecar, any `VACUUM INTO` target, and the OS temp directory — with a deliberately-plaintext control database as the positive control. No seam in the spec could do that. At-rest encryption is the product's stated differentiator (§4), so it is the one claim that must be provable rather than assumed.

### What makes a good test

- **Test external behaviour, not implementation details.** Every test enters through S1 or S2 and asserts on what a user or an agent can observe: command output and exit codes, MCP tool results, the bytes of the store file, and manifest verification outcomes. No test reaches into a module's internals.
- **Assert the invariants of §3 directly.** Each INV row names its test. An invariant with no test is a comment, not a rule.
- **Assert fail-closed behaviour as a first-class case, not an error path.** A locked store is a designed state; it gets its own tests for capture, for each MCP tool, and for the CLI.
- **Assert the negative for confinement.** It is not enough that no write tool is registered — the test asserts the advertised tool set **equals** the six expected names, so an accidentally added tool fails the suite rather than shipping.
- **Do not test the classifier's accuracy.** The assistive classifier is v2 and its accuracy is not a property the product claims (INV-9). What is tested is that a *flagged* record is withheld, which is deterministic.
- **Recorded fixtures are checked in, not generated.** Connector tests must be reproducible without network access.

### What will be tested, and at which seam

| # | Behaviour | Seam |
|---|---|---|
| T-1 | The six advertised MCP tools, and **only** those six; no write, SQL, web or URL tool exists | S1 (MCP) |
| T-2 | Every MCP tool fails closed with a structured error while the store is locked, and the server never prompts for a key | S1 (MCP) |
| T-3 | The raw store file contains no plaintext for any ingested content — **including FTS5 terms, the FTS5 shadow tables and embedding bytes** — with a positive control | S1 triggers the writes; the **assertion** is at S3 |
| T-4 | A wrong passphrase fails closed and the store is not readable | S1 (CLI) |
| T-5 | Per-source participant filtering: non-participant fixtures produce zero records | S2 |
| T-6 | Exclusion is pre-ingest: an excluded record reaches neither the index, the embeddings, nor the manifest | S1 (CLI + verify) |
| T-7 | Install, auto-start registration, single-instance locking, and **clean uninstall** | S1 (CLI) |
| T-8 | Pruning appends exactly one tombstone and the manifest never shrinks | S1 (CLI + verify) |
| T-9 | Manifest verification succeeds **without a key** and fails on a tampered leaf | S1 (CLI) |
| T-10 | Transcript retrieval paginates and never returns a truncated result as complete | S1 (MCP) |
| T-11 | Search returns the current version of a superseded record by default | S1 (CLI + MCP) |
| T-12 | A sensitive-flagged record is withheld from MCP results unless explicitly included | S1 (MCP) |
| T-13 | Incremental capture resumes after an interruption without re-fetching everything | S2 |
| T-14 | Coverage reports a gap for a source not captured inside its provider retention window | S2 |
| T-15 | The shipped agent configuration contains the allowlist, `trust: untrusted`, and sampling/elicitation disabled | static assertion |
| T-16 | No shipped string makes a compliance or evidence claim (INV-9) | static assertion |

### Prior art

Researched after the seam decision. It **confirms S1 and S2 as the architectural seams** and surfaces one property neither can observe — see "The gap this surfaced" at the end of this section.

#### MCP stdio end-to-end (S1)

**An official conformance suite exists — `modelcontextprotocol/conformance`**, run as `npx @modelcontextprotocol/conformance`. It supports footprint's target revision directly (`--spec-version 2025-11-25`), validates every JSON-RPC message on the wire against the spec's JSON schema for the negotiated version (`wire-schema-valid`), and ships per-revision requirement sets plus an `--expected-failures` baseline with stale-entry detection. **But its server mode is HTTP-only** (`conformance server --url ...`), and the repository tree contains **zero stdio paths**. Spec-conformance coverage of a stdio server is therefore not directly available.

**The Python SDK's own suite is the model to copy** — `tests/transports/stdio/`. It spawns **real subprocesses** and monkeypatches `_create_platform_compatible_process` / `_terminate_process_tree` to *record calls while the real implementations still run*; teardown SIGKILLs each spawn-time process group so a crashed test cannot orphan a sleep-forever subprocess. `test_posix.py` asserts a gracefully exiting server's child survives client shutdown, and that a surviving child's write to inherited stdout fails with `EPIPE`. `test_windows.py` asserts the **Windows Job Object** reaps the child — which differs from POSIX — plus selector fallback and CRLF framing. `_liveness.py` proves child liveness through an out-of-band TCP connect-back rather than trusting stdout.

**The documented default is the trap.** Both official SDKs document *in-process* testing (`Client(mcp)`, "no subprocess, no port, nothing on the wire") — a mode that **cannot see the process boundary at all**, and therefore cannot see stdio framing, child cleanup, `EPIPE`, lock contention, fail-closed-when-locked, or daemon-restart persistence. S1's tests must spawn the real CLI and daemon over a real pipe.

**Drivers.** The official `mcp` SDK `Client` over `stdio_client(StdioServerParameters(...))` — it spawns a real subprocess, it is what the SDK tests itself with, and it is the same client the consumer uses. **MCP Inspector CLI** (`mcp-inspector --cli <cmd> --method tools/list --format json`) is a black-box smoke driver with stable exit codes (0 ok, 1 usage, 2 no app, 3 auth, 4 unreachable, 5 tool error) and one JSON line on stderr, so it is branchable in CI. `mcp-pytest` exists but is single-author v0.1.1 — acceptable as a smoke driver, not as the conformance gate.

**Regret this avoids:** in-process-only MCP testing, whose scars are visible in the SDK itself as `test_posix.py` / `test_windows.py`.

#### Encrypted-store leak assertions

**The leak surface is documented** (Zetetic, SQLCipher Design): each page encrypted individually at 4096 bytes, every page write carrying an HMAC-SHA512 checked on read, the whole file appearing as random data. Journals are encrypted — a rollback journal has an unencrypted header but no data — and WAL pages and statement journals are encrypted. **The critical exception: "Other transient files are not encrypted, so you must disable file based temporary storage if your application will use temp space."**

**footprint-specific finding.** `sqlcipher3` 0.6.2's `setup.py` already sets `SQLITE_TEMP_STORE=2`, `SQLITE_HAS_CODEC=1` and `SQLITE_DEFAULT_PAGE_SIZE=4096`, so the wheel is built with the safe temp-store configuration. **Assert it at runtime anyway** (`PRAGMA temp_store` must be 2/MEMORY), because a rebuild or a different binding can regress it. The same build sets `SQLITE_ENABLE_LOAD_EXTENSION=1` — which is what lets sqlite-vec load, and also means the daemon must keep `sqlite3_enable_load_extension` **off** by default.

**How to assert "no plaintext" without a flaky `strings` grep.** A canary corpus — an ASCII canary, a UTF-8 multibyte canary, a long one — inserted as column values, then a byte-level search of the **entire data directory**: main file, `-wal`, `-shm`, `-journal`, any `VACUUM INTO` target, and the OS temp directory the process was given. Run it after every write path: insert, FTS `optimize`, vector insert, prune, export, `VACUUM`, WAL checkpoint, close. **Then run the identical scan against a deliberately-plaintext control database and assert the canary IS found.** Without that positive control, a broken scanner, a wrong path or an empty file all pass vacuously — **that control is the whole trick.**

**Do not gate on entropy or `strings`.** `strings` misses short and multibyte secrets and false-positives on random bytes that happen to be printable; Shannon entropy is a statistical test with false alarms. Smoke checks at most.

**Structural assertions.** The first 16 bytes must not equal `SQLite format 3\0`; `PRAGMA cipher_plaintext_header_size` must be 0, since a recognisable SQLite header is itself a metadata leak. `PRAGMA cipher_integrity_check` must return empty — and the checker must be **self-tested** by corrupting one byte and asserting it names *that* page. SQLCipher's own `test/sqlcipher-core.test` asserts the magic is absent; `test/sqlcipher-integrity.test` shows a middle-page corruption reading as `database disk image is malformed` and a page-1 corruption as `file is not a database`, and that `cipher_use_hmac=OFF` makes the same corruption read as `ok`.

**Enumerate the leak vectors as separate tests**, because each is a real regression risk: a plaintext `ATTACH` target; `VACUUM INTO`; `PRAGMA temp_store=FILE`; a `-wal` left behind after a crash; `sqlite3_backup`; the export verb; crash mid-transaction then restart; `PRAGMA mmap_size`.

#### Per-user daemon lifecycle, without a CI machine per OS

**`uniservice`** is the closest match: it delegates to `systemd --user` / LaunchAgents / Windows Scheduled Tasks, and its suite is deliberately two-layer. **Artifact tests run everywhere** and assert the *generated unit / plist / Task XML as data*; **live lifecycle tests are opt-in and capability-gated** — skipped unless an environment variable is set, and each additionally skips itself when the host lacks a usable manager (`systemctl --user is-system-running`, `launchctl print gui/$uid`, `schtasks.exe`). The Linux live test asserts the full contract: add → appears in `list_info()` → `enabled` → `running` → remove → gone.

**Tailscale** (`tstest/integration`) is the "build once, spawn the real binary per test" model, with an in-process control plane rather than a mocked daemon.

**Single-instance locking.** `portalocker` / `filelock` abstract `fcntl.flock` / `lockf` and `msvcrt.locking` / `LockFileEx`. Test by launching **two real subprocesses** and asserting the second exits with a distinct code — **and that the lock is released after `SIGKILL`**, since a stale lock blocking restart is the classic failure.

**`docker-systemctl-replacement`** executes unit files without systemd, so `systemctl --user enable/start/status` semantics can be exercised in a plain container with no user session bus.

**Regret this avoids:** mocking `systemctl` / `launchctl` / `schtasks`. Tests pass, the unit file is malformed, the daemon never auto-starts. Every project surveyed either tests the generated artifact as data or runs the real manager.

#### Connector and OAuth fixtures (S2)

**Fake authorization servers good enough to drive a real client exist and are mature.** `oauth2-mock-server` (axa-group, npm) covers Authorization Code **with PKCE**, refresh-token, client-credentials, ROPC and JWT-bearer, with discovery and JWKS serving **real signed tokens** ("without mocking the verification layer"), per-test overrides for forcing `invalid_grant` or expiring a token, and it runs **in-process or standalone as a process for non-JS projects — explicitly including Python**. `mock-oauth2-server` (navikt) is the Kotlin/Docker alternative; `node-oidc-provider` is the option if a *spec-conformant* OP is wanted rather than a mock.

**The field has converged on a split, and it is the recommendation.** The **auth plane** is driven against a real fake authorization-server process — hand-mocking OAuth is the regret, because the flow is stateful (authorize → code → token → refresh → revoke, PKCE, state/nonce, clock skew, discovery) and the mock drifts, so the test passes while the refresh path — the one that actually breaks in production — stays dark. The **data plane** uses recorded cassettes (`vcrpy`) plus a few `Prism` / Schemathesis contract checks so cassette rot is detected. **Never record real tokens:** scrub at record time and re-mint from the fake authorization server at replay time.

#### Transparency-log verifier testing

**Third-party test vectors exist.** `transparency-dev/merkle` — the Trillian Merkle library, used by CT, Rekor and sumdb — publishes canonical ground truth (`LeafInputs()`, `NodeHashes()`, `RootHashes()`, `CompactTrees()`, `EmptyRootHash()`), and its `testonly/vectors_test.go` reproduces the **Subtree Test Vectors appendix of `draft-ietf-plants-merkle-tree-certs`**, folding every valid subtree hash for trees up to size 130 into one rolling SHA-256 and comparing against the value published in the draft. That is an external vector set, not a self-consistency check.

**Test the verifier against a third thing, never against the writer.** The library keeps a deliberately naive reference implementation (`refRootHash`, `refInclusionProof`, `refConsistencyProof`) that "directly implement[s] the definitions from RFC 6962", and its own comment says it exists "only for testing correctness of other more flexible and performant algorithms". Writer and verifier are each checked against the reference — never against each other — plus **checked-in frozen fixtures** (log + proof + root) so the verifier test never runs the writer, which is the only way to catch a *coordinated* writer-and-verifier bug. Fuzzing runs in both directions: writer against the reference, and writer through to the verifier.

**Tamper cases are first-class tests:** truncate the proof, flip a hash byte, swap sibling order, use a proof for a different index, use a consistency proof for an older size, reuse a leaf hash as an interior node. Each must be rejected.

**This is not theoretical.** CVE-2026-56865 / GO-2026-6179: `golang.org/x/mod/sumdb/tlog`'s `tileHashReader.ReadHashes` did not verify all tiles against their parents, so a malicious GOPROXY could forge up to two sumdb tiles and **bypass the GOSUMDB check**, persisting attacker-controlled module content. CVSS 8.4, CWE-347, fixed in `x/mod` 0.40.0 — in a mature, heavily reviewed library. **This is the strongest single argument for treating the verifier as a separately tested artifact**, and why its tests deliberately do **not** go through S1 even though the CLI verb that invokes it does.

#### The gap this surfaced — resolved: S3

The research **confirms S1 and S2 are the right architectural seams**, and identifies one property **neither seam can observe**: **store encryption-at-rest**. "Nothing leaks outside the cipher boundary" is a property of a *file*, asserted by scanning bytes on disk including the OS temp directory and every journal and WAL sidecar. S1 can *trigger* writes, but it cannot *observe* the file. Since this is a hard product claim (§4 — the host's disk encryption is not assumed), it is the one property in the spec that currently has no way to be tested.

**Resolved: S3 is added, as a test-only seam.** The prior art treats at-rest encryption as its own concern, separate from the database's own suite — SQLCipher's own tests live at that level, as does the canary-plus-positive-control pattern.

**A fourth seam was considered and rejected.** If "local-only by default" were a *tested* rather than asserted property, S1 could observe inference outputs but could not prove the absence of egress. Proving an absence is materially harder than proving a file has no plaintext, and the claim is already enforced by construction (no cloud endpoint is configured, no embedding API is called). It stays an asserted property with a static check, not a seam.

## Out of scope

Beyond §1's product non-goals, the following are outside **this spec's** scope:

- **Implementation.** This document specifies; it does not build. Ticket breakdown is the next step.
- **The two spikes' execution.** They are specified as the gating procedure (§2) and become real tickets with acceptance criteria in the breakdown — not footnotes.
- **Legal advice.** The PDPA memo is research and is referenced, not reproduced. Nothing here is counsel.
- **The ADR set.** Five decisions are ADR material rather than spec prose and are listed in Appendix B.
- **Chat capture in v1.** Deferred to v2 by §2, except as specified above.
- **The v2 web UI's visual design.** Only its four security requirements are fixed here.
- **Update mechanism.** Not yet decided; see Appendix A.
- **Multi-user, team or hosted deployments.** Excluded by INV-10.
- **Third-party connectors for platforms outside the boundary** (§6's explicit exclusions).
- **A sensitive-data classifier.** v2 by decision, not by omission.

## Further notes

- **The ordering of §3 before §4 is deliberate.** Boundary rules are invariants the architecture must satisfy, not properties it happens to have. Putting architecture first would make them look like consequences.
- **The unattended-key problem has no clean answer.** A process that decrypts must hold the key in memory; the best achievable is a wrapped key in the OS credential store, which sets the security floor at the OS login session. §4 accepts that floor explicitly. It is the single most important thing to get right in review.
- **Encryption is in tension with long-term readability.** ArchiveBox's goal of being readable in 50–100 years without the tool is in direct conflict with store-level encryption. This spec chooses encryption and accepts the tension; export is the mitigation (A-12).
- **A bad transcript is worse than no transcript.** Under the PDPA posture, a record that looks authoritative and is not is more dangerous than an absent record. This is why the meetings scope is conditional on the spikes rather than assumed.
- **The product should surface the [company-policy checklist](https://github.com/SoongGuanLeong/footprint/blob/main/docs/policy-checklist.md) rather than pretend to enforce it.** Sensitive-data controls are one of the items it covers, and the go/no-go gate is a user decision.
- **`docs/agents/issue-tracker.md` was updated rather than quietly violated** when the spec moved from issues to a tracked file (#6, Q16). The same discipline applies here: this spec records where it departs from a prior decision — the one departure is the User Stories section, which #6's ten-section list does not name and the spec template mandates.

## Appendix A — Open questions with triggers

Each carries the **event that resolves it**, so none can be left dangling indefinitely.

| # | Question | Trigger to resolve |
|---|---|---|
| A-1 | **External pin: OpenTimestamps, RFC 3161, or both.** OTS wins on trust model but needs a local Bitcoin Core node and pends for hours; RFC 3161 returns immediately and handles revocation but substitutes a TSA. Carrying both costs the union of requirements. | **First commit of the manifest module.** No default exists. |
| A-2 | **Local inference runtime and default model** — llama.cpp, Ollama, or an embedded runtime; which quantised model ships as the default for an 8GB card. A first-class product dependency with a real packaging burden: model download size, first-run experience, and whether models are fetched or bundled. | **Before the daemon ships an inference path.** |
| A-3 | **Local embedding model**, and what it does to index size inside the encrypted file. Constrained to local-only by the egress decision, so no cloud embedding API is available. | **Before semantic search is implemented.** |
| A-4 | **macOS speech-to-text divergence** — CTranslate2 has no GPU path on macOS, so faster-whisper would be CPU-only on Apple Silicon; whisper.cpp with Metal/Core ML is the alternative. One STT path or two? | **When macOS release work starts** (v1 is Windows-first, so this is not blocking). |
| A-5 | **Remote-session hardening** — re-unlock on a detected remote session, or suppress the localhost UI listener during one, or accept remote sessions as ordinary. **Recommendation: accept them for v1.** | **When the v2 web UI ships**, or earlier if an employer's DLP or policy turns out to care. |
| A-6 | **Update mechanism and delivery** — how a new version reaches an installed unpackaged app, and whether it is automatic. Interacts with code signing and with the per-user install. | **Before the second release.** |
| A-7 | **Sensitive-data classifier design** — which model, and how its unreliability is communicated. v2 by decision. | **After v1 ships and manual exclusion has real usage.** |
| A-8 | **Biometric status of voice and speaker identification.** The PDPA memo's open question R3: whether speaker identification or voice matching processes biometric data. Until determined otherwise, raw audio is sensitive-flagged by default and withheld from the agent surface. | **If speaker identification is implemented.** Otherwise it stays a documented default. |
| A-9 | **Zoom non-host recording ownership.** Whether a user-level OAuth grant can reach a cloud recording owned by another user's account is unverified. If it cannot, Zoom coverage shrinks to meetings the user personally recorded. | **When the Zoom connector is built.** |
| A-10 | **M365 versus Google Workspace prioritisation.** No installed-base split for Malaysia is established, so prioritisation between the two connector families is a judgement call rather than a sourced conclusion. | **When a second connector family is added.** |
| A-11 | **Per-source capture cadence.** Each source has its own retention window (Google Meet transcript entries 30 days; Teams deleted messages 21 days; deleted users/teams 30 days). The polling interval per source needs to be set against the tightest window it serves. | **At connector implementation**, per source. |
| A-12 | **Export format and portability contract.** What travels with the user, and in what format, given that the store is encrypted and the tool may not exist in 20 years. | **Before v1 ships**, since it is a user-facing promise. |

## Appendix B — Referenced decisions, ADRs and research

### ADRs to be created by the ticket breakdown

Per #6, these are **ADR material rather than spec prose**. They are decisions this spec depends on but does not itself argue:

1. **Application-level encryption of the store *and* the index** — including why the host's disk encryption is not assumed.
2. **The AGPL process-boundary rule** — AGPL tools are invoked as separate unmodified processes, never linked, bundled or hosted.
3. **The one-way store → manifest dependency** — the manifest is derived from the store and never the reverse.
4. **The external pin choice** — depends on A-1.
5. **The unattended-key floor** — the key lives in a session; the OS credential store is the floor, and no stronger guarantee is claimed.

### Decision records

- [#5 — Architecture and storage decision](https://github.com/SoongGuanLeong/footprint/issues/5#issuecomment-5829911933) (Q1–Q14)
- [#6 — Spec scope and handoff shape](https://github.com/SoongGuanLeong/footprint/issues/6) (Q16–Q22)
- [#7 — Company policy checklist](https://github.com/SoongGuanLeong/footprint/blob/main/docs/policy-checklist.md)
- [Map #1](https://github.com/SoongGuanLeong/footprint/issues/1)

### Research memos

Referenced, not reproduced — only the actionable rules are extracted above. The PDPA memo is explicitly **not legal advice**, and #4's audit is referenced through `hermes-mcp-integration.md` because the project inverted Hermes into an external consumer, which makes the audit's original framing answer a question this project no longer asks.

| Memo | Used for |
|---|---|
| [oss-reuse-survey](https://github.com/SoongGuanLeong/footprint/blob/main/docs/research/oss-reuse-survey.md) + [sub-surveys](https://github.com/SoongGuanLeong/footprint/tree/main/docs/research/oss) | §4 storage, §8 evidence, licence discipline |
| [platform-support](https://github.com/SoongGuanLeong/footprint/blob/main/docs/research/platform-support.md) | §2 platform order, §10 distribution and signing |
| [form-factor](https://github.com/SoongGuanLeong/footprint/blob/main/docs/research/form-factor.md) | §4 process topology, §9 v2 UI, §10 auto-start |
| [hermes-mcp-integration](https://github.com/SoongGuanLeong/footprint/blob/main/docs/research/hermes-mcp-integration.md) | §7 surface specification |
| [malaysia-workplace-stack](https://github.com/SoongGuanLeong/footprint/blob/main/docs/research/malaysia-workplace-stack.md) | §6 capture paths and retention ceilings |
| [pdpa-2010-boundary](https://github.com/SoongGuanLeong/footprint/blob/main/docs/research/pdpa-2010-boundary.md) | §3 invariants (B1–B10) |
| [hermes-agent-audit](https://github.com/SoongGuanLeong/footprint/blob/main/docs/research/hermes-agent-audit.md) | §4 (why the agent runtime cannot be the store) |
| [policy-checklist](https://github.com/SoongGuanLeong/footprint/blob/main/docs/policy-checklist.md) | INV-11 |
