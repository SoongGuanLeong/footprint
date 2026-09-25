# Platform support and distribution gates

*Research memo for wayfinder ticket #5, grilling question Q8. Written 2026-09-25. Informs which platforms footprint supports in v1 and in what order.*

---

## Summary

**The technical stack is genuinely cross-platform. Sequencing is therefore driven almost entirely by distribution economics, not by code.** No platform in the lineup forces a FUSE dependency or an end-user build toolchain.

The recommendation that follows from this research is **Windows first, macOS second, Linux last** — the inverse of what was assumed before this research, which had Linux first because it is the development host.

- **Windows** — ~70% of Malaysian desktops by the only available measure; the entire stack works including CUDA; and the distribution problem has a genuinely **free** solution (Microsoft Store MSIX, where Microsoft re-signs).
- **macOS** — ~10% of desktops, and the only platform with a mandatory, non-bypassable, recurring gate ($99/yr plus notarisation plus a Mac in the build loop). It is also where the stack is *weakest*, because CTranslate2 has no GPU path on macOS.
- **Linux** — ~3% of desktops, so it buys the least reach, but it is the only platform with no distribution gate and no recurring cost, and it is the development host, so it rides along cheaply as a tarball or AppImage.

**The biggest surprise:** the cheap modern Windows signing route, Azure Artifact Signing, is geo-fenced away from Malaysia — and it is the one option that is both cheap *and* exempt from the CA/Browser Forum hardware-token mandate. That is what pushes a Malaysian developer onto the more expensive and more awkward OV-certificate path, and it is the strongest argument for the free MSIX route.

---

## 1. Desktop OS share in Malaysia

Source: StatCounter GlobalStats, `os-market-share/desktop/malaysia`, 12-month average Oct 2025 – Sep 2026:

| | Windows | macOS | Linux | ChromeOS | Unknown |
|---|---|---|---|---|---|
| **Malaysia** | **70.4%** | 9.6% | 3.2% | 5.4% | 11.4% |
| Singapore | 60.7% | 20.4% | 3.2% | — | 15.5% |
| Indonesia | 78.8% | 8.1% | 3.7% | — | 9.2% |
| Philippines | 75.3% | 8.4% | 2.3% | — | 9.0% |

Latest single month (Sep 2026) for Malaysia: Windows 74.8% (Win11 60.9, Win10 13.9), macOS 11.0%, Linux 5.4%.

**Caveats that matter more than the numbers:**

- These are **consumer web-traffic figures, not enterprise**. StatCounter counts page views, so it is weighted to consumer browsing, not corporate fleets. It is the wrong instrument for the enterprise question.
- The Unknown bucket is large and volatile (Malaysia swung 3.9%–15.7% month to month), so any single month is noisy.
- Malaysia's ~5% ChromeOS figure is anomalously high against Singapore's and Indonesia's ~0.2%. Almost certainly misclassification; distrust it.

**No credible Malaysia-specific enterprise desktop OS split exists in public sources.** Checked and found wanting: MCMC Internet Users Survey (consumer survey, does not report desktop OS); DOSM *ICT Use and Access by Individuals and Households* (Malaysia-specific and authoritative, but measures computer *access*, not OS — 2025 report: household computer access 92.6%, individual computer usage 81.5%); MDEC and MAMPU (publish no OS splits); GlobalData's *Malaysia Enterprise ICT Country Intelligence Report* (paywalled). Microsoft and Google Malaysia announcements contain no fleet OS data. No global enterprise number has been substituted here, deliberately.

**One defensible inference:** Windows 10 is still ~14% of Malaysian desktops, so a Windows build **cannot assume Windows 11**.

---

## 2. Per-platform technical viability

### SQLCipher / SQLite3MultipleCiphers — and a correction

**This corrects an earlier claim in the OSS reuse survey.** That survey said the SQLCipher Python wheels are Linux-x86_64 only and that a build or vendor step is therefore required for a cross-platform product. The claim was literally true of the package name it named, and stale.

