# OSS Transcription and Diarization for Malay/English Code-Switched Meetings

*Research memo for the **footprint** project. Verified against primary sources on 2026-09-25. Licenses were read from the actual `LICENSE` file in each repository, or from the Hugging Face model API `cardData.license` field — not from blog posts.*

---

## 1. The question, and the short answer

**Question.** For a distributable, single-user, participant-only archive that must transcribe Malaysian workplace meetings where Malay and English are code-switched mid-sentence, which free/open-source speech-to-text and speaker-diarization components are actually usable — legally, technically, and on the reference hardware?

**Short answer.** The *plumbing* is solved and permissively licensed: `faster-whisper` (MIT), `whisper.cpp` (MIT), OpenAI Whisper weights (Apache-2.0/MIT), `silero-vad` (MIT) and `pyannote-audio`'s *code* (MIT) can all be shipped inside a product without a copyleft or non-commercial problem. The *Malay/English code-switching accuracy* is **not solved by any permissively-licensed, ready-to-use OSS component**, and this is a genuine gap, not a documentation gap:

- Whisper is monolingual by design. Its decoder is conditioned on a **single language token** per 30-second window. A paper adapting Whisper for code-switching states it plainly: *"Whisper was never intended to be a solution to code switching, and was only trained to be used in a monolingual context"* ([2506.00291](https://arxiv.org/html/2506.00291v1)). There is a known family of workarounds (multiple language prompts, language-aware adapters, per-segment detection), but none is a drop-in switch.
- The only published **Malay–English code-switching** WER evidence I could find puts off-the-shelf Whisper at roughly **21% average WER on code-switched test audio** and **~30% on monolingual Malay**, with the best research adaptation only reaching ~17% ([AsyncSwitch, arXiv 2506.14190](https://arxiv.org/abs/2506.14190), Table II). These are *research* numbers on a Singapore/Malaysia corpus, and the framework is not released as a model.
- The **best Malaysian-specific models** (Mesolitica's `malaysian-whisper-*` / `Malaysian-whisper-large-v3-turbo-v3`) are explicitly trained on Malay + Manglish + Mandarin + Tamil, are not gated, and are technically the closest fit — **but their model weights have no declared license at all**. That is a hard blocker for a distributable product until the author clarifies it.
- **Diarization** is optional for this product and currently carries the most license friction (gated Hugging Face access; one major alternative is non-commercial). It should be treated as a v2 feature, not a launch dependency.

**Net:** ship a permissive Whisper-based pipeline now, with the language hint left unset and audio chunked so language detection can re-run per segment; treat Mesolitica's Malaysian models as the accuracy upgrade *gated on a license conversation*; treat true code-switching quality as an open risk to be measured with a spike, not assumed.

---

## 2. Comparison matrix

| Candidate | License (verified) | What it covers | What it does NOT cover | Last release / activity | Verdict |
|---|---|---|---|---|---|
| **OpenAI Whisper** (code + weights) | Code **MIT**; weights **Apache-2.0** (`whisper-large-v3`, `whisper-small`) / **MIT** (`whisper-large-v3-turbo`) | 99 languages incl. Malay (`"ms": "malay"`); robust, well-tooled baseline | Code-switching: single language token per window; no CS benchmark; model card admits "strong ASR results in ~10 languages" only | Weights 2023–2024; repo maintained | **ADOPT** as baseline engine, with a documented CS weakness |
| **faster-whisper** (CTranslate2) | **MIT** | 4× faster than reference Whisper; INT8/FP16; CUDA + CPU; no system FFmpeg needed | No diarization, no alignment; inherits Whisper's monolingual limitation | v1.1.0 benchmarks; actively maintained | **ADOPT** as the default runtime |
| **whisper.cpp** | **MIT** | CPU/Metal/Vulkan; no-Python, no-sudo path; good for Apple Silicon & GPU-less hosts | Same Whisper limitations; slower than CUDA faster-whisper on NVIDIA | Actively maintained (ggml-org) | **ADOPT** as the no-GPU/portable runtime |
| **WhisperX** | **BSD-2-Clause** | VAD-batched ASR, word-level timestamps, diarization glue; claims ~70× realtime (large-v2) | **No Malay alignment model** — `DEFAULT_ALIGN_MODELS_HF` has no `ms` entry, so word timestamps fail for Malay; pulls in gated pyannote | v3.x; actively maintained | **ADOPT-AS-PATTERN** (copy VAD-batching + alignment design; do not depend wholesale) |
| **distil-whisper** | **MIT** (code) | Distillation recipe; official distilled checkpoints are English-only | No Malay/Manglish distilled checkpoint from HF itself | Maintained | **ADOPT-AS-PATTERN** (recipe; Mesolitica applied it to Malay) |
| **Mesolitica / malaya-speech** (code) | **MIT** (Copyright HUSEIN ZOLKEPLI) | Malaysian speech toolkit: ASR, VAD, speaker diarization, speaker verification, STT/TTS, language detection | Library, not a product; heavy TF+PyTorch deps; diarization quality not benchmarked publicly | Last commit 2025-09-25 | **ADOPT** (code) / **WATCH** (weights, see below) |
| **Mesolitica Malaysian Whisper weights** | **UNDECLARED** (no license tag, no LICENSE file) | `Malaysian-whisper-large-v3-turbo-v3`: ms+en+zh+ta, Manglish-aware, non-gated, adds `<|transcribeprecise|>` word timestamps | No declared license → legally "all rights reserved"; no published CS WER; trained partly on NC/ND data | v3 released 2025-07 | **WATCH** — best technical fit, blocked on licensing |
| **NVIDIA NeMo** (framework) | **Apache-2.0** | Framework for ASR + diarization; Sortformer/Canary model families | **No Malay STT model** (Canary = en/de/es/fr; Parakeet = en); diarization weights NC | Actively maintained | **REJECT** for this scope (no Malay; NC diarization weights) |
| **NeMo Sortformer diarization** (`nvidia/diar_sortformer_4spk-v1`) | **CC-BY-NC-4.0** | 4-speaker diarization, DER ~5.9–14.8 on CALLHOME/DIHARD | **Non-commercial** — unusable in a distributable/commercial product | 2026-09 | **REJECT** (NC license) |
| **pyannote-audio** (code) | **MIT** (Copyright CNRS) | Reference diarization pipeline, VAD, overlap detection | Code license does not cover the model weights | Actively maintained | **ADOPT-AS-PATTERN** (design) |
| **pyannote models** | `speaker-diarization-3.1` **MIT** (gated); `speaker-diarization-community-1` **CC-BY-4.0** (gated) | State-of-the-art diarization; WhisperX default | **Gated**: each end user needs an HF account + token + accepts conditions; onboarding friction & network dependency | community-1 2025-09 | **WATCH** — usable but gated; diarization optional anyway |
| **Silero VAD** | **MIT** | Lightweight, accurate voice activity detection; WhisperX alternative VAD | Not a recognizer | Maintained | **ADOPT** |
| **Qwen3-ASR** (`Qwen/Qwen3-ASR-1.7B`) | **Apache-2.0** | LLM-based ASR, 30 languages incl. **Malay (ms)** + language ID + streaming; forced aligner in 11 langs | **No published code-switching benchmark**; LLM-based, heavier/slower; 1.7B | Released 2026-01 | **WATCH** — permissive and Malay-capable, CS quality unproven |
| **Meta MMS / SeamlessM4T** | **CC-BY-NC-4.0** | 1,100+ languages incl. Malay | **Non-commercial**; the AsyncSwitch paper uses MMS as a *baseline* | 2023–2024 | **REJECT** (NC license) |
| **wav2vec2 / XLS-R Malay fine-tunes** | Base XLS-R **Apache-2.0**; no ready Malay CTC model found | Could serve as a Malay forced-aligner base | **No maintained Malay (ms) CTC fine-tune exists** on HF — searches return Malayalam, not Malay | n/a | **WATCH** (gap; would require training) |
| **AsyncSwitch** (research framework) | Paper only; no released weights found | Best published Malay–English CS result (9.02% relative WER reduction; CS avg 17.04%) | Not a product; no public checkpoint; needs paired CS speech data | arXiv 2025-06 | **WATCH** (methodology reference) |

---

## 3. Per-candidate detail and citations

### 3.1 OpenAI Whisper — the baseline, and why code-switching is its weak spot

- **Repo / license:** <https://github.com/openai/whisper> — `LICENSE` is **MIT** (Copyright (c) 2022 OpenAI), read directly from `raw.githubusercontent.com/openai/whisper/main/LICENSE`.
- **Weights license (separate artifact):** Hugging Face model API metadata shows `openai/whisper-large-v3` and `openai/whisper-small` as **`license:apache-2.0`**, and `openai/whisper-large-v3-turbo` as **`license:mit`**; none is gated.
- **Malay support:** Whisper's tokenizer lists `"ms": "malay"` among its languages (<https://github.com/openai/whisper/blob/main/whisper/tokenizer.py>).
- **The code-switching problem (verified):** Whisper conditions decoding on a **single language token**. OpenAI's own model card is candid about the consequences: *"The models are primarily trained and evaluated on ASR and speech translation to English tasks. They show strong ASR results in ~10 languages,"* and *"Our models perform unevenly across languages… lower accuracy on low-resource and/or low-discoverability languages"* (<https://huggingface.co/openai/whisper-large-v3>). It makes **no code-switching claim**.
- **Independent confirmation:** *"Whisper was never intended to be a solution to code switching, and was only trained to be used in a monolingual context. However, due to the nature of pre-training, some English words have been naturally incorporated into the vocabulary of non-English tokens"* — *Improving Code Switching with Supervised Fine Tuning and GELU Adapters*, [arXiv 2506.00291](https://arxiv.org/html/2506.00291v1). The same paper notes the known workaround family: two language ID tokens in the prompt ("Prompting Whisper"), insertion tokens between language segments, and mid-sequence language tokens. It found the simplest method (two-language tokenization) worked best of those tried.

**Answer to "does the language hint actually break code-switched audio?"** — Yes, in the sense that a single forced language token *disables* the model's ability to switch languages within the decode window. Setting `language="ms"` tells the decoder to emit Malay throughout, which tends to mangle or translate English spans; setting `language="en"` does the reverse; and auto-detection picks **one** language for the window. The widely used mitigations (which I label as engineering inference, not a documented guarantee) are: (a) leave the language hint unset so detection runs per window, (b) cut audio into short segments (VAD-delimited) so detection re-runs often, (c) set `condition_on_previous_text=False` so a wrong-language context does not propagate, and (d) fine-tune on code-switched data. Only (d) is shown to work well in the literature, and it needs code-switched training audio.

### 3.2 faster-whisper and whisper.cpp — the runtime layer (both fine)

- **faster-whisper** — <https://github.com/SYSTRAN/faster-whisper>, `LICENSE` **MIT** (Copyright (c) 2023 SYSTRAN), read from the raw file. Its own README benchmark, run on an **RTX 3070 Ti 8 GB**, transcribes 13 minutes of audio with `large-v2` in **1 m 03 s at FP16** (RTF ≈ 0.08×) or **16 s at INT8 with `batch_size=8`** (RTF ≈ 0.02×), using ~4.5 GB VRAM. It depends on **CTranslate2**, which is also **MIT**.
- **whisper.cpp** — <https://github.com/ggml-org/whisper.cpp>, `LICENSE` **MIT** (Copyright (c) 2023-2026 The ggml authors). Best fit for the *no-GPU / no-sudo / Apple Silicon* path; the faster-whisper README's CPU table shows `whisper.cpp` roughly 2–3× faster than `faster-whisper` on CPU for the `small` model.

### 3.3 WhisperX — great design, but its signature feature is broken for Malay

- **Repo / license:** <https://github.com/m-bain/whisperX> — `LICENSE` **BSD-2-Clause** (Copyright (c) 2024, Max Bain), read from the raw file.
- **What it adds:** VAD-based batching (the README claims *"70× realtime with large-v2"* — note the README does not state the GPU, so treat this as a vendor claim), and word-level timestamps via forced phoneme alignment with wav2vec2.
- **The verified Malay gap:** in `whisperx/alignment.py`, `DEFAULT_ALIGN_MODELS_TORCH` covers `{en, fr, de, es, it}` and `DEFAULT_ALIGN_MODELS_HF` covers ~35 languages — **there is no `ms` entry** (it has `ml` = Malayalam, `id` = Indonesian, `tl` = Filipino, but not Malay). The code logs *"No default alignment model for language: ms"* and word timestamps are simply unavailable unless the caller supplies a Malay wav2vec2 model. The README also warns *"Diarization is far from perfect"* and *"Language specific wav2vec2 model is needed."*
- **Diarization coupling:** WhisperX now defaults to `pyannote/speaker-diarization-community-1`, and the README states you must *"accept the user agreement"* and pass an HF token — i.e. it inherits pyannote's gating (see §3.5).

**Verdict: ADOPT-AS-PATTERN.** Copy the VAD-batching and the "align-then-assign-speakers" architecture; do not adopt WhisperX wholesale, because its alignment path is dead for Malay and it drags a gated dependency into the install.

### 3.4 Mesolitica / malaya-speech — the closest Malaysian fit, and the licensing trap

- **Code:** <https://github.com/malaysia-ai/malaya-speech> — `LICENSE` **MIT** (Copyright (c) 2020 HUSEIN ZOLKEPLI), read from the raw file. Last commit **2025-09-25** (i.e. current). The toolkit covers ASR, VAD, speaker diarization, speaker verification, TTS and more (<https://malaya-speech.readthedocs.io/>).
- **Weights:** hosted at <https://huggingface.co/mesolitica>. The most relevant model is `mesolitica/Malaysian-whisper-large-v3-turbo-v3`, whose model card declares languages **ms, en, zh, ta**, base model `openai/whisper-large-v3-turbo`, and adds a `<|transcribeprecise|>` token for word-level timestamps. It is explicitly built for Malaysian speech including **Manglish**.
- **The blocker (verified):** querying the Hugging Face models API for `mesolitica/malaysian-distil-whisper-large-v3`, `mesolitica/malaysian-whisper-small-v2`, `mesolitica/Malaysian-whisper-large-v3-turbo-v3` and others returns **no `license` field in `cardData`**, and the repositories contain **no `LICENSE` file** (confirmed from the file listing). None is `gated`. Under default copyright, "no license" means **all rights reserved** — the *code* is MIT, but the *weights* are a separate artifact that has not been licensed.
- **Provenance concern:** the training data compounds this. `malaysia-ai/malay-conversational-speech-corpus` is a mirror of a MagicHub corpus released under **CC BY-NC-ND 4.0** (non-commercial, no-derivatives), and `mesolitica/IMDA-STT` derives from Singapore's IMDA National Speech Corpus under its own terms. The mesolitica STT datasets declare no license either.

**Verdict:** the *code* is ADOPT; the *weights* are **WATCH** — technically the best Malay/Manglish option in existence, but not shippable inside a distributable product until the author states a license and the training-data terms are reconciled. This is the single highest-value item to resolve by asking the author directly.

### 3.5 pyannote-audio — diarization, gated, and optional anyway

- **Code:** <https://github.com/pyannote/pyannote-audio> — `LICENSE` **MIT** (Copyright (c) 2020 CNRS), read from the raw file.
- **Models (verified via the HF models API):**
  - `pyannote/speaker-diarization-3.1` — `license: mit`, **`gated: "auto"`**. Its gated prompt says the pipeline *"uses MIT license and will always remain open-source"* but will email users about *"premium pipelines and paid services."*
  - `pyannote/segmentation-3.0` — `license: mit`, gated.
  - `pyannote/speaker-diarization-community-1` — `license: cc-by-4.0`, gated. This is WhisperX's current default. CC-BY-4.0 is permissive (attribution only) and **not** copyleft, so it is *not* a distribution blocker in the way AGPL would be — but it is a content license, not a software license, and the attribution obligation must be honoured in the product's notices.
- **The real friction is the gate, not the license.** `gated: "auto"` means every end user of a distributed footprint install must create a Hugging Face account, accept pyannote's conditions, generate an access token, and let the app authenticate over the network. For a product whose selling point is "install locally, no account, no API key, nothing leaves your machine," that is a meaningful contradiction.
- **NeMo's alternative is worse:** `nvidia/diar_sortformer_4spk-v1` is **`license: cc-by-nc-4.0`** — non-commercial. For a distributable product this is a **REJECT**.

**Does diarization matter for a participant-only archive?** Partly. The user is always a participant, so "who spoke" matters for recall (*"what did the client say about the deadline?"*), but the *identity* of the other speakers is often already available: platform transcripts (Teams/Zoom/Meet) carry speaker attribution natively when captured via delegated per-user OAuth, and for the user's own speech the label is trivially "me". Diarization's unique value is for **locally recorded audio where no platform transcript exists** — in-person meetings, or calls captured from the system audio. That is a real but secondary path. Recommendation: **defer diarization to v2**, and when it lands, prefer an ungated or self-hosted approach over pyannote's gate.

### 3.6 Silero VAD — cheap, MIT, adopt

`LICENSE` **MIT** (Copyright (c) 2020-present Silero Team), read from the raw file. It is a small, well-regarded VAD and is already an alternative VAD inside WhisperX. Because VAD-segmented audio is also the mechanism that lets Whisper re-run language detection per segment, Silero VAD is useful beyond just trimming silence. **ADOPT.**

### 3.7 Qwen3-ASR — the most interesting newcomer (permissive, Malay-capable)

`Qwen/Qwen3-ASR-1.7B` is **`license:apache-2.0`** and not gated; the `QwenLM/Qwen3-ASR` repository LICENSE is Apache-2.0. Its model card lists **Malay (ms)** explicitly among 30 supported languages (plus 22 Chinese dialects), with built-in language identification and streaming, and a separate `Qwen3-ForcedAligner-0.6B` for timestamps. Because it is an LLM-based recognizer rather than Whisper's fixed-token decoder, it *may* handle intra-sentence switching better — **but the model card makes no code-switching claim and I found no code-switching benchmark**, so this is unverified. Also note the model is 1.7 B parameters, so it is heavier and likely slower than `large-v3-turbo`. **WATCH** — worth a measured spike, especially as a fallback if Mesolitica licensing cannot be resolved.

### 3.8 Non-commercial traps to avoid

Both of these are frequently recommended for multilingual ASR and both are **CC-BY-NC-4.0** (verified via the HF models API), i.e. non-commercial and therefore unusable in a distributable product:

- `facebook/mms-1b-all` — **cc-by-nc-4.0**.
- `facebook/seamless-m4t-v2-large` — **cc-by-nc-4.0**.

The AsyncSwitch paper uses MMS-1B-All and SeamlessM4T-v2 as *baselines*; do not copy those choices into a product without checking the license.

---

## 4. Hardware: what runs on an RTX 3050 8 GB, and the realtime factor

**VRAM is not the constraint.** Whisper `large-v3` needs roughly 3 GB at FP16 and `large-v3-turbo` roughly 1.7 GB; both fit comfortably in 8 GB alongside a small VAD and (optionally) a diarization model. The constraint is **throughput**.

**Primary datapoint** (faster-whisper README, measured on an **RTX 3070 Ti 8 GB**, CUDA 12.4, 13 minutes of audio):

| Config | Time (13 min audio) | RTF | VRAM |
|---|---|---|---|
| `large-v2` FP16, beam 5 | 1 m 03 s | ≈ 0.08× | 4525 MB |
| `large-v2` FP16, `batch_size=8` | 17 s | ≈ 0.02× | 6090 MB |
| `large-v2` INT8, beam 5 | 59 s | ≈ 0.076× | 2926 MB |
| `large-v2` INT8, `batch_size=8` | 16 s | ≈ 0.02× | 4500 MB |

An RTX 3050 8 GB has materially less compute than a 3070 Ti, so expect **RTF roughly in the 0.15–0.3× range** for `large-v3`-class models at FP16, and better with INT8 and batching. A third-party aggregation lists an RTF of **~0.15×** for `large-v3` on an RTX 3050 8 GB (<https://gigagpu.com/whisper-vram-requirements/>) — that is a **secondary source and I could not independently verify it**, so treat it as an estimate. Practically: on this GPU a one-hour meeting should transcribe in roughly 10–20 minutes of wall-clock, which is fine for an archive that processes recordings after the fact rather than live. `whisper.cpp` remains the fallback for hosts without a usable NVIDIA GPU.

**Caveat:** I did not benchmark anything on the actual reference host; the numbers above are the maintainers' published benchmark plus one third-party estimate.

---

## 5. Closest fits, and the gap

I actively looked for a single OSS project that already does most of this. **None exists.** The two closest stacks, and what each leaves out:

**Closest fit #1 — Mesolitica `malaya-speech` + Malaysian Whisper weights.**
This is the only stack that is *Malaysian by construction*: a Malay/Manglish ASR model, a VAD, speaker diarization and speaker verification in one MIT-licensed library, with non-gated weights. **Gap:** the weights have **no declared license** (the decisive blocker); there is **no published Malay–English code-switching WER** to trust; the library pulls in both TensorFlow and PyTorch; and it is a research library, not an installable end-user product.

**Closest fit #2 — `faster-whisper` + VAD batching + `pyannote` (the WhisperX shape).**
This is the best-engineered, best-licensed *pipeline*: MIT/BSD code throughout, fast, word timestamps, diarization glue. **Gap:** Whisper's monolingual decoding is the accuracy ceiling for code-switching; WhisperX's Malay forced-alignment model **does not exist**, so its headline word-timestamp feature is unavailable for Malay; and pyannote's gate forces every end user to obtain a Hugging Face token.

**The specific gap, stated plainly:** there is **no OSS component that is simultaneously (a) licensed for redistribution, (b) accurate on intra-sentence Malay/English switching, and (c) benchmarked on Malaysian speech.** Each candidate satisfies at most two of the three. The published evidence for Malay–English code-switching (AsyncSwitch, and the Universiti Malaya bilingual-system paper cited in the companion workplace-stack memo) is **research, not productised**: AsyncSwitch reports ~21% average code-switched WER for off-the-shelf Whisper and ~17% after adaptation, and no checkpoint was released.

**Implication for footprint:** treat code-switching quality as an *open risk to be measured*, not a solved input. The practical plan is (1) ship a permissive faster-whisper/whisper.cpp pipeline with the language hint unset and VAD-based chunking so detection can re-run per segment; (2) run a spike comparing plain `large-v3-turbo`, Mesolitica's `Malaysian-whisper-large-v3-turbo-v3`, and Qwen3-ASR on a handful of real code-switched meetings; (3) in parallel, ask Mesolitica to declare a weight license; (4) defer diarization.

---

## 6. Open questions and things I could not verify

1. **Mesolitica weight licensing.** No `license` field and no `LICENSE` file on any Malaysian Whisper model I checked. Whether the author intends MIT (matching the code) or something else is unknown, and the training data includes CC BY-NC-ND and IMDA-derived corpora. **This must be asked directly before any Mesolitica weight can ship.**
2. **No verified Malay–English code-switching benchmark for any *shippable* model.** The AsyncSwitch numbers are the only CS numbers I found, they are on a Singapore/Malaysia corpus, and the model is not released. There is **no** published CS WER for Mesolitica's models, for Qwen3-ASR, or for any Whisper checkpoint. Anything I say about real-world accuracy on Malaysian meetings is inference, not measurement. The same absence appears to extend past STT: the companion agent-interface memo (`docs/research/oss/agent-interface.md`, §5) reports no Malay/English code-switching benchmark for retrieval tooling either, so if footprint ever embeds or reranks code-switched transcript text, that quality claim is likewise unbenchmarked.
3. **The RTX 3050 RTF is a third-party estimate.** I did not benchmark on the reference host. The primary benchmark is on an RTX 3070 Ti 8 GB.
4. **Whether "leaving the language hint unset + short VAD segments" measurably helps.** This is a reasonable engineering inference from how Whisper's single-token decoding works, but I found no controlled study quantifying it. The literature's own answer is fine-tuning on code-switched data.
5. **Diarization quality on Malaysian meeting audio** (accents, overlapping speech) is unmeasured; pyannote's published benchmarks are on English/telephone corpora.
6. **Qwen3-ASR's code-switching behaviour** is unverified — it lists Malay and has language identification, but no CS claim. Worth a spike.
7. **`whisper.cpp`'s and `faster-whisper`'s exact latest release tags** were not pinned to a version in this memo; the licenses are stable MIT regardless.
8. **Any AGPL/GPL component in this space.** I found none among the candidates surveyed — every core component here is MIT, BSD-2-Clause or Apache-2.0. The binding constraints are **CC-BY-NC-4.0** (MMS, SeamlessM4T, NeMo Sortformer) and **undeclared licenses** (Mesolitica weights), not copyleft.

---

## Sources (primary)

- OpenAI Whisper code + LICENSE: <https://github.com/openai/whisper> · <https://raw.githubusercontent.com/openai/whisper/main/LICENSE> · tokenizer: <https://github.com/openai/whisper/blob/main/whisper/tokenizer.py>
- Whisper weights license metadata: <https://huggingface.co/api/models/openai/whisper-large-v3> · <https://huggingface.co/api/models/openai/whisper-large-v3-turbo> · model card: <https://huggingface.co/openai/whisper-large-v3>
- faster-whisper: <https://github.com/SYSTRAN/faster-whisper> · LICENSE: <https://raw.githubusercontent.com/SYSTRAN/faster-whisper/master/LICENSE>
- whisper.cpp: <https://github.com/ggml-org/whisper.cpp> · LICENSE: <https://raw.githubusercontent.com/ggml-org/whisper.cpp/master/LICENSE>
- WhisperX: <https://github.com/m-bain/whisperX> · LICENSE: <https://raw.githubusercontent.com/m-bain/whisperX/main/LICENSE> · alignment maps: <https://raw.githubusercontent.com/m-bain/whisperX/main/whisperx/alignment.py>
- distil-whisper: <https://github.com/huggingface/distil-whisper> · LICENSE: <https://raw.githubusercontent.com/huggingface/distil-whisper/main/LICENSE>
- malaya-speech: <https://github.com/malaysia-ai/malaya-speech> · LICENSE: <https://raw.githubusercontent.com/malaysia-ai/malaya-speech/master/LICENSE> · docs: <https://malaya-speech.readthedocs.io/>
- Mesolitica models: <https://huggingface.co/mesolitica/Malaysian-whisper-large-v3-turbo-v3> · <https://huggingface.co/mesolitica/malaysian-distil-whisper-large-v3> · API metadata (no license): <https://huggingface.co/api/models/mesolitica/malaysian-whisper-small-v2>
- Malay conversational corpus license (CC BY-NC-ND 4.0): <https://huggingface.co/api/datasets/malaysia-ai/malay-conversational-speech-corpus>
- pyannote-audio: <https://github.com/pyannote/pyannote-audio> · LICENSE: <https://raw.githubusercontent.com/pyannote/pyannote-audio/main/LICENSE>
- pyannote model licenses/gating: <https://huggingface.co/api/models/pyannote/speaker-diarization-3.1> · <https://huggingface.co/api/models/pyannote/speaker-diarization-community-1> · <https://huggingface.co/api/models/pyannote/segmentation-3.0>
- NeMo: <https://github.com/NVIDIA/NeMo> · LICENSE: <https://raw.githubusercontent.com/NVIDIA/NeMo/main/LICENSE> · Sortformer (CC-BY-NC-4.0): <https://huggingface.co/api/models/nvidia/diar_sortformer_4spk-v1>
- Silero VAD: <https://github.com/snakers4/silero-vad> · LICENSE: <https://raw.githubusercontent.com/snakers4/silero-vad/master/LICENSE>
- Qwen3-ASR: <https://github.com/QwenLM/Qwen3-ASR> · model card: <https://huggingface.co/Qwen/Qwen3-ASR-1.7B>
- MMS / SeamlessM4T (CC-BY-NC-4.0): <https://huggingface.co/api/models/facebook/mms-1b-all> · <https://huggingface.co/api/models/facebook/seamless-m4t-v2-large>
- AsyncSwitch, *Asynchronous Text-Speech Adaptation for Code-Switched ASR* (Malay–English WER): <https://arxiv.org/abs/2506.14190>
- *Adapting Whisper for Code-Switching through Encoding Refining and Language-Aware Decoding* (SEAME; MER reductions): <https://arxiv.org/abs/2412.16507>
- *Adapting Whisper for Parameter-efficient Code-Switching Speech Recognition via Soft Prompt Tuning*: <https://arxiv.org/abs/2506.21576>
- *Improving Code Switching with Supervised Fine Tuning and GELU Adapters* ("Whisper was never intended to be a solution to code switching"): <https://arxiv.org/html/2506.00291v1>
- RTX 3050 `large-v3` RTF estimate (secondary, unverified): <https://gigagpu.com/whisper-vram-requirements/>
