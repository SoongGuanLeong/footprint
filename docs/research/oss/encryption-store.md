# Application-level encryption at rest and embedded local store options for footprint

- **Scope:** external survey. How can footprint ship its *own* encryption at rest, given the host cannot be assumed to have full-disk encryption (the reference host is Ubuntu 26.04 with a plain ext4 root, no LUKS), on Linux + macOS (Windows later), ideally without sudo?
- **Date:** 2026-09-25
- **Method:** primary sources only — the project's own repository, documentation, and LICENSE/COPYING files, fetched directly. Licenses below were read from the actual license file unless explicitly marked otherwise. Where something could not be verified, it is called out in section 6.
- **Verdict vocabulary:** ADOPT (use as-is), ADOPT-AS-PATTERN (copy the design, do not depend on the code), REJECT (with reason), WATCH (promising but not ready).

---

## 1. The question and the short answer

**Question.** footprint is a single-user, participant-only personal work archive (chats, meeting recordings/transcripts, email) that must be encrypted at the *application* level because the host's disk cannot be assumed encrypted. It must stay searchable (semantic + timeline) and queryable by an external agent (Hermes) over a read-only surface, and it must be installable by other people on their own Linux/macOS machines without sudo. What existing free/OSS solution should it build on?

**Short answer.**

1. **Use an embedded, whole-file-encrypted database as the single store** — the messages, transcripts, the full-text index, and the embeddings all live inside one cipher boundary. The mature choice is **SQLCipher** (BSD-3-Clause, Zetetic) with **sqlite-vec** (Apache-2.0/MIT) for vectors and SQLite FTS5 for keyword search. This is the only family that keeps the *index and the embeddings* inside the encryption boundary while remaining directly searchable, cross-platform, and sudo-free. A credible MIT alternative implementation is **SQLite3MultipleCiphers**.
2. **Do not use FUSE vaults as the primary mechanism.** gocryptfs (MIT), Cryptomator (GPL-3.0), and CryFS (LGPL-3.0) are good products, but they require a mount lifecycle, macOS requires macFUSE/FUSE-T (historically a kernel extension installed with admin rights), and the search index would still have to live *inside* the vault — at which point you have paid the FUSE tax for no additional protection of the index. Keep gocryptfs as an optional "paranoid mode" for the raw attachment blob directory only.
3. **Do not use LUKS or VeraCrypt** (both need root/kernel-level mount) and **do not use encfs** (dormant, unauthenticated legacy crypto, a documented 2014 audit, and fresh 2026 CVEs; its own author now recommends gocryptfs).
4. **Steal the key hierarchy from Standard Notes and the key storage from Signal Desktop.** Standard Notes splits an Argon2-derived root key into a local *master key* and a server password, then uses the master key to wrap per-purpose *item keys*. Signal Desktop encrypts its local message DB with SQLCipher and keeps the DB key in the OS credential store (Electron safeStorage -> Keychain / libsecret / DPAPI). That combination is what makes a **scheduled background capture process** possible without a human typing a passphrase at 3 a.m.
5. **Be honest about the limit.** A process that must read plaintext must hold the key in memory *at that moment*. You can eliminate persistent plaintext key storage on disk and minimise long-lived key residency; you cannot eliminate key-in-memory. The unattended design's security floor is therefore the OS login session (see section 3.5, question 3).
6. **No existing OSS project does the whole job.** The closest is **msgvault** (MIT, by Wes McKinney) — a local-first archive of email, chat, meetings and contacts with keyword + semantic search, attachment extraction, and an MCP server. Its own SECURITY.md states: "The SQLite database is not encrypted. Anyone with filesystem access to ~/.msgvault/ can read all archived emails... Mitigation: Rely on OS-level full-disk encryption (FileVault, BitLocker, LUKS)." That is exactly the assumption footprint cannot make. The gap is real and specific (see section 4).

---

## 2. Summary table