- `sqlcipher3-binary` — Linux-x86_64 only, confirmed across all 73 files of all 13 versions it ever published. It has **never** shipped a macOS, Windows or aarch64 wheel.
- **`sqlcipher3` 0.6.2** (uploaded 2026-01-07) — **78 wheels covering all three platforms**: macOS universal2/x86_64/arm64; Linux manylinux_2_28 x86_64/aarch64/i686 and musllinux_1_2 x86_64/aarch64; Windows win32/win_amd64/win_arm64. Wheels were downloaded and unpacked to confirm they are real, not stubs: win_amd64 is a single 5.9 MB `_sqlite3.cp312-win_amd64.pyd`, manylinux is a 21 MB `.so`, both statically bundling SQLCipher 4.x (confirmed by `cipher_version` and `sqlcipher_codec_ctx_init_kdf_salt` symbols). Licence MIT. **No build toolchain needed on any platform.**
- `sqlcipher3-wheels` (laggykiller fork) is also cross-platform but pins SQLCipher 3.x, so superseded; 0.6.2's METADATA credits laggykiller as co-author — the efforts merged.
- **SQLite3MultipleCiphers** (utelle, MIT) is the opposite story: v2.5.1 ships prebuilt binaries **for Windows only**. macOS and Linux get amalgamation source to compile. Fine when compiling into your own native module, but not a drop-in prebuilt.

Node bindings, if the app goes Electron/Node: `better-sqlite3-multiple-ciphers` (MIT) **does** bundle prebuilds inside its npm tarball (darwin-arm64/x64, linux-arm64/x64, linuxmusl-arm64/x64, win32-arm64/x64) — it has no GitHub release assets, which makes a naive check look empty. Avoid `@journeyapps/sqlcipher` (BSD-3): its `install` script is `node-gyp rebuild`, so every user needs a full C++ toolchain.

### sqlite-vec — clean everywhere

Wheel 0.1.9: macOS x86_64+arm64, Linux x86_64+aarch64, Windows amd64. npm platform packages for linux-x64/arm64, darwin-x64/arm64, windows-x64. Licence MIT OR Apache-2.0. No Windows-on-ARM build.

### faster-whisper / CTranslate2 — a real platform divergence

- faster-whisper 1.2.1 is pure Python; the native work is `ctranslate2` 4.8.2, which ships wheels for macOS arm64+x86_64, Linux x86_64+aarch64, and Windows amd64.
- CTranslate2's docs state that **the Linux and Windows Python wheels support GPU execution** (CUDA 12.x plus cuDNN 8 for speech models). So **Windows CUDA works.**
- **macOS is CPU-only for CTranslate2.** There is no Metal backend. On Apple Silicon, faster-whisper transcribes on CPU — a genuine capability gap on the platform with the best GPU-per-watt in the lineup.
- **whisper.cpp is the mirror image:** Apple Silicon is a first-class citizen (ARM NEON, Accelerate, Metal, Core ML on the Apple Neural Engine), and it also supports NVIDIA GPU, Vulkan and OpenVINO, building on Windows with MSVC or MinGW.
- **Practical read:** whisper.cpp on macOS and faster-whisper on Windows/Linux — or standardise on whisper.cpp everywhere to avoid two STT paths.

### llama.cpp / Ollama — viable everywhere

llama.cpp: Metal on by default on macOS; CUDA for NVIDIA; Vulkan/HIP/SYCL/OpenCL. Builds on Windows x64 and arm64. Ollama requires NVIDIA compute capability 5.0+ with driver 550+; an RTX 3050 is compute 8.6, so supported. **Install footprint differs:** Ollama's Linux install is root-level (installs to /usr, creates a system user, registers a systemd service), while the Windows installer needs no Administrator and installs into the home directory.

### FUSE — confirmed not needed

SQLCipher, SQLite3MultipleCiphers and sqlite-vec are in-process libraries; llama.cpp/Ollama and Whisper are ordinary userspace processes. **There is no FUSE dependency anywhere in the stack.** The embedded-SQLite decision does avoid a FUSE vault, as expected.

One packaging caveat: **AppImage itself uses FUSE.** Recent AppImages bundle FUSE3 but older ones may need libfuse2 (Ubuntu 24.04+ needs `libfuse2t64`); `--appimage-extract` is the escape hatch. deb/rpm/tarball need none; Flatpak uses bubblewrap.

### Admin rights

Linux: Ollama and deb/rpm installs need root; AppImage and tarball need none. Windows: a per-user installer needs no admin, a per-machine MSI/NSIS does. macOS: signing and notarisation need no runtime admin; `~/Applications` needs none.

### Integration risk: pin the SQLite version

SQLCipher and sqlite-vec do compose. The known failure mode is a **SQLite version mismatch**, not an incompatibility: sqlite-vec's `MATCH ... LIMIT` syntax needs SQLite >= 3.42, and the reported breakage was a bundled SQLCipher carrying SQLite 3.39.4 (sqlite-vec issue #12), fixed by using the `k = 10` constraint form instead of `LIMIT`. Current versions are past this (sqlcipher3 0.6.2 bundles SQLite 3.51.1), but pin it and verify rather than assume, and prefer the `k =` form for portability.

---

## 3. Distribution gates and costs

### macOS — a real gate, unavoidable, recurring

Apple's docs: "Beginning in macOS 10.15, all software built after June 1, 2019, and distributed with Developer ID must be notarized." Notarisation requires code-signing on **every** distributed executable, a **Developer ID** certificate (explicitly not Mac Distribution, ad hoc, Apple Development, or local development certs), Hardened Runtime enabled, and a secure timestamp.

Unsigned or un-notarised apps are blocked by Gatekeeper; the user must find the override (right-click > Open, or System Settings > Privacy & Security > Open Anyway). For an office worker self-installing, that is real drop-off.

**Cost: US$99 per membership year, recurring.** Tooling needs Xcode 14+ (the notary service stopped accepting Xcode 13-or-earlier uploads on 1 Nov 2023), which means **a Mac in the build loop**.

### Windows — no hard gate, but a reputation gate, and the cheap option excludes Malaysia

SmartScreen evaluates publisher reputation and file-hash reputation. Unsigned means "Windows protected your PC" and the user must choose "Run anyway"; enterprise policy can prevent continuation entirely. Signed-but-new also warns until reputation accumulates. An unsigned file accumulates reputation **per file hash**, so every new version restarts from zero; reputation only transfers between versions that share a publisher identity.

**EV no longer bypasses SmartScreen.** Microsoft's own page says so, and states that paying a premium for EV solely to avoid SmartScreen is no longer justified.

| Option | Cost | Availability |
|---|---|---|
| **Microsoft Store MSIX** | **free** | Microsoft re-signs; **no SmartScreen warnings at all** |
| Azure Artifact Signing (ex-Trusted Signing) | ~$9.99/month | **Organizations: USA, Canada, EU, UK. Individuals: USA and Canada only — NOT MALAYSIA** |
| OV certificate | $150–300/year | worldwide |
| EV certificate | $400+/year | worldwide |
| Self-signed / unsigned | free | blocked for public users |

**The geo-fence is the decision-relevant fact.** Re-confirmed verbatim from the Microsoft Learn source. And it is costly rather than merely inconvenient for a specific reason: that same page notes Azure Artifact Signing requires **no hardware token**, integrating directly with CI/CD. It is the one option that is both cheap *and* exempt from the CA/Browser Forum mandate.

**Hardware token requirement, verified at source.** CA/Browser Forum Code Signing Baseline Requirements: "Effective June 1, 2023, for Code Signing Certificates, CAs SHALL ensure that the Subscriber's Private Key is generated, stored, and used in a suitable Hardware Crypto Module." This applies to **all** code-signing certificates, OV included. For a solo developer that means a physical USB token or a paid cloud-HSM signing service, and either an awkward CI story or an extra recurring vendor cost.

**New cost driver:** CA/B Forum ballot CSC-31 — certificates issued on or after 1 March 2026 must not exceed **460 days** validity. So certificate spend is effectively annual-or-less forever, with no multi-year lock-in to amortise.