| Candidate | License (verified) | What it covers | What it does NOT cover | Last release / activity | Verdict |
|---|---|---|---|---|---|
| **SQLCipher** | BSD-3-Clause (Zetetic LLC) | Whole SQLite DB file, page-level AES-256-CBC + HMAC-SHA512; covers FTS5 tables and sqlite-vec vectors stored in the same DB; journal/WAL written through the same codec | Separate files (audio, PDFs); key management; plaintext in process memory while open | Active (copyright 2025; 7.3k stars) | **ADOPT** |
| **sqlite-vec** | Apache-2.0 (dual MIT) | Vector search *inside* the SQLite DB -> inherits SQLCipher's cipher boundary | Brute-force (no ANN); performance at very large scale | Active | **ADOPT** (component) |
| **SQLite3MultipleCiphers** | MIT | Alternative SQLite encryption layer; SQLCipher-compatible format + ChaCha20; designed as a drop-in amalgamation | Same limits as SQLCipher; different crypto implementation to audit | Active (copyright 2026) | **ADOPT** (fallback) |
| **SQLite SEE** | Proprietary (paid, ~US$2000) | Official SQLite encryption extension | Not OSS | — | **REJECT** (not OSS) |
| **DuckDB (v1.4+) built-in encryption** | MIT (DuckDB Foundation) | AES-GCM-256 / AES-CTR-256, DB file + **WAL + temporary files**; HNSW vector index (VSS) persisted into the encrypted DB file | Self-declared *not yet NIST-compliant* (issue #20162); VSS is experimental, no incremental index updates | v1.4.2 (Nov 2025), active | **WATCH** |
| **libSQL (Turso)** | MIT | SQLite fork with encryption-at-rest feature | Encryption feature not verified from primary source in this pass; Turso cloud is the commercial focus | Active | **WATCH** |
| **gocryptfs** | MIT | Whole directory tree via FUSE; AES-256-GCM, scrypt KDF; hides filenames | Needs FUSE; macOS support is "available... occasional problems" (needs macFUSE/FUSE-T); index inside vault is still just inside the mount | Active; audited 2017 (defuse.ca) | **ADOPT-AS-PATTERN** / optional blob vault |
| **Cryptomator** | GPL-3.0 (dual-licensed, commercial for ISVs) | Documented vault format 8: scrypt -> KEK -> AES Key Wrap (RFC 3394) of encryption+MAC master keys; AES-SIV-GCM block mode; filename encryption | GPL-3.0 is a distribution constraint if you link/copy code; Java runtime is heavy; needs FUSE/WebDAV/WinFsp | Active (commits Sep 2026) | **ADOPT-AS-PATTERN**; **REJECT as dependency** (GPL) |
| **CryFS** | LGPL-3.0 | FUSE vault; hides file sizes, directory structure, metadata | Heavier (C++/Qt); FUSE dependency; same index-inside-vault issue | Active | **WATCH** |
| **encfs** | LGPL | Legacy FUSE vault | Unauthenticated legacy crypto; 2014 audit ("probably insecure"); 4 CVEs patched in Debian 2026; project dormant, C++ removed, Rust rewrite is beta; author recommends gocryptfs | Rust port beta (Jul 2026) | **REJECT** |
| **rclone crypt** | MIT | File-level encryption of any directory/remote; NaCl secretbox; no FUSE needed for the crypt layer | Not a database; no queryable index; Go-centric API | Active | **WATCH** / ADOPT-AS-PATTERN |
| **age** | BSD-3-Clause | File/stream encryption (X25519, ChaCha20-Poly1305), simple explicit keys | No store, no index, no key rotation at scale | Active | **ADOPT** (for export/backup and attachment blobs) |
| **sops** | MPL-2.0 | Encrypting *values in config files* (YAML/JSON/env), key via age/KMS/PGP | Not a data store | Active | **REJECT** (wrong tool) |
| **VeraCrypt** | Apache-2.0 + TrueCrypt License 3.0 (dual) | Container/partition encryption; FUSE mount on Linux | Mount requires root/privileged FUSE; not embeddable; container format not queryable | Active | **REJECT** (needs root) |
| **LUKS / cryptsetup** | GPL-2.0 (cryptsetup) | Block-device encryption | Requires root; cannot be assumed on the target host | Active | **REJECT** (needs root) |
| **Signal Desktop (design)** | AGPL-3.0 | SQLCipher-encrypted local message DB; DB key held in OS Safe Storage | — | Active | **ADOPT-AS-PATTERN** |
| **Joplin (design)** | AGPL-3.0-or-later | Master password -> 256-byte master key; per-item E2EE (AES-256-GCM); master password never synced | — | Active (docs 2026) | **ADOPT-AS-PATTERN** |
| **Standard Notes (design)** | AGPL-3.0 | Argon2 KDF -> root key split (local master key + server password); master key wraps per-purpose item keys; persistence stores encrypted; master key in OS keychain; optional app passcode wraps it | — | Active (encryption version 004) | **ADOPT-AS-PATTERN** (best key hierarchy) |
| **KeePassXC (design)** | GPL-2.0 or GPL-3.0 | KDBX4: Argon2 KDF, AES-256/ChaCha20 cipher, HMAC-SHA256 integrity; whole-file DB | Single-writer whole-file model; not a searchable archive | Active | **ADOPT-AS-PATTERN** |
| **Anytype** | **Any Source Available License 1.0** — source-available, **NOT OSI open source** | Encrypted local-first object store | License forbids uses an open-source product needs; not OSI-approved | Active | **REJECT** (license) |
| **Obsidian / Obsidian Sync** | Proprietary (closed source) | E2E sync for paying users | Not OSS at all | Active | **REJECT** (not OSS) |
| **msgvault** | MIT | Closest whole-product fit: local archive of email/chat/meetings/contacts, keyword + semantic + hybrid search, attachment extraction, MCP server, Go + SQLite | **No encryption at rest** (documented); OAuth tokens plaintext JSON; alpha | 0.20.0, alpha (2026) | **ADOPT-AS-PATTERN** (and fill its documented gap) |
| **piler / MailPiler** | GPL-3.0-only (custom preamble in LICENSE; GitHub reports NOASSERTION) | App-level message encryption, digital fingerprinting/verification, legal hold, retention rules, dedup, audit-log search — all present in the OSS tree | Crypto design is poor: **static key + static IV from the config file**, unauthenticated AES-256-CBC, with legacy Blowfish-CBC for older records (see 3.6) | Active (2024 copyright) | **REJECT** as a reference; WATCH as a feature checklist |
| **TriliumNext Trilium** | AGPL-3.0 | Per-note ("protected notes") encryption — selective encryption at item granularity | AGPL; single-user notes, no multi-source capture or semantic search | Active | **ADOPT-AS-PATTERN** (selective/field-level encryption) |
| **Paperless-ngx** | GPL-3.0 | Document archive with OCR + full-text search | **No encryption at all** — README: "should never be run on an untrusted host because information is stored in clear text without encryption" | Active | **REJECT** as a store (evidence this camp is empty) |
| **Open WebUI** | "Open WebUI License" — **source-available, not OSI** (bespoke, "All rights reserved") | Optional SQLite encryption | Non-OSI license; not a store | Active | **REJECT** (license) |

---

## 3. Per-candidate detail

### 3.1 Embedded encrypted databases (the recommended family)

#### SQLCipher — ADOPT

- Repo: https://github.com/sqlcipher/sqlcipher · Docs: https://www.zetetic.net/sqlcipher/
- **License: BSD-3-Clause**, read from https://raw.githubusercontent.com/sqlcipher/sqlcipher/master/LICENSE.txt — "Copyright (c) 2025, ZETETIC LLC ... Redistribution and use in source and binary forms... Neither the name of the ZETETIC LLC nor the names of its contributors may be used to endorse...". This is a permissive 3-clause BSD. There is no open-core trap for the feature we need: AES-256 encryption is in the BSD **Community Edition**. A separate **Commercial Edition** exists for support/FIPS-style needs, but it is not required for footprint.
- **Cryptography** (verified against the README and against the SQLCipher-compatible implementation documented at https://utelle.github.io/SQLite3MultipleCiphers/docs/ciphers/cipher_sqlcipher/):
  - Cipher: **AES-256-CBC**, applied per database page.
  - Integrity: per-page **HMAC** tag — SQLCipher v4 uses **SHA-512** (64-byte tag); v3 used SHA-1; SHA-256 optional. A v4 page therefore reserves 80 bytes (16-byte IV + 64-byte tag).
  - KDF: **PBKDF2** over the passphrase with a random 16-byte salt stored in the first 16 bytes of the file; **256,000 iterations** by default in v4 (64,000 in v3); HMAC key derivation uses 2 iterations.
  - **Raw-key mode:** PRAGMA key = "x'...64 hex chars...'" supplies the 256-bit key **without key derivation**. This matters for the unlock story in section 3.5.
  - The README advertises "on-the-fly encryption, tamper detection, memory sanitization, [and] strong key derivation" — the memory-sanitization setting is directly relevant to the "key in memory" question.
- **Why it covers the index and embeddings:** SQLCipher is a SQLite fork whose codec sits *below* the page cache, so every page — table data, FTS5 inverted-index shadow tables, and any sqlite-vec vector table — is encrypted by the same codec, and the rollback journal / WAL are written through the same codec. You cannot accidentally leave a plaintext index next to an encrypted DB, because there is no separate index file. This is the decisive property for footprint.
- **Python / Node without sudo:**
  - Python: sqlcipher3 (zlib license, https://github.com/coleifer/sqlcipher3) is source-only on PyPI (latest 0.6.2). sqlcipher3-binary ships prebuilt wheels, but **as of 0.6.0 these are Linux x86_64 only** (cp38-cp314, manylinux2014) — **no macOS and no arm64 wheels**. So in practice a Python distribution must vendor/build the SQLCipher amalgamation itself, which is possible without sudo but is real packaging work. This is the single biggest practical cost of choosing SQLCipher for a Python app.
  - Node: better-sqlite3-multiple-ciphers (**MIT**, v13.0.3, https://github.com/m4heshd/better-sqlite3-multiple-ciphers) bundles the SQLite3MultipleCiphers amalgamation and ships prebuilds; it is the practical Node route and is SQLCipher-format-compatible.
- **Verdict: ADOPT** as the primary store. Battle-tested in exactly this role (Signal Desktop, Joplin's encrypted profile), permissive license, whole-store coverage.

#### sqlite-vec — ADOPT (component)

- Repo: https://github.com/asg017/sqlite-vec · **License: Apache-2.0** (dual-licensed; the repo carries both LICENSE-APACHE and LICENSE-MIT; GitHub classifies it Apache-2.0). Verified from the repo file listing.
- A pure-C, zero-dependency SQLite extension for vector search. Because it is an extension *inside* the SQLite process, its vector data lives in ordinary SQLite tables and is therefore encrypted by SQLCipher. This is how footprint gets *encrypted semantic search* without inventing anything.
- Caveat: it is brute-force KNN, not ANN. For a personal archive (tens of thousands to low millions of chunks) this is acceptable; it would not be for a web-scale corpus.

#### SQLite3MultipleCiphers — ADOPT (fallback)

- Repo: https://github.com/utelle/SQLite3MultipleCiphers · **License: MIT** (read from LICENSE).
- An independent, MIT-licensed implementation of SQLite encryption that supports a SQLCipher-compatible format *and* additional ciphers (e.g. ChaCha20). Useful if the SQLCipher build/packaging path proves painful, or if you prefer an implementation whose whole stack is MIT. Note that it is a *different crypto implementation* from SQLCipher, so its audit history is not SQLCipher's.

#### SQLite SEE — REJECT (not OSS)

- The official SQLite encryption extension is a paid, proprietary add-on (~US$2000 per the DuckDB blog). Not usable in an OSS product.

#### DuckDB built-in encryption — WATCH

- Blog: https://duckdb.org/2025/11/19/encryption-in-duckdb · **License: MIT** (DuckDB Foundation, read from LICENSE).
- Starting with **v1.4.0**, DuckDB supports transparent data-at-rest encryption: **AES-GCM-256** (authenticated) or **AES-CTR-256** (faster, unauthenticated). DuckDB derives a 32-byte key from the user-supplied key via a KDF, checks it against an encrypted canary in the header, and caches the derived key.
- **Crucially for footprint's index concern:** the blog states the **WAL is encrypted by default** when a key is given, and that **temporary files are encrypted** too. That closes two classic plaintext-leak holes that a naive implementation would miss.
- **Why WATCH and not ADOPT:** DuckDB's own documentation says "DuckDB's encryption does not yet meet the official NIST requirements" and points to issue **#20162** ("Store and verify tag for canary encryption") for the fix. For a product whose entire pitch is encryption, shipping a cipher that self-declares non-compliance is hard to justify today. Additionally, the vector index extension (VSS) is **experimental**, persists the whole HNSW index into the DB on each checkpoint, and has **no incremental updates**.
- **Re-evaluate** when #20162 closes and VSS stabilises. DuckDB is otherwise a superb fit for the *timeline/analytical* half of footprint's search.

#### libSQL (Turso) — WATCH

- Repo: https://github.com/tursodatabase/libsql · **License: MIT** (read from LICENSE.md).
- A SQLite fork with an encryption-at-rest feature. I did **not** verify the crypto design or its maturity from a primary design document in this pass; treat as unverified (see section 6). The project's commercial centre of gravity is the Turso cloud, so watch for open-core drift.

### 3.2 FUSE and container vaults

#### gocryptfs — ADOPT-AS-PATTERN (and optional blob vault)

- Repo: https://github.com/rfjakob/gocryptfs · **License: MIT** (read from LICENSE).
- FUSE-based encrypted overlay; explicitly "inspired by EncFS and strives to fix its security issues". Security design document at https://nuetzlich.net/gocryptfs/security/; **audited March 2017** (https://defuse.ca/audits/gocryptfs.htm).
- macOS support exists but the README is candid: "most things work fine but you may hit an occasional problem" (ticket #15). It needs FUSE — on macOS that means **macFUSE** (historically a kernel extension requiring admin install and kext approval) or **FUSE-T**.
- **Why not primary:** mounting is a lifecycle (mount at login, unmount on lock, handle crashes), and the search index would live inside the mount anyway, so the FUSE vault adds operational complexity without extending protection to the index. **Keep it as an optional "paranoid mode"** that wraps the raw attachment blob directory for users who want defence in depth.

#### Cryptomator — ADOPT-AS-PATTERN for the format; REJECT as a dependency

- Repo: https://github.com/cryptomator/cryptomator · **License: GPL-3.0** (read from LICENSE.txt; the project is dual-licensed GPLv3 / commercial for ISVs).
- **Vault format 8** is well documented (https://cryptomator.org/tags/vault-format/, https://docs.cryptomator.org/security/architecture/): the masterkey file is protected by **scrypt** (non-parallel; documented parameters cost 32768, block size 8) producing a KEK; the KEK wraps the encryption and MAC master keys with **AES Key Wrap (RFC 3394)**; block encryption uses a **SIV-GCM** mode; filenames are encrypted separately.
- **REJECT as a dependency** because **GPL-3.0 is a distribution constraint** for a product like footprint — if you link Cryptomator code into a distributed binary you inherit copyleft obligations, and if you ever offer footprint as a service the calculus worsens. Also, it is Java (heavy packaging) and needs FUSE/WebDAV/WinFsp.
- **ADOPT-AS-PATTERN:** its *key-wrapping* design (passphrase -> KDF -> KEK -> wrap data keys) is exactly the pattern footprint should copy, and it is documented well enough to reimplement cleanly under a permissive license.

#### CryFS — WATCH

- Repo: https://github.com/cryfs/cryfs · **License: LGPL-3.0** (read from LICENSE.txt).
- Unlike file-by-file vaults, CryFS hides **file sizes, directory structure, and metadata**. LGPL-3.0 is more permissive than GPL for linking, but the FUSE/macOS tax and the index-inside-vault issue are the same as gocryptfs. WATCH.

#### encfs — REJECT

- Repo: https://github.com/vgough/encfs · **License: LGPL**.
- The project's own README is damning: "This project has been mostly dormant for years. I've recently ported EncFS to Rust... The old C++ code has been removed... the new implementation is already functional it is still considered a beta release, and I would always have a separate backup." The author's advice: "If you're considering setting up a new encrypted filesystem, I'd recommend looking into newer alternatives, such as the excellent GoCryptFS."
- The legacy (1.9.x) cryptography is the problem: EncFS uses **unauthenticated** encryption, and Taylor Hornby's 2014 audit (https://defuse.ca/audits/encfs.htm) showed practical chosen-plaintext and watermarking attacks against it. Debian shipped fixes for **four encfs CVEs** in 2026 (https://www.progressiverobot.com/2026/03/15/debian-13-encfs-vulnerability-patch-remediation/).
- **REJECT.** Do not ship it, do not recommend it.

#### rclone crypt — WATCH / ADOPT-AS-PATTERN

- Repo: https://github.com/rclone/rclone · **License: MIT** (read from COPYING).
- rclone's crypt backend encrypts files (and filenames) on top of any storage, including a local directory. **It does not require FUSE for the crypt layer itself** — only rclone mount does. That makes it a genuinely no-root, cross-platform way to keep an encrypted blob directory. Its weakness for footprint is that it is Go, not a library you call from Python/Node, and it gives you no queryable index. WATCH; useful as an operational tool rather than an embedded dependency.

#### VeraCrypt — REJECT

- Repo: https://github.com/veracrypt/VeraCrypt · **License: dual Apache-2.0 + TrueCrypt License 3.0** (read from License.txt). Note the TrueCrypt License 3.0 has unusual restrictions and the license explicitly restricts use of the VeraCrypt/IDRIX names.
- Mounting an encrypted volume requires a privileged operation (kernel driver or privileged FUSE). **REJECT** for a no-sudo, embeddable product.

#### LUKS / cryptsetup — REJECT

- Block-device encryption; formatting and opening require root. The reference host has no LUKS and footprint cannot assume it. **REJECT** for the product (it remains the right thing to recommend to users *in addition*).

### 3.3 File- and config-level crypto primitives

#### age — ADOPT (as a utility, not the store)

- Repo: https://github.com/FiloSottile/age · **License: BSD-3-Clause** (read from LICENSE; 3-clause form with the "Neither the name of the age project..." clause).
- Modern, simple, X25519 + ChaCha20-Poly1305, small explicit keys, no config. Excellent for **export bundles, backups, and encrypting attachment blobs** where you do not need queryability. It is not a store and has no index.

#### sops — REJECT (wrong tool)

- Repo: https://github.com/getsops/sops · **License: MPL-2.0** (read from LICENSE).
- sops encrypts *values inside config files* (YAML/JSON/env) with a key from age/KMS/PGP. It is for secrets management, not for an archive store. Useful only if footprint has a config file with secrets (e.g. the OAuth client secret) — but that is a different problem.

### 3.4 Designs worth copying from comparable products

#### Signal Desktop — ADOPT-AS-PATTERN

- Repo: https://github.com/signalapp/Signal-Desktop · **License: AGPL-3.0** (read from LICENSE).
- Signal Desktop keeps "every message, conversation, contact, and attachment record in one SQLCipher-encrypted" database, and the DB key is protected by the **OS Safe Storage** layer (Electron safeStorage -> macOS Keychain / Linux libsecret / Windows DPAPI), with the encrypted key material in the profile's config.json. Forensic tooling confirms the key pipeline (https://securityronin.github.io/signal-desktop-forensic/). Its dependency @signalapp/sqlcipher **4.1.0** ships prebuilt native binaries for x64 and arm64 (verified in package.json).
- **Copy:** SQLCipher + OS-credential-store-held key. **Do not copy the license** (AGPL) — reimplement the pattern, do not take the code.

#### Joplin — ADOPT-AS-PATTERN

- Repo: https://github.com/laurent22/joplin · **License: AGPL-3.0-or-later** for the default tree (read from LICENSE; some subdirectories, e.g. packages/server, have their own licenses).
- Spec: https://joplinapp.org/help/dev/spec/e2ee/native_encryption/. A user **master password** encrypts a randomly generated **256-byte (2048-bit) master key**; the master key encrypts notes and resources; the master password "is stored locally in the database and is never updated to the Sync Target", so a third-party sync target cannot decrypt anything. Native encryption methods use **AES-256-GCM**.
- **Copy:** the separation of "password used to unlock" from "key used to encrypt data", and the rule that the unlock secret never leaves the device.

#### Standard Notes — ADOPT-AS-PATTERN (best key hierarchy)

- Repo: https://github.com/standardnotes/app · **License: AGPL-3.0** (read from LICENSE).
- Spec: https://docs.standardnotes.com/specification/encryption/ and https://standardnotes.com/help/security (current **encryption version 004**).
- The design, quoted from the spec: a password is stretched with a KDF; "The first half of that key is kept locally as the 'master key' and is never revealed to the server"; the second half becomes the account password. The master key encrypts an arbitrary number of **item keys**, and item keys encrypt the actual data. "Persistence stores are always encrypted with the account master key, and the master key is stored [in the OS] keychain (when available)." There is an optional **application passcode** that wraps the master key with an additional layer, and a *"key wrapper"* mode for environments without a keychain.
- **Copy this shape.** It cleanly separates: user secret -> root key -> per-purpose data keys -> records; it tells you where to keep the master key (OS keychain) and how to add a second factor (app passcode). For footprint, "per-purpose item keys" maps naturally onto *content* key, *attachment/blob* key, and *index/embedding* key.

#### KeePassXC — ADOPT-AS-PATTERN

- Repo: https://github.com/keepassxreboot/keepassxc · **License: GPL-2.0 or GPL-3.0** (read from COPYING).
- KDBX4 uses **Argon2** as the KDF, **AES-256 or ChaCha20** as the cipher, and **HMAC-SHA256** for integrity, over a whole-file encrypted database. It is a good demonstration of a strong, modern, *whole-file* encrypted container with authenticated integrity — the same philosophy as SQLCipher, applied to a different data model. Not a searchable archive, so pattern only.

#### Anytype — REJECT (license)

- Repos: https://github.com/anyproto/anytype-ts and https://github.com/anyproto/anytype-heart.
- **License: "Any Source Available License 1.0" (ASAL-1.0)** — the repos carry an ASAL-1.0 badge and the READMEs say "Open code under the Any Source Available License 1.0". GitHub classifies the license as **"Other"/NOASSERTION**, not an OSI license. This is **source-available, not open source**. For a product that must be freely distributable and possibly embedded, **REJECT as a dependency**; it is not even a safe pattern to copy verbatim.

#### Obsidian / Obsidian Sync — REJECT (not OSS)

- Obsidian's desktop app and Sync service are **proprietary/closed source**; there is no license file to read because there is no source release. Its E2E sync is a paid feature. There is nothing to adopt here; listed only because it was in the brief.

### 3.5 The five key questions, answered directly

**Q1. Can it be used from Python or Node without sudo?**
Yes for the embedded-database family. SQLCipher is a library, not a service; nothing about it requires root. The practical friction is *packaging*, not privilege: sqlcipher3-binary publishes **Linux x86_64 wheels only** (no macOS, no arm64), so a Python distribution must build or vendor the amalgamation. On Node, better-sqlite3-multiple-ciphers (MIT) ships prebuilds and avoids the problem. FUSE vaults are sudo-free on Linux *if* /dev/fuse and the FUSE package are present, but on macOS they pull in macFUSE (a kernel extension historically installed with admin rights) — macFUSE on macOS 26 can use an FSKit backend to run filesystems in user space, and **FUSE-T** (https://github.com/macos-fuse-t/fuse-t) deliberately replaces macFUSE with a user-space NFSv4/SMB/FSKit server "instead of a kernel extension", precisely because "it's getting harder and harder to load kernel extensions" on macOS. Either way it is extra machinery. **Embedded DB wins on both privilege and portability.**

**Q2. Does it protect the search index and embeddings too, or only the main DB?**
This is the question that eliminates most candidates. FUSE vaults and whole-file containers (gocryptfs, Cryptomator, VeraCrypt, LUKS) protect *files*, so if your index is a separate file you must put it inside the vault — fine, but then you must also keep the vault mounted for every query, and any cache the search library writes outside the vault is plaintext. **SQLCipher is the clean answer**: because the codec is below the page cache, the FTS5 index and sqlite-vec vectors are the *same encrypted pages* as the content, with no separate plaintext artifact to forget. DuckDB's encryption is the other clean answer, and it additionally encrypts **WAL and temp files** — but its vector extension is experimental and its cipher is self-declared non-NIST today. **Plaintext sidecar indexes over encrypted content are the classic failure mode, and the embedded-DB approach structurally prevents it.**

**Q3. What is the key-derivation and unlock story for a scheduled background capture process — can it work unattended without holding the key in memory?**
First, the honest part: **no.** A process that decrypts data to read or write it must hold the key in memory for the duration of that operation. What you *can* eliminate is (a) a plaintext key on disk and (b) long-lived key residency. The realistic ladder, weakest-to-strongest:

1. **Passphrase-derived key, never stored** (Argon2id -> key, a la KeePassXC/KDBX4). Strongest at rest, but requires a human to unlock — so capture cannot run while locked. This is the "locked mode".
2. **Data key wrapped by a key held in the OS credential store** (Standard Notes / Signal Desktop pattern). After login, the scheduled capture process can unwrap the data key without prompting. Unattended. **The security floor equals the OS login session** — any process running as the user can obtain the key. This is the pragmatic default.
3. **Short-lived key agent** (ssh-agent style): the user unlocks once per session; a small agent holds the key in locked (mlock'd) memory and hands it to the capture process on demand, never writing it to disk. Unattended within a session; better memory hygiene; more moving parts.
4. **TPM / Secure Enclave binding** — strongest, but not portable across Linux+macOS and not something a no-sudo install can rely on.

Recommendation: implement **(2) as the default** with **(1) as a "locked / paranoid" mode** and **(3) as an opt-in hardening step**. Design details that matter: use SQLCipher's **raw-key mode** (PRAGMA key = "x'...'") so the expensive KDF runs once at unlock and the derived key is what is stored/wrapped; enable SQLCipher's **memory sanitization**; and derive *separate* data keys for content, blobs, and the index so a compromise of one does not open the others.

**Q4. Cross-platform availability and packaging weight.**
- SQLCipher: small C library; BSD; needs a build/vendor step for Python on macOS/arm64; Node prebuilds available. Lowest runtime weight.
- SQLite3MultipleCiphers: MIT, amalgamation-friendly; same profile as SQLCipher.
- DuckDB: MIT, single self-contained binary, first-class pip/npm wheels for all platforms — but heavier (tens of MB) and its crypto is young.
- gocryptfs / CryFS: Go / C++ binaries plus a FUSE dependency; macOS needs macFUSE or FUSE-T.
- Cryptomator: GPL-3.0, Java runtime — the heaviest option, and a copyleft constraint.
- age / rclone: small Go binaries, MIT/BSD, no runtime dependencies.

**Q5. Known CVEs or broken-crypto reputations.**
- **encfs:** the clear offender — unauthenticated legacy crypto, the 2014 Hornby audit, four CVEs patched in Debian in 2026, and the author himself now points users to gocryptfs. **REJECT.**
- **SQLCipher:** no notable recent CVE history surfaced in this pass; its 3-clause BSD license carries the standard warranty disclaimer, and its crypto (AES-256-CBC + HMAC-SHA512 + PBKDF2) is conservative and widely deployed (Signal, Joplin). One honest caveat: SQLCipher v4 changed defaults (page size, iterations) such that v4 cannot open v1-v3 databases without explicit PRAGMAs — a migration detail, not a vulnerability.
- **DuckDB encryption:** not a CVE, but a *self-declared* gap — "does not yet meet the official NIST requirements", tracked in issue #20162. Treat as not-yet-proven.
- **Anytype:** not a crypto issue — a **license** issue (source-available, not open source).
- **VeraCrypt:** dual-licensed with the TrueCrypt License 3.0, whose terms are unusual; and mounting needs privilege.

### 3.6 Archive platforms (cross-checked against the archive-platform survey)

The archive-platform survey (docs/research/oss/archive-platforms.md) concluded that no candidate offers an encrypted multi-source store to reuse. I re-verified the load-bearing claims from primary sources and found one important correction.

**piler / MailPiler — GPL-3.0-only; app-level encryption exists, but do not copy the design.**
- Repo: https://github.com/jsuto/piler. The LICENSE file reads: "piler, an enterprise level email archiving application. Copyright (C) 2012-2024, Janos SUTO ... under the terms of the GNU General Public License as published by the Free Software Foundation, **version 3** of the License" — i.e. **GPL-3.0-only** (not "or later"). GitHub reports NOASSERTION only because of the custom preamble.
- The README does list "message encryption", "digital fingerprinting and verification", "legal hold" and "retention rules" (README lines 12, 15, 16), and the code is in the **OSS tree**, not gated behind the commercial Enterprise edition: the config parser declares `encrypt_messages` and `iv` (src/cfg.c), and the encrypt/decrypt path is in src/archive.c.
- **However, the crypto is weak** — which the survey could not see because the wiki page 404s. From src/archive.c:
  - A comment at line 168 reads "The new encryption scheme uses piler id starting with 5000....", and the code selects between two ciphers: `EVP_aes_256_cbc()` for newer records and **`EVP_bf_cbc()` (Blowfish-CBC)** for older ones (lines 174-197).
  - The key and IV come straight from configuration — `cfg->key` and `cfg->iv` (src/cfg.h defines `unsigned char key[KEYLEN]` and `unsigned char iv[MAXVAL]`; src/cfg.c parses a plain `iv` string).
- Consequences: a **static IV** with CBC mode makes encryption deterministic (identical plaintexts produce identical ciphertexts, and shared prefixes leak), and CBC without a MAC provides **no tamper detection** — which directly conflicts with footprint's append-only / verifiable-artifact requirement. Blowfish is a 64-bit-block cipher that has been deprecated for years.
- **Verdict: REJECT as a reference implementation.** Keep piler as a *feature checklist* (legal hold, retention rules, fingerprinting, audit-log search are all things footprint's evidence model will want), not as a crypto design to copy. This also answers the survey's open question: the features are in the OSS tree, and the algorithm is AES-256-CBC (new) / Blowfish-CBC (legacy) with a static key and IV from config.

**TriliumNext Trilium — AGPL-3.0; ADOPT-AS-PATTERN for selective encryption.**
- Repo: https://github.com/TriliumNext/Trilium; LICENSE verified as **AGPL-3.0**. Its per-note "protected notes" model encrypts individual items rather than the whole store. That granularity is a useful pattern for footprint's PDPA constraint (sensitive personal data excluded, purpose-limited): it shows how to encrypt only designated records rather than forcing an all-or-nothing store.

**Paperless-ngx — GPL-3.0; explicitly unencrypted.**
- Repo: https://github.com/paperless-ngx/paperless-ngx; LICENSE verified as **GPL-3.0**. Its README states, verbatim (line 102): "Paperless-ngx should never be run on an untrusted host because information is stored in clear text without encryption." A useful, citable admission that this whole camp assumes full-disk encryption.

**Open WebUI — source-available, not OSI.**
- Repo: https://github.com/open-webui/open-webui. The LICENSE file is a bespoke "Open WebUI License ... All rights reserved" with conditions — **not an OSI-approved license** — so REJECT regardless of its optional SQLite encryption. Same category as Anytype.

**Others with no documented at-rest encryption:** Khoj, Karakeep, ArchiveBox, Onyx, AnythingLLM, Memos, Docspell (per the archive-platform survey). SurfSense stores only API keys in the OS keychain, which is credential handling, not store-level encryption.

### 3.7 Design precedents for a verifiable manifest (cross-checked)

Section 6, item 9 asks how to reconcile append-only, independently verifiable artifacts with an encrypted store. Cross-checked with the archive-platform survey; every citation below is verified from a primary source.

**Anti-pattern to name in the ADR: restic (BSD-2-Clause).** Verified from https://github.com/restic/restic/blob/master/LICENSE and doc/design.rst:
- "Apart from the files stored within the `keys` directory, all files" are encrypted with **AES-256-CTR** and integrity-protected by a **Poly1305-AES MAC** ("IV || CIPHERTEXT || MAC", random IV per file, 32-byte overhead) — design.rst lines 50-61.
- The **config file is encrypted** (line 64) and "Individual files for the index, locks or snapshots are encrypted" (line 228).
- The **storage ID is "the SHA-256 hash of the content stored in"** the file — that is, of the *encrypted* bytes (lines 22, 33, 42-43).
- Consequence: the integrity protection is genuine, but **verifying it requires the key**, and the identifier is meaningless to anyone who cannot decrypt. That is exactly the property footprint must avoid, because it makes the evidence unverifiable without the very secret the evidence is supposed to outlive. Name restic as the anti-pattern, not as a model.

**Structural precedent, verified, but unencrypted: WARC + CDX/CDXJ.** The WARC 1.1 specification (https://iipc.github.io/warc-specifications/specifications/warc-format/warc-1.1/) defines two optional per-record digest headers:
- **WARC-Block-Digest** — "an optional parameter indicating the algorithm name and calculated value of a digest applied to the full block of the record", formatted as `algorithm:digest-value` (example `sha1:AB2CD3EF4GH5IJ6KL7MN8OPQ`), and the spec states "No particular algorithm is recommended" — i.e. algorithm-agile.
- **WARC-Payload-Digest** — the digest of the payload, "not necessarily equivalent to the record block".
A separate index (CDX/CDXJ) is conventionally maintained alongside. This **is** the container-plus-independent-manifest split footprint wants, and it is a standards-based precedent rather than an invention — but it is **plaintext**. So: adopt the *shape* (self-describing records carrying algorithm-agile digests, plus a separate index), and add the encryption nobody in this lineage has — but do **not** adopt WARC's **plaintext manifest** by default, because an unencrypted digest manifest is itself a disclosure surface (section 3.8).

**Property to steal from Maildir + notmuch/mu:** the index is *derivable from the store alone* and disposable — it can be deleted and rebuilt without loss. footprint's manifest should have the same property: reconstructible and checkable from the store, never a second source of truth that can drift out of sync.

**Greenfield: external timestamping.** No candidate in either survey integrates an RFC 3161 TSA or OpenTimestamps in any form. That part of footprint's evidence design is new work with no precedent to borrow — and, per section 3.9, it is the **load-bearing** part of the design rather than a decoration: without an external pin, the strongest claim the archive can make is one it signs about itself.

**One-directional dependency (caution from ArchiveBox's MCP server).** ArchiveBox derives its agent tools by dynamic Click introspection over its CLI, and consequently leaks write commands (add/crawl/snapshot/shell) alongside the read tools — the general lesson being that a derived interface inherits the blast radius of what it is derived from. Applied to the manifest: the dependency must run strictly **store -> manifest, never manifest -> store**. A manifest writer that can rewrite records means the store is not append-only, whatever the schema claims.

### 3.8 The manifest is itself a disclosure surface (and restic is not simply wrong)

Section 3.7 framed the goal as "a manifest verifiable without the key". oss-archive-platforms identified the trap in that framing, and it is a real one: **an unencrypted per-record manifest is a metadata leak, and an unkeyed digest is a confirmation oracle.** The ADR must state this trade explicitly rather than treat plaintext verifiability as strictly better — and restic's choice should be recorded as *correct for backups*, not as a mistake.

Why the oracle matters *here* specifically: a plaintext manifest of record digests discloses how many records exist, their sizes and their timestamps; and it lets anyone who can *guess* a record's content hash the guess and test for membership. That is a named, published attack class rather than a theoretical worry. Halevi, Harnik, Pinkas and Shulman-Peleg, "Proofs of Ownership in Remote Storage Systems" (CCS 2011; https://eprint.iacr.org/2011/207) describe exactly it: "an attacker who knows the hash signature of a file can convince the storage service that it owns that file", enabling "access to potentially huge files of other users based on a very small amount of side information". For a participant-only personal archive of emails and meeting transcripts under PDPA, letting a future owner of the machine — or a curious adversary — *confirm that a particular email exists* without the key is a meaningful disclosure, and it is precisely the property footprint's participant-only framing exists to prevent.

The design space, then, is not "manifest vs no manifest":

1. **Plaintext manifest, unkeyed digests.** Verifiable by anyone, no key needed. Leaks counts, sizes and timestamps, and provides a plaintext confirmation oracle. This is where WARC + CDX/CDXJ sits, and where ArchiveBox sits.
2. **Manifest with keyed digests** (an HMAC under a *separate* manifest key, not the data key). Kills the plaintext oracle, but verification now needs the manifest key — which is only an improvement if that key is stored or escrowed **separately from the data key**, so that "verify authenticity" and "read the content" become two distinct capabilities. That separation is the actual design decision, and it is the one the ADR should make explicitly rather than inherit from whichever precedent gets copied.
3. **Plaintext manifest over ciphertext** (digest the encrypted record rather than the plaintext). Verifiable with no key at all, since you are hashing what is already on disk, and it leaks no plaintext oracle. But the digest then proves only that the *ciphertext* is unchanged: it detects tampering and corruption, not substitution-with-reencryption, and it says nothing about whether the record decrypts to what it should.

Prior art for the third path, and for the append-only property generally, is mature: **transparency logs**. RFC 6962 (Certificate Transparency; https://www.rfc-editor.org/rfc/rfc6962.txt) specifies "publicly auditable, append-only, untrusted" logs built from a Merkle Tree (section 3.4) with a Signed Tree Head (section 3.5), and Google's Trillian (https://google.github.io/trillian/docs/TransparentLogging.html) generalises this to "transparent, append-only logging of arbitrary data" using "Merkle trees, inclusion/consistency proofs, signed tree heads". The decisive design freedom is stated explicitly in Trillian: the *application* chooses what goes into a leaf — "the default Merkle hash for a Trillian Log leaf is SHA-256(0x00 | leaf.LeafValue)". So a signed Merkle tree whose leaves are hashes of **ciphertext** gives public verifiability of integrity and append-only-ness **without** a plaintext oracle, while a separate, key-wrapped plaintext-level authentication value can supply content authenticity.

**Proposal for the ADR (a proposal, not a verified finding):** combine (3) and (2) — a signed Merkle tree over ciphertext for public, key-free integrity and append-only proofs, plus an optional keyed HMAC over the plaintext, stored encrypted under a manifest key escrowed separately from the data key. That yields three separable capabilities: *verify the archive is intact and append-only* (no key), *verify a record decrypts to the expected plaintext* (manifest key), and *read the content* (data key). No project in either survey implements this; the two surveys found only option (1), or the restic anti-pattern. It is a position to be argued in the ADR, not a precedent to be cited. Note also that the signed tree head this proposal rests on must be **externally pinned** before the append-only claim means anything at all — see section 3.9.

**And the honest note on restic:** encrypting the index and keying the identifier is the right call *for a backup tool*, where a stolen repository should leak neither filenames nor sizes nor a membership oracle. The difference is not that restic is wrong; it is that footprint's evidence must stay verifiable by someone who does not hold the key — a requirement backups simply do not have. State that trade in the ADR.

### 3.9 What a transparency-log structure does and does not buy (the pin is load-bearing)

Section 3.8 proposed a signed Merkle tree over ciphertext. oss-archive-platforms raised the caveat that decides whether that proposal is honest, and it is correct: **a signed tree head is only as trustworthy as the independence of its signer.**

In Certificate Transparency the append-only guarantee does not come from the Merkle tree alone. RFC 6962 does state that "the append-only property of each log is technically achieved using Merkle Trees", but the same document makes clear the guarantee is enforced socially as well as cryptographically:
- Section 5.3 (Monitor): "Monitors watch logs and check that they behave correctly... A monitor needs to, at least, inspect every new entry in each log it..."
- Section 7.3 (Misbehaving Logs): "A log can misbehave in two ways..." — with detection resting on "gossiping, i.e., everyone auditing logs comparing their versions of..."
- Line 1171: "All clients should gossip with each other, exchanging STHs at least" — though the same paragraph defers the mechanism ("The exact mechanism for gossip will be described in..."), so in practice the guarantee is only as real as the deployed monitor and gossip ecosystem.

A single-user local archive has **no monitors and no gossip**. The same key that writes the archive can rewrite history and sign a fresh tree head, producing a clean split view that nobody is positioned to detect. So the transparency-log structure buys **efficient proofs relative to a tree head** — it does not, by itself, buy append-only-ness. Something external must **pin** the head (an RFC 3161 TSA, OpenTimestamps, or publication somewhere the owner does not control) before "append-only" is a guarantee rather than a claim.

This reframes the external-timestamping item in section 3.7: it is not a nice-to-have bolted onto the manifest, it is **the load-bearing half**. Without an external pin, the strongest statement footprint can make is "this tree head says the archive looked like this", signed by the party whose evidence is in question. That is the wrong property for the case footprint is built for: in a workplace dispute the archive's owner is plausibly the party who most needs the evidence to be credible to someone else, so a self-signed append-only archive fails precisely where it is needed most.

**Second design detail worth taking from Trillian: identity hash vs Merkle hash.** Trillian distinguishes two per-leaf hashes. The *Merkle hash* "percolates up the Merkle tree and is therefore incorporated into the root hash"; a separate per-leaf *identity hash* "identifies which leaf values should be considered equivalent, in the sense that an existing leaf with a given (application-provided) identity hash prevents Trillian from accepting any new leaves with the same identity hash". Trillian describes the use case in terms that map directly onto footprint's evidence constraint: "for applications where it is important to record the time of logging a thing, together with the thing itself — where a later attempt to log something that is already logged should be rejected." For CT, the identity hash "is a hash over the logged certificate itself, without the tree leaf structure that includes the log timestamp. This means that a certificate only gets logged once; a second attempt to log the same certificate will have a duplicate identity hash even though the Merkle hash is different (because it has a different timestamp)."

Two consequences for footprint:

1. **Separate the identity notion from the tree hash.** If the leaf embeds a timestamp, the Merkle hash of a re-submitted record differs, so dedup cannot rely on it. An identity hash over the record content (or its ciphertext) lets a re-submission be **rejected** rather than silently creating a second timeline entry — which is what both the append-only constraint and piler's dedup feature actually require.
2. **A timestamp inside the leaf is the archive's own assertion**, exactly as the CT leaf timestamp is the log's own assertion. It is not an external attestation, and it carries the same trust gap as the tree head, one level down. If footprint embeds a per-record timestamp, that timestamp needs the same external anchoring as the head, or it is self-asserted provenance dressed as evidence.

**Which pin, and why it is not a coin flip.** The two obvious choices have different trust models, both verified from primary sources:
- **RFC 3161** (https://www.rfc-editor.org/rfc/rfc3161.txt) is explicit that "a TSA may be operated as a Trusted Third Party (TTP) service" — so it substitutes one trusted party for another. For an archive whose premise is not having to trust a vendor, that is a partial retreat from the design's own principle, though it can still be the right call if the TSA is a jurisdictionally independent party.
- **OpenTimestamps** (https://opentimestamps.org/) "defines a set of operations for creating provable timestamps and later independently verifying them", anchored in the Bitcoin blockchain: "Anyone could realize a timestamp with the permissionless blockchain by paying the transaction fees, for your convenience we offer calendar servers that perform this operation for you. These servers are free to use and they don't require any registration or api key." The calendar servers are an availability dependency, not a trust dependency, because the proof is checkable against Bitcoin.
I have **not** verified OpenTimestamps' latency/pending behaviour, so this remains a lead rather than a settled choice. On the trust model alone OpenTimestamps looks better aligned — but that is a value judgement, not the whole picture, because the two pins have *different* long-horizon risk profiles (open question 10). **Correction to my earlier framing in this memo:** RFC 3161 handles the expiry/revocation case at the protocol level. It exists to "prove that a digital signature was generated during the validity" period and to allow "verifying signatures created prior to the time of revocation" (lines 51, 73-77), and its verification procedure is bounded by "the validity period of the signer's certificate" with the revocation required to be "later than the date/time indicated by" the timestamp (lines 1137-1156). OpenTimestamps has no analogous protocol-level answer to Bitcoin's long-run security assumptions. **But that provision retires less than it first appears.** Step 5 of the same verification procedure requires: "The revocation information about that certificate, at the date/time of the Time-Stamping operation, MUST be retrieved" (lines 1152-1153). The RFC *mandates retrieving* historical revocation data; it cannot guarantee that a CRL or OCSP response from years earlier is still obtainable or interpretable. RFC 3161's answer to revocation is therefore **protocol-level, but its inputs are infrastructure-level** — the same class of dependency as OpenTimestamps' calendars, one layer down. That does not refute the asymmetry (the RFC genuinely has an answer OTS lacks), but the long-horizon gap is narrower than "solved versus unsolved", and the deciding unknown is whether real TSAs actually ship the long-term-validation material this procedure depends on.

So the pin trade runs on **three axes**, not two: **trust model** (OpenTimestamps needs no trusted party; RFC 3161 substitutes a TTP), **latency** (which inverts in RFC 3161's favour — see below), and **long-horizon revocation handling** (RFC 3161 has a bounded answer; OTS has none). Which matters more is a decision for the ADR, not something to default.

**OpenTimestamps' operational costs are real, and they qualify the recommendation.** Verified from the official client README (github.com/opentimestamps/opentimestamps-client, README.md, checked 2026-09-25):
- **Verification needs a Bitcoin node.** "While OpenTimestamps can *create* timestamps without a local Bitcoin node, to *verify* timestamps you need a local Bitcoin Core node (a pruned node is fine)." This lands directly on the evidence constraint: if the point of append-only is that someone *other* than the archive owner can check it — a colleague, a lawyer, a court, a future owner of the machine — then that verifier must run Bitcoin Core. **"OpenTimestamps" and "verifiable by any third party" are therefore not the same claim**, and for a distributable product whose selling point is verifiable evidence this is an adoption barrier that belongs in the ADR as a documented cost, not a surprise at the point of dispute.
- **Proofs come in two states, and only one survives the calendars.** "Incomplete timestamps are ones that require the assistance of a remote calendar to verify; the calendar provides the path to the Bitcoin block header." The default calendars are donation-funded, so an un-upgraded proof is hostage to volunteer infrastructure that may not exist in a decade. A *complete* timestamp is self-contained, and the client provides `ots upgrade` to convert one to the other.
- **Design rule for the ADR: once a stamp confirms, upgrade it and store the complete proof — never the pending one.** This is controllable by us, and it converts a volunteer-uptime dependency into a one-time operation.

**Why carrying both anchors is better than redundancy — and what it costs.** The two are complementary on *different axes*, which is a stronger rationale than "two shots at the same target":
- **Trust model:** OpenTimestamps needs no trusted party; RFC 3161 substitutes a TTP.
- **Latency, and this one inverts in RFC 3161's favour:** an OTS stamp is pending for "a few hours" until a Bitcoin block confirms, whereas a TSA returns a token immediately. So for the capture-time bound in limit 1, **the TSA gives the tighter bound** — which partially offsets its weaker trust model, and means carrying both genuinely improves the *earliest* attestation that can be shown rather than merely adding redundancy. One honesty caveat: the token's tightness is itself an assertion. RFC 3161's `genTime` is "the time at which the time-stamp token has been created by" the TSA, with an `accuracy` field representing "the time deviation around the UTC time"; so the tighter bound is the TSA's own claim about its own clock, and that claim is exactly what the trust substitution buys.
- **Revocation handling:** RFC 3161, as bounded above.
So the case for the ADR is not "neither single dependency is fatal" but **"each anchor covers the other's weakness on a different axis"**.

**The cost, which cuts the other way.** Carrying both costs the **union** of the verification requirements, not the intersection: the verifier needs a Bitcoin node *and* a TSA chain plus a trust store *and* the historical revocation data. That compounds the adoption barrier rather than relieving it — carrying both maximises the chance that at least one anchor survives while simultaneously maximising the burden on whoever has to check the archive. For a distributable product whose evidence may be checked by a third party, those two goals pull in opposite directions, so the ADR must state **which one it is optimising** rather than assume both are free. Two mechanical rules follow: bind both anchors to the **same manifest root** so they attest one value rather than creating two incompatible evidence chains; and store complete, upgraded OTS proofs, never pending ones.

**Two limits on what a pin can attest, both from the protocols' own wording.**

1. **A pin bounds existence from above; it never attests the moment of capture.** Both protocols define themselves identically: OpenTimestamps says "A timestamp proves that some data existed **prior to** some point in time", and RFC 3161 says a time-stamping service "supports assertions of proof that a datum existed **before** a particular time" (line 43). A pin therefore cannot attest *when* a record was captured — only that it existed before some later instant. For a workplace dispute where the question is literally "when did I learn X", the capture timestamp stays **self-asserted** no matter how strong the Merkle structure is. The consequences are concrete: **stamp at capture time, not at manifest-build time**, and keep the stamping interval short. An archive that stamps only when it builds a manifest proves "this record existed before some later point", which sounds stronger than it is. The latency is quantified in the client's own README: the calendars report "Pending confirmation in Bitcoin blockchain" and "It takes a few hours for the timestamp to get confirmed by the Bitcoin [blockchain]". So stamping at capture gives a bound of capture **plus a few hours** — far tighter than manifest-build time, but not a same-second attestation, and nobody should assume one. The client's own success output states the semantics plainly: "Bitcoin block 358391 attests existence **as of** 2015-05-28 CEST".
2. **The leaf design and the pin choice are coupled, not independent.** If the pin is public and permanent — which Bitcoin is — the pinned value must never be an unkeyed plaintext digest. Publishing a Merkle root over ciphertext leaves, or over keyed HMACs, is safe; publishing an unkeyed plaintext digest to the blockchain would be the most durable **confirmation oracle** conceivable: permanent, public, checkable by anyone who can guess a record's content, and unretractable. So the ADR needs a **hard rule, enforced by construction rather than by review: the structure that gets pinned must be keyed or ciphertext-derived.** This also means the proposal above is not merely additive — option 2's keyed HMACs are what make a public OpenTimestamps pin *safe*, so they are a precondition for that pin rather than an optional extra.

---

## 4. Closest fits and the gap

**Is there one OSS project that already does most of this? No.** The landscape splits cleanly into two camps, and footprint sits in the empty space between them.

**Camp A — encrypted, but not a searchable archive.**
Joplin (AGPL-3.0), Standard Notes (AGPL-3.0), KeePassXC (GPL-2/3), Signal Desktop (AGPL-3.0) all solve encryption-at-rest well. None captures chats/meetings/email, none does semantic search, none exposes a read-only agent surface, and all are copyleft. They are **patterns**, not dependencies.

**Camp B — a searchable archive, but not encrypted.**
- **msgvault** (MIT, https://github.com/kenn-io/msgvault) — the closest single project by far. It is a local-first archive of email, chat, meetings, calendars and contacts; it syncs Gmail/IMAP/Microsoft 365; it does keyword, semantic and hybrid search; it extracts attachment text; and it ships an **MCP server** for agents. It is Go + SQLite and permissively licensed. **Its gap is exactly footprint's reason to exist:** SECURITY.md says "The SQLite database is not encrypted. Anyone with filesystem access to ~/.msgvault/ can read all archived emails", OAuth tokens are plaintext JSON, and the recommended mitigation is "Rely on OS-level full-disk encryption (FileVault, BitLocker, LUKS)" — the one thing footprint cannot assume. It is also **alpha** ("APIs, storage format, and CLI flags may change"), and transcription of meeting *audio* is not its stated focus.
- **Khoj** (AGPL-3.0) — self-hostable personal AI with semantic search over notes/docs. No at-rest encryption; AGPL.
- **Reor** (AGPL-3.0) — local AI note-taking with a vector DB (LanceDB). No at-rest encryption; AGPL.
- **Karakeep** (formerly Hoarder, AGPL-3.0) — self-hosted archive with AI tagging and semantic search. No at-rest encryption; AGPL; web-server-shaped rather than single-user-local.
- **LanceDB** (Apache-2.0) and **Chroma** (Apache-2.0) — vector stores, permissively licensed, but **no encryption at rest**; using either would force you back onto a FUSE vault to protect the index, reintroducing every problem in section 3.5 Q2.
- **Paperless-ngx** (GPL-3.0) — a strong document archive, but it says outright in its README that it "should never be run on an untrusted host because information is stored in clear text without encryption". Nothing to reuse for the store.
- **ArchiveBox** — deliberately stores a plain, browsable directory, on the stated principle that the archive should remain readable in 50-100 years *without the tool*. That durability goal is in genuine tension with store-level encryption, and it is the sharpest design question footprint inherits (see section 6, item 9).

**The specific gaps footprint must close, and with what:**

| Gap | Closed by |
|---|---|
| No at-rest encryption in the archive projects (msgvault, Khoj, Reor, Karakeep) | SQLCipher (BSD) whole-store encryption |
| Index/embeddings leaking as plaintext sidecars | Vectors via **sqlite-vec** *inside* the encrypted DB; FTS5 inside the same DB |
| Unlock story for unattended capture | Standard Notes key hierarchy + OS-credential-store-held wrapped key (Signal Desktop pattern) |
| Attachment blobs (audio, PDF, raw email) are outside the DB | Encrypt blobs with the same master key via an AEAD (or **age**, BSD-3) — or store small blobs as rows in the encrypted DB; optionally offer **gocryptfs** (MIT) as a "paranoid mode" vault for the blob directory |
| Append-only + verifiable timestamps (evidence constraint) | Design concern outside this memo's scope, but note: an HMAC hash-chain over records *inside* the encrypted DB is compatible with the SQLCipher design and should be specified alongside it |
| Read-only agent surface | Not an encryption concern; MCP is a separate memo |

**Recommendation (with the trade-off stated).** Build the store on **SQLCipher (BSD-3-Clause)** — or **SQLite3MultipleCiphers (MIT)** if you want an all-MIT stack — holding content, transcripts, the FTS5 index, and sqlite-vec embeddings in one encrypted file, with attachments encrypted as AEAD blobs under the same key hierarchy. **Trade-off:** you accept a build/vendor step for SQLCipher on macOS/arm64 (the Python wheels are Linux-x86_64 only) and you accept that a single-writer embedded DB is the concurrency model. In exchange you get the only architecture in this survey that protects the *index and embeddings* by construction, needs no root, no mount lifecycle, and no kernel extension, and is proven in production by Signal and Joplin. **Keep DuckDB (MIT) on WATCH** as the eventual analytical/timeline engine — its WAL and temp-file encryption are exactly right — and re-evaluate it once its NIST gap (#20162) closes and its vector index leaves "experimental".

---

## 5. Recommended shape (for the ADR that should follow)

1. **Master key hierarchy:** user passphrase -> Argon2id -> root key; root key split into a local **master key** and a derived *unlock verifier*; master key wraps three purpose keys — **content**, **blobs**, **index/embeddings** (Standard Notes pattern).
2. **Storage:** one SQLCipher database for content + FTS5 + sqlite-vec vectors; attachment blobs encrypted with an AEAD under the blob key (inline if small, sidecar files if large).
3. **Unlock:** wrapped master key in the OS credential store (libsecret on Linux, Keychain on macOS) for unattended capture; an optional "locked mode" that requires the passphrase per session; optional short-lived key agent.
4. **Key hygiene:** SQLCipher raw-key mode so the KDF runs once; memory sanitization enabled; no plaintext key files; ensure transcription temp audio and model caches are either inside the encrypted store or shredded after use.
5. **Evidence pin:** stamp **at capture time**, not at manifest-build time, and keep the interval short — a pin bounds existence from above and never attests the moment of capture. Ensure the **pinned structure is keyed or ciphertext-derived**; an unkeyed plaintext digest pinned to a public chain is a permanent, unretractable confirmation oracle, so keyed HMACs are a precondition for a public pin rather than an optional extra. Once a stamp confirms, **upgrade it and store the complete proof**, never the pending/incomplete one — an incomplete proof depends on volunteer-run calendar servers that may not exist in a decade. If both an OTS proof and an RFC 3161 token are carried, bind them to the **same manifest root** so they attest one value rather than two incompatible chains, and state explicitly whether the design optimises anchor survivability or third-party verifier burden — the two pull in opposite directions.
6. **Explicitly out of scope:** FUSE vaults as the primary mechanism; encfs; LUKS/VeraCrypt as product dependencies; Anytype code; any GPL/AGPL code linked into the shipped binary.

---

## 6. Open questions and things I could not verify

1. **libSQL encryption** — I confirmed the MIT license but did **not** verify the encryption design, its cipher/KDF, or its production maturity from a primary design document. Treated as WATCH, not rejected.
2. **SQLCipher CVE history** — I found no notable recent SQLCipher CVEs in this pass, but I did **not** exhaustively read a CVE database (the sources I reached were rate-limited or blocked). The claim should be re-checked against NVD/OpenCVE before it appears in a user-facing security page.
3. **macFUSE installation privilege on current macOS** — macFUSE's site says macOS 26's FSKit backend lets supported filesystems "run entirely in user space" with "no more rebooting into recovery", and FUSE-T advertises no kernel extension. I did **not** verify whether installing the macFUSE *package itself* still requires an admin password on every supported macOS version. This is the deciding detail for the "no sudo on macOS" claim about FUSE vaults.
4. **sqlcipher3-binary macOS/arm64 wheels** — as of 0.6.0 the PyPI release lists Linux x86_64 wheels only. It is possible newer releases add macOS wheels, or that a different distribution channel does; re-check before finalising the packaging plan.
5. **msgvault transcription** — msgvault documents "meeting notes" and document/image processing, but I did **not** verify whether it transcribes meeting *audio* (and certainly not Malay/English code-switching). Do not assume it does.
6. **KeePassXC KDBX4 parameter details** — Argon2 + AES-256/ChaCha20 + HMAC-SHA256 is widely documented, but I read it from secondary sources rather than the KDBX4 specification file; treat the exact parameter names as indicative.
7. **"No existing whole product" claim** — this is a judgement from the candidates surveyed, not an exhaustive search. The strongest counterexample found was msgvault, and it lacks encryption; if a project exists that does encrypted-at-rest + semantic search + capture + MCP, I did not find it.
8. **PDPA 2010 / s.129 interaction** — out of scope for this memo (covered by the separate PDPA boundary memo); the encryption choice here should be validated against the evidence/retention requirements recorded there.

9. **Durability vs encryption (the ArchiveBox tension).** ArchiveBox argues an archive should stay readable in 50-100 years without the tool, which is why it stores plain files; store-level encryption works against that. footprint must decide how to reconcile append-only, independently verifiable artifacts with a store that is opaque without the key. **I originally wrote that the manifest split was unvalidated; that is now corrected, and the framing has been revised twice.** Sections 3.7 and 3.8 work it through. In short: nobody implements the *encrypted* version of the split; **restic is the anti-pattern for our purpose** (keyed integrity means verification requires the key) though it is the *correct* choice for a backup tool; **WARC + CDX/CDXJ is the verified structural precedent** (per-record, algorithm-agile digests plus a separate index) but its plaintext manifest is not free; **Maildir/notmuch** supplies the derivability property; **external timestamping is greenfield**. Critically, a plaintext digest manifest is itself a metadata leak and a **confirmation oracle** (a published attack class — see the Proofs-of-Ownership paper in 3.8), so the ADR must choose deliberately among plaintext-unkeyed, separately-keyed, and ciphertext-digest manifests — or the signed-Merkle-tree-over-ciphertext proposal in 3.8 — rather than inheriting option 1 because WARC made it look free. Settle this in an ADR; do not assume it. And note the trust gap that makes or breaks the whole scheme (section 3.9): a signed tree head from a single-user archive with no monitors and no gossip can be re-signed after a rewrite, so the structure buys *proofs relative to a head*, not append-only-ness. The external pin (RFC 3161 / OpenTimestamps / publication) is therefore the load-bearing component, not an add-on — and a leaf timestamp is self-asserted in the same way and needs the same anchoring.

10. **Long-horizon verifiability of the pin — and it is not symmetric.** oss-archive-platforms flagged that both pins carry multi-decade risk; on checking, the risk is **not** the same on both sides, and I corrected my own earlier framing in section 3.9 accordingly. RFC 3161 has a protocol-level answer to certificate expiry and revocation: a token "prove[s] that a digital signature was generated during the validity" period and supports "verifying signatures created prior to the time of revocation" (lines 51, 73-77), with verification bounded by the signer's certificate validity period and the revocation required to postdate the timestamp (lines 1137-1156). OpenTimestamps' dependency — Bitcoin's continued existence and security model, plus the OTS format staying interpretable — has no analogous protocol-level mitigation. What remains unsolved for **both** is trust-anchor/CA-root distrust (a root removed from trust stores decades later is not covered by the RFC 3161 provision) and hash-algorithm longevity. Also unverified: whether real TSAs actually ship the long-term-validation material (full chain, CRLs) that the RFC's procedure assumes — that is a deployment question I did not check. Recommendation: treat "which pin" as a genuine **three-axis** trade (trust model / latency / revocation handling) rather than a settled default. Carrying **both** an OTS proof and an RFC 3161 token is a stronger idea than pure redundancy — the two are complementary on different axes, and the TSA's immediate return actually gives a *tighter* capture bound than OTS's few-hour pending window — but it costs the **union** of the verification requirements, so it trades anchor survivability against verifier burden, and the ADR must say which it optimises. See section 3.9. Still my suggestion, not a verified practice. The OTS latency/tooling lead I flagged as unverified is now **closed** (see section 3.9): latency is a few hours, and verification requires a Bitcoin Core node — which means third-party verifiability is real but not frictionless. The format-stability half is partially answered: the client README notes that format changes mean users "just need to upgrade your client software; existing timestamps will be [upgraded]", so the sharper long-term risk is not the format but the **incomplete-proof state**, which the upgrade rule in section 3.9 makes controllable by us.