Individual-specific products exist (SSL.com sells an "IV Code Signing" certificate with a personal name and no business documents), but **pricing could not be verified** (JS-rendered) and whether an IV certificate is SmartScreen-equivalent to OV is an **open question, not a given**.

### Linux — no OS-level gate

No notarisation equivalent, no vendor gatekeeper. Repo signing (GPG for deb/rpm) is free and only matters if running your own repository. Channels: tarball, deb, rpm, AppImage, Flatpak (Flathub review, free), Snap. All free; the cost is CI and packaging time.

---

## 4. What this means for platform sequencing

1. **Windows first, and it is not close.** ~70% of Malaysian desktops by the only available measure; the whole stack works including CUDA; and the distribution problem has a free solution — ship MSIX through the Microsoft Store and Microsoft re-signs it, sidestepping the certificate, the hardware token, the geo-fence and the recurring cost at once. If Store distribution is unacceptable for product reasons (capabilities MSIX forbids, or Store review), the fallback is an OV certificate (~$150–300/yr, hardware token or cloud HSM, renewable at most every 460 days) plus SmartScreen warnings until reputation accumulates. **Target Windows 10 as well as 11.**
2. **macOS second — and the reason is reach, not ease.** ~10% of desktops, and the only mandatory recurring gate. It is also where the stack is weakest: CTranslate2 has no GPU path, so faster-whisper would be CPU-only on Apple Silicon and whisper.cpp with Metal/Core ML is the better choice there. Both facts argue for treating macOS as a deliberate v2 with its own STT path, not a free byproduct of the Linux/Windows build. The Python side is otherwise fine — sqlcipher3 0.6.2 covers macOS arm64/x86_64.
3. **Linux last, despite being the development host.** ~3% of desktops, so it buys the least reach; its attraction is being the only platform with no distribution gate and no recurring cost, and it is the easiest to keep working incidentally since the same wheels and binaries serve it. It can ride along as a tarball or AppImage without a dedicated release effort. If shipping AppImage, verify FUSE3 is bundled.

**The structural point:** because no platform forces FUSE or a build toolchain, sequencing is a distribution-economics question. Windows is cheap and high-reach; macOS is expensive and low-reach; Linux is free and low-reach.

---

## Open questions and unverified

1. Any Malaysia-specific **enterprise/corporate** desktop OS split — none found. The StatCounter figures are consumer web-traffic and must not be presented as enterprise.
2. Thailand's desktop OS split (Unknown >50%, unusable).
3. SSL.com IV code-signing price, and whether IV certificates are SmartScreen-equivalent to OV.
4. A current standalone price for a code-signing hardware token.
5. Whether sqlcipher3 0.6.2 has been tested **together with** sqlite-vec. The SQLite versions are compatible (3.51.1 >= 3.42) but no direct evidence of joint use was found.
6. Whether the MSIX route permits everything footprint needs — relevant if the capture daemon requires capabilities a packaged app cannot hold.
7. Whether an OV certificate is even purchasable by an individual in Malaysia without a registered business entity.

---

## Sources

- StatCounter GlobalStats, desktop OS market share, Malaysia and SEA, CSV export pulled 2026-09-25.
- DOSM, *ICT Use and Access by Individuals and Households*, 2025 report (released 23 Apr 2026).
- Apple, notarisation requirements and Developer Program enrolment page.
- Microsoft Learn, *Code signing options* (`package-and-deploy/code-signing-options`), read from raw HTML.
- CA/Browser Forum, Code Signing Baseline Requirements; ballot CSC-31.
- CTranslate2 installation docs; faster-whisper and whisper.cpp READMEs; llama.cpp and Ollama GPU docs.
- PyPI release metadata and unpacked wheels for `sqlcipher3`, `sqlcipher3-binary`, `sqlcipher3-wheels`, `sqlite-vec`, `ctranslate2`; npm tarball contents for `better-sqlite3-multiple-ciphers`.
- sqlite-vec issue #12 (SQLite version mismatch with bundled SQLCipher).
