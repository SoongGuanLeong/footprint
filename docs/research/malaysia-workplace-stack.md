# Malaysia Workplace Stack Survey — Capture Paths, Consent Requirements, and the Participant-Only Boundary

*Research memo for GitHub issue #2 ("Malaysia workplace stack survey"). Written 2026-09-25.*
*Scope note: this memo covers **technical capture paths and consent mechanics only**. The legal analysis (PDPA 2010, employer policy) is issue #3 and is deliberately out of scope here.*

---

## Summary: the question and the answer

The question is: for a personal, participant-only work archive in Malaysia, **which chat, meeting, and email platforms can actually be read programmatically, by what mechanism, and does that mechanism stay inside a "participant-only" boundary** — meaning the capture is authorised by the individual user's own delegated consent rather than by a tenant-wide administrator grant?

The answer, in short:

1. **A participant-only connector set is technically achievable, and it is broader than the earlier seed notes assumed.** Microsoft Teams 1:1/group chat, Microsoft 365 mail, Gmail, Google Chat, Slack, Zoom cloud recordings, and Google Meet conference metadata can all be read with **delegated per-user OAuth that does not require tenant administrator consent** ([7], [8], [9], [10], [28], [30]).
2. **The boundary bites hardest on exactly the surfaces people assume are easy.** Teams *channel* messages require admin consent even with a delegated permission ([8], [11]); Teams *transcripts and recordings* require admin consent through their dedicated Graph APIs ([8], [12], [13]); bulk Teams export is an application-only "protected API" that "only an administrator can approve" ([14]); and reading *someone else's* mailbox requires application permissions or domain-wide delegation ([8], [20]).
3. **There is an important exception that widens the Teams meeting surface.** Teams recordings and transcripts are ordinary files in the organiser's OneDrive (private meetings) or the channel's SharePoint site (channel meetings) ([17]), and the delegated file permissions `Files.Read`, `Files.Read.All` and `Sites.Read.All` do **not** require admin consent ([8]). So a user can often capture their own meeting artefacts as files without ever touching the admin-gated transcript API.
4. **Personal WhatsApp is a genuine gap, and the seed note needs one correction.** The consumer WhatsApp app has no official API for a user's own history; the only automated route is a per-chat on-device `.txt` export with media stripped ([1]). Meta *does* operate a chat-history synchronisation path (the `history` webhook), but it is restricted to Solution/Tech Providers onboarding a **WhatsApp Business app** number, covers at most 180 days, excludes group chats, and requires the business customer's consent ([3], [4]). It is not a personal-archive path.
5. **Malay/English code-switching is a real, documented transcription gap, but it is narrower than the seed suggested.** Google Meet transcripts and Microsoft's multilingual speech recognition both support a fixed set of languages that **excludes Malay** ([26], [44]); Otter.ai likewise does not support Malay ([43]); Sonix does ([42]). The weak spot is the *native* transcription engines, not the third-party ones.

---

## 1. What "participant-only" means as a technical constraint

Two different consent models are in play, and conflating them is the main source of error in this area.

- **Delegated permissions** are granted by the signed-in user and let an app act *on behalf of that user*, seeing only what that user can see. Microsoft's own description: delegated permissions "Get access on behalf of a user", and "Users can consent for their data" ([7]).
- **Application permissions** are granted to the app itself and let it run "without a signed-in user". For these, "**Only admin can consent**" ([7]).

Crucially, delegated does **not** automatically mean "no admin consent". Microsoft Graph annotates each permission with an `AdminConsentRequired` flag, and some delegated permissions are flagged `Yes` ([8]). For example, `ChannelMessage.Read.All` is admin-consent-required for *both* application and delegated use, while `Chat.Read` is delegated-only and admin-consent-required `No` ([8]). The participant-only boundary therefore cannot be drawn from the word "delegated" alone; it has to be drawn per permission.

The same shape appears outside Microsoft. Google Workspace has no per-permission admin-consent flag, but a Workspace administrator can still block or restrict an app wholesale through API controls, independently of the user's OAuth grant ([25]). Slack has an equivalent "Approve apps" setting that puts an administrator between a user and an app install ([34]). So "no admin consent required" should be read as "no admin consent is required *by the platform's permission model*", with tenant policy able to override it in all three ecosystems.

---

## 2. Comparison matrix

Legend for the last column: **Yes** = reachable with a user's own delegated consent and no administrator grant; **Policy-dependent** = no admin grant required by the platform, but a tenant/workspace administrator can still block it; **No** = requires an administrator grant, an enterprise licence, or an application-level credential.

| Source | Provider | Capture path (API or export) | Admin consent required? | Typical MY deployment | Inside participant-only boundary? |
|---|---|---|---|---|---|
| Chat | WhatsApp (consumer app) | No official API. On-device per-chat **Export chat** to `.txt`; media only as attachments in the sharesheet, and the file cannot be re-imported ([1]) | Not applicable (no admin) | Dominant consumer messenger in MY; widely used for informal work chat ([39]) | Yes, but manual and out-of-band — **gap** |
| Chat | WhatsApp Business Platform (Cloud API) | Send-only: `POST /{Phone-Number-ID}/messages`; inbound messages arrive only via webhooks ([2], [5]). History only via the `history` webhook under Coexistence ([3], [4]) | Yes — business number; requires Solution/Tech Provider status and customer consent ([4]) | SME customer-service messaging | No (business number, not the personal account) |
| Chat | Microsoft Teams — 1:1 and group chats | Graph `GET /me/chats/{id}/messages` with delegated `Chat.Read` ([9], [10]) | **No** (delegated; admin consent flag `No`) ([8]) | M365 tenants in enterprise and government | **Yes** |
| Chat | Microsoft Teams — channel messages | Graph `GET /teams/{id}/channels/{id}/messages`, `ChannelMessage.Read.All` ([11]) | **Yes** — `ChannelMessage.Read.All` is `AdminConsentRequired = Yes` for delegated *and* application ([8]) | M365 tenants | **No** |
| Chat | Microsoft Teams — bulk export | Teams Export APIs; `Chat.Read.All`, `ChannelMessage.Read.All`, `User.Read.All` (application) ([14]) | **Yes** — protected APIs; "Only an administrator can approve application permissions" ([7], [14]) | Compliance / eDiscovery | **No** |
| Chat | Microsoft Teams — chat delta | `GET /me/chats/{id}/messages/delta` — **application-only**, `Chat.Read.All` ([8], [48]) | **Yes** | Compliance | **No** |
| Chat | Google Chat | Chat API with user credentials; `chat.messages.readonly`, `chat.spaces.readonly` ([20]) | No admin consent in the OAuth model; Workspace admin can block via API controls ([25]) | Google Workspace tenants | **Policy-dependent** |
| Chat | Slack | `conversations.history` with a **user token** and `channels:history` / `groups:history` / `im:history` / `mpim:history` ([32]) | No per-app consent by default; admin can enable "Approve apps" ([34]) | Slack workspaces | **Policy-dependent** |
| Chat | Telegram | Official Desktop **Export Telegram data** to JSON or HTML, per-chat or full account ([35]); MTProto user API for live access ([36]) | No | Consumer; used by some tech teams | **Yes** (export); MTProto works but is a user-client API |
| Chat | WeChat / WeCom | Consumer WeChat: no documented API. WeCom: **会话内容存档** (session content archive) and **获取会话记录** (fetch conversation records), enterprise-scoped ([37], [38]) | **Yes** — requires the enterprise admin console to install a public key, and archiving is a paid enterprise service ([38]) | Chinese-MY firms, cross-border operations | **No** |
| Meeting | Microsoft Teams — recordings / transcripts | Files land in the organiser's OneDrive (private meetings) or the channel's SharePoint site (channel meetings) ([17]); Graph `onlineMeetings` / `callTranscripts` for the API view ([12], [13]) | Transcript/recording **APIs**: `OnlineMeetingTranscript.Read.All` and `OnlineMeetingRecording.Read.All` are `AdminConsentRequired = Yes` even delegated ([8]); application permissions additionally need an application access policy ([12], [15]). **Files**: `Files.Read`, `Files.Read.All`, `Sites.Read.All` delegated are `No` ([8]) | M365 tenants | **Yes via the user's own files**; **No** via the dedicated transcript API |
| Meeting | Google Meet | Meet REST API `conferenceRecords`, `participants`, `recordings`, `transcripts`, `transcripts.entries`, `smartNotes`; scopes `meetings.space.created` / `meetings.space.readonly` ([21], [23]). Artifacts are an MP4 and a Google Doc in the **organiser's Drive** ([22]) | No admin consent in the OAuth model; Workspace admin can block via API controls ([25]) | Google Workspace tenants | **Yes** for conference records, participants and transcript *entries*; **file download depends on Drive access to the organiser's file** ([22]) |
| Meeting | Zoom | Cloud recording plus audio transcript (VTT); **user-managed OAuth** app with `cloud_recording:read:list_recording_files`, `cloud_recording:read:list_user_recordings`, `cloud_recording:read:meeting_transcript` ([28], [30]). Server-to-Server OAuth is account-level and admin-authorised ([29]) | No for user-managed OAuth; **Yes** for S2S ([29], [30]) | Zoom is widely used for external and cross-company meetings in MY | **Yes** for the user's own recordings; a recording owned by another host is likely outside the boundary (see §6) |
| Meeting | Third-party transcription (Otter.ai, Sonix, Notta, Whisper-based tools) | Upload the recording, or a bot joins the meeting; each vendor's own API | No | Varies | **Yes** (user-initiated), subject to language coverage ([42], [43], [46]) |
| Email | Microsoft 365 / Exchange Online | Graph `GET /me/messages` with delegated `Mail.Read` or `Mail.ReadBasic` ([16]) | **No** (`Mail.Read` delegated flag is `No`) ([8]) | M365 is the mainstream enterprise and government mail platform in MY ([40]) | **Yes** |
| Email | Microsoft 365 — other people's mailboxes | `Mail.Read.All` / `Mail.ReadBasic.All` (application) ([8]) | **Yes** | IT / compliance | **No** |
| Email | Google Workspace (Gmail) | Gmail API `users.messages.*` with `gmail.readonly`, a **restricted** scope ([18], [19]) | No admin consent for the user's own mailbox; admin can block the app ([25]) | Google Workspace is the main alternative to M365 | **Policy-dependent** |
| Email | Gmail — other people's mailboxes | Domain-wide delegation: a service account impersonates users ([27]) | **Yes** | Admin / IT | **No** |

---

## 3. Chat: what the evidence actually says

### 3.1 WhatsApp

The consumer app is the single largest informal work-chat surface in Malaysia, but it is the least automatable. WhatsApp's own help centre documents exactly one user-facing export mechanism: open the chat, choose **More options → More → Export chat**, then **Without media** or **Include media**, and share the resulting file ([1]). The page is explicit that this is a dead end for re-import — "Your chat history can't be re-imported because it's a text file and not a backup file" — and that with media only "the most recent media sent will be added as attachments" ([1]). In other words: per-chat, text-only, lossy on media, and no API.

The seed note claimed the WhatsApp Business Cloud API is "the business's own number, no history migration". That is **half right and worth correcting**. Meta's documentation index for the WhatsApp Business Platform contains a `history` webhook whose stated purpose is "to synchronize the WhatsApp Business app chat history of a business customer onboarded by a solution provider" ([3]). The Coexistence onboarding guide explains the mechanics: a business customer connects their existing **WhatsApp Business app** number to Cloud API, and after they "give the business the option to share their chat history", history is synchronised ([4]). The limits are material and are stated in the same documents:

- Coverage is "all messages sent or received within **180 days** of the time when the business was onboarded onto Cloud API", delivered in three phases (day 0–1, 1–90, 90–180) ([3]).
- "Messages that are part of a **group chat will not be included**" ([3]); the feature comparison table confirms group chats are "Not supported … Group chats will not be synchronized" ([4]).
- Media asset IDs are only sent "for media messages sent within **14 days** of onboarding" ([3]).
- The partner "must already be a **Solution Partner or Tech Provider**" ([4]).

So there is a history-sync path, but it is a partner-programme capability tied to a business phone number, not something an individual can invoke for their personal account. For a participant-only personal archive, personal WhatsApp remains an export-only, manual gap.

The Cloud API itself is unambiguously send-oriented. The overview describes the platform as enabling "businesses to communicate with customers at scale" and says that "the contents of any message sent from a WhatsApp user to your business phone number is communicated via webhook" ([2]). The Message API reference exposes a single operation, `POST /{Version}/{Phone-Number-ID}/messages` ([5]). There is no read or list endpoint for message history.

*Retrieval note: Meta's developer documentation returns an error page to ordinary clients. The pages cited here were retrieved with a search-engine user agent, and the `.md` endpoints were discovered through Meta's own machine-readable index at `https://developers.facebook.com/llms.txt` ([6], [49]). The consumer app's absence of an API is established negatively — the entire WhatsApp Business Platform documentation index ([6]) contains no consumer-account API — rather than by a single sentence stating "there is no API".*

### 3.2 Microsoft Teams

Teams is where the participant-only boundary is sharpest, and where the seed note was directionally right but imprecise.

Reading the user's own 1:1 and group chats is squarely inside the boundary. The "chat: list messages" operation lists `Chat.Read` as the least-privileged delegated permission, with `Chat.ReadWrite` above it, and application permissions `ChatMessage.Read.Chat`, `Chat.Read.All` and `Chat.ReadWrite.All` ([10]). The permissions reference records `Chat.Read` as delegated-only with `AdminConsentRequired = No` ([8]). This is the cleanest participant-only chat connector in the survey.

Channel messages are not. The "channel: list messages" operation lists `ChannelMessage.Read.All` as the least-privileged **delegated** permission ([11]), but the permissions reference marks `ChannelMessage.Read.All` as `AdminConsentRequired = Yes` for both application and delegated ([8]). The seed note said "ChannelMessage.Read.All need tenant admin consent" in the context of the export APIs; the sharper statement is that **even the delegated channel-read permission needs admin consent**, so channel messages fall outside the boundary regardless of which API you use.

Bulk export is explicitly administrative. Microsoft's Export APIs page states that "Microsoft Teams APIs in Microsoft Graph that access sensitive data are considered **protected APIs**", that "Application permissions are used by apps that run without a signed-in user present", and that "**Only an administrator can approve** application permissions", listing `Chat.Read.All`, `ChannelMessage.Read.All`, `User.Read.All`, `OnlineMeetingTranscript.Read.All` and `OnlineMeetingRecording.Read.All` ([14]). This is consistent with the general Graph rule that "Only admin can consent" for application permissions ([7]).

The chat delta endpoint is application-only: the `chatmessage-delta` permissions table shows "Not supported" for both delegated columns and `Chat.Read.All` under Application ([8], [48]). That matters because delta is the usual way to do incremental, complete backfill — and it is unavailable to a participant-only app. A participant-only Teams chat connector must therefore paginate `/me/chats` and `/me/chats/{id}/messages` rather than rely on delta.

### 3.3 Google Chat, Slack, Telegram, WeChat

**Google Chat** supports user-credential access. Google's documentation states that authenticating "with user credentials lets Chat apps access user data and perform [actions] on behalf of the user", and the Chat API scope list includes `chat.messages.readonly` and `chat.spaces.readonly` alongside the administrator-only `chat.admin.*` scopes ([20]). There is no admin-consent flag in the Google model, but Workspace administrators can classify apps as Trusted, Limited, Specific Google data, or Blocked in the Admin console ([25]), so this is policy-dependent rather than unconditionally available.

**Slack** exposes the user's own conversations. `conversations.history` documents both bot-token and **user-token** scopes — `channels:history`, `groups:history`, `im:history`, `mpim:history` — and notes that "Only user tokens can access public channels they are not in" ([32]). A user token is obtained through the v2 OAuth flow with a `user_scope` parameter ([33]). The counterweight is administrative: when an admin enables the "Approve apps" setting, "apps must then be requested by a Slack user and approved by an admin before they're actually installed for a team to use", and Enterprise Grid admins can delegate that approval work to an app holding `admin.apps:read` and `admin.apps:write` ([34]).

**Telegram** is the most export-friendly of the consumer messengers. Telegram's own announcement describes a Desktop tool that exports "some (or all) of your chats, including photos and other media they contain", producing "all your data accessible offline in JSON-format or in beautifully formatted HTML", reachable via **Settings → Export Telegram data** or per-chat **Export chat history** ([35]). For live programmatic access, the MTProto method `messages.getHistory` is annotated "Only users can use this method", and Telegram documents the need to register a user's phone to use the API ([36]) — i.e. a user-account API rather than a business API.

**WeChat and WeCom** sit outside the boundary. Personal WeChat has no documented API for a user's own message history; the enterprise product does have one, but it is enterprise-scoped and administrator-driven. WeCom's "获取会话记录" (fetch conversation records) endpoint is described as retrieving "会话记录" for an enterprise over a time window, and the documentation instructs the application to install a public key through the enterprise admin console ("请在【企业微信管理后台->安全与管理->管理工具->数据与智能专区->会话内容】设置公钥") before archiving happens at all ([38]); the archive feature itself is the enterprise "会话内容存档" service ([37]). This is an employer archive capability, not a participant capability.

---

## 4. Meeting capture

### 4.1 Microsoft Teams

Teams meeting capture has two distinct paths, and they have opposite consent profiles — this is the most useful finding in the survey.

The **API path** is admin-gated. `onlineMeeting: list transcripts` requires `OnlineMeetingTranscript.Read.All` as the least-privileged delegated permission, and the permissions reference marks that permission `AdminConsentRequired = Yes` for both application and delegated ([8], [13]). The same is true of `OnlineMeetingRecording.Read.All` ([8]). For application permissions there is a further hurdle: "tenant administrators must create an application access policy and grant it to a user" ([12], [15]). On top of that, the transcript documentation describes two independent tenant administrator settings, one of which — "Graph API access to transcripts" — causes all transcript requests to return `403 Forbidden` with a `GraphAccessToTranscriptsDisabled` inner error when disabled ([12]).

The **file path** is not admin-gated. Microsoft's cloud recording documentation states that the recording "gets uploaded to the meeting organizer's OneDrive (private meetings) or SharePoint (channel meetings)", and that "OneDrive file storage, and access permissions apply to the meeting recording files the same as with other files" ([17]). Since delegated `Files.Read`, `Files.Read.All` and `Sites.Read.All` are all `AdminConsentRequired = No` ([8]), a user who can already open the file in Teams can also read it through Graph with their own consent. For a participant-only archive this is the practical Teams meeting connector: find the transcript/recording file in the user's own OneDrive (organiser case) or a SharePoint site they belong to (channel meeting case), and read it with delegated file permissions. The limitation is obvious and should be stated: if the user was only an attendee and the file lives in someone else's OneDrive, this path closes.

### 4.2 Google Meet

The Meet REST API is unusually well aligned with participant-only capture. The overview lists exactly the capabilities needed: "Get conference records, attendance details, and participant session metadata" and "Retrieve meeting artifacts (recordings, transcripts, transcript entries, and smart notes)" ([21]). The artifacts guide is explicit about who may call it: "**If you're a meeting space owner or participant**, you can call the get and list methods on the recordings, transcripts, transcript[.entries]" ([22]). Scopes are `meetings.space.created` and `meetings.space.readonly` ([23]).

Two caveats are documented in the same source. First, artifacts are saved "to the meeting organizer's Google Drive", with recordings exposed as an MP4 via `exportUri` and transcripts as a Google Doc, and the guide points at the Drive API for the actual download ([22]) — so obtaining the *file* is a Drive-access question, not a Meet-permission question, and for a non-organiser this is the same "someone else's Drive" problem as Teams. Second, "Transcript entries provided by the Meet REST API are deleted **30 days** after the conference ends" ([22]), which is a hard retention ceiling for the structured transcript data.

For real-time capture, the Workspace Events API delivers Meet events through Pub/Sub, including `google.workspace.meet.conference.v2.started`, `google.workspace.meet.conference.v2.ended`, `google.workspace.meet.participant.v2.joined`, `google.workspace.meet.recording.v2.started`, `google.workspace.meet.recording.v2.ended` and the transcript file-generated event ([24]).

### 4.3 Zoom

Zoom's user-managed OAuth app type is the participant-only option, and Zoom says so directly: "**User-managed**: Individual users add and manage the app. The app has access to only the user's authorized data", as distinct from "**Admin-managed**: Account admins add and manage the app" ([30]). Server-to-Server OAuth is explicitly the account-level alternative, described as using "account credentials" with a two-step flow that "does not require user interaction", and its documentation notes that "Account administrators authorize the scopes available to developers building these app types" ([29]).

The scope names carry the same distinction. Zoom's published Meeting API specification lists, for `GET /meetings/{meetingId}/recordings`, the scopes `cloud_recording:read:list_recording_files`, `cloud_recording:read:list_recording_files:admin` and `cloud_recording:read:list_recording_files:master` — the un-suffixed form being the user-level grant ([28]). The user-level list endpoint uses `cloud_recording:read:list_user_recordings`, and meeting transcripts use `cloud_recording:read:meeting_transcript` ([28]). The seed note's shorthand `cloud_recording:read` is a legacy scope name; the current granular names are as above.

One unresolved limitation: these endpoints are keyed on `meetingId` and `userId`, and the participant-only grant is over "the user's authorized data" ([30]). A cloud recording belongs to the account of the user who recorded it, so a user who merely attended someone else's meeting is unlikely to reach that recording. I could not find a primary sentence that states this explicitly; it is flagged in §6 rather than asserted.

### 4.4 Transcription language coverage — the Malay/English code-switching weak spot

The seed note's concern is borne out for the *native* transcription engines, with specific numbers:

- **Google Meet** transcripts are "available in: English, French, German, Italian, Japanese, Korean, Portuguese, Spanish" — eight languages, **Malay not among them** ([26]).
- **Microsoft** multilingual speech recognition supports "English, Spanish, French, German, Italian, Portuguese, Chinese, Japanese, and Korean" — nine languages, **Malay not among them** ([44]).
- **Otter.ai**'s help centre states it "currently only supports English, Spanish, French, German, Japanese, and Chinese (Simplified)" — **Malay not among them** ([43]).
- **Sonix** does list Malay ("Transcribe Malay audio and video") among its supported languages ([42]).
- **Zoom** advertises AI captions/translation "in 46 languages", but this is a *captioning* claim on a marketing page, not a statement about the language of the stored audio transcript file ([31]). I could not retrieve Zoom's supported-languages knowledge-base article, which is JavaScript-rendered; treat Zoom's Malay transcript quality as unverified.

The underlying research problem is real and has been studied in Malaysia specifically: Mustafa, Yek and Mat Kiah's "A Malay-English Code-Switching Bilingual ASR System" (Universiti Malaya, *Telematique* 22(1), 2023) treats Malay-English code-switching as a distinct ASR problem requiring a bilingual system ([45]). The practical consequence for this project is that even where a vendor *does* support Malay, Malaysian workplace speech — which routinely mixes Malay and English within a sentence — is the harder case, and no vendor's marketing page makes a code-switching accuracy claim.

---

## 5. Email, and the Malaysian deployment picture

### 5.1 Email

Both mainstream mail platforms offer a clean participant-only path for the user's own mailbox.

On the Microsoft side, `message: get` lists `Mail.ReadBasic` and `Mail.Read` for delegated use, and `Mail.ReadBasic.All` / `Mail.Read` for application use ([16]); the permissions reference records `Mail.Read` delegated as `AdminConsentRequired = No` and `Mail.ReadBasic.All` as application-only ([8]). Reaching anyone else's mailbox means `Mail.Read.All` as an application permission, which is admin-consent-required ([8]).

On the Google side, the Gmail API is "the best choice for authorized access to a user's Gmail data" and is listed for "read-only mail extraction, indexing, and backup" ([19]). `https://www.googleapis.com/auth/gmail.readonly` ("View your email messages and settings") is classified by Google as a **restricted scope** ([18]), which is a compliance burden (verification/assessment for public distribution) but not an administrator-consent requirement. Reading *other* users' mailboxes requires domain-wide delegation, where a service account is authorised "to use domain-wide delegation" to impersonate users ([27]) — an administrator action.

### 5.2 What Malaysian deployment actually looks like

This is the weakest evidence area in the survey, and I want to be explicit about it rather than paper over it.

**Consumer messaging.** DataReportal's Digital 2025: Malaysia report gives 34.9 million internet users (97.7% penetration), 25.1 million social media user identities (70.2% of population) and 43.3 million cellular connections in January 2025 ([39]). **It does not publish a WhatsApp user figure for Malaysia at all** — the per-platform sections cover YouTube (25.1M) and Facebook (23.1M), and the report's platform slides are images, not text ([39]). The seed note's "WhatsApp ~26M users, DataReportal/Statista #1" could **not** be verified against this primary source, and should be treated as unverified. I also could not retrieve the Malaysian Communications and Multimedia Commission's Internet Users Survey, whose landing page renders its content via JavaScript and returned only navigation text ([47]). No primary source retrieved in this pass establishes WhatsApp's Malaysian user count or its rank.

**Enterprise and government suites.** The evidence I could retrieve is vendor-primary and directional rather than quantitative. Microsoft's Malaysia news centre documents that, under the "Bersama Malaysia" pledge, Microsoft "partnered with the Malaysian Administrative Modernisation and Management Planning Unit (MAMPU) to support their target of taking our nation's government to the cloud", signing a Cloud Framework Agreement with Enfrasys Solutions "to usher in trusted cloud services to public sector agencies" ([40]). Google's Malaysian blog announced a "US$2 billion investment in the country" including its "first data center and Google Cloud region" at Elmina Business Park, framing it as support for "the nation's 'Cloud First Policy'", and noting that the data center "will power our popular services like Search, Maps, and **Workspace**" ([41]). Both vendors are clearly embedded in the Malaysian public sector; **neither source establishes a relative installed-base share between Microsoft 365 and Google Workspace**, and I found no primary Malaysian government or statistical source that does. The seed note's "M365/Exchange dominant in MY enterprise/govt; Google Workspace main alternative" is therefore a reasonable working assumption but should be labelled as such, not as a sourced finding.

**Zoom.** I found no primary Malaysian source quantifying Zoom adoption; the seed note's "Zoom is commonly used in Malaysia" is likewise an unverified working assumption.

---

## 6. Connector shortlist inside the participant-only boundary

The following connectors stay inside the boundary defined in §1 — each is reachable with the individual user's own delegated grant, with no tenant-wide administrator consent *required by the platform*. Where a tenant or workspace administrator can nonetheless block the app, that is noted, because in a real Malaysian enterprise deployment that is the binding constraint.

**Recommended (delegated per-user OAuth, no admin consent required by the platform):**

1. **Microsoft Teams — 1:1 and group chats.** Graph `/me/chats` and `/me/chats/{id}/messages`, delegated `Chat.Read` ([8], [10]). Paginate; delta is application-only ([8], [48]).
2. **Microsoft 365 mail.** Graph `/me/messages`, delegated `Mail.ReadBasic` or `Mail.Read` ([8], [16]).
3. **Microsoft Teams — meeting artefacts, via files.** Read the transcript/recording file from the user's own OneDrive or a SharePoint site they belong to, with delegated `Files.Read` / `Files.Read.All` / `Sites.Read.All` ([8], [17]). This deliberately avoids the admin-gated transcript API.
4. **Gmail.** Gmail API `users.messages.*` with `gmail.readonly` ([18], [19]). Restricted-scope verification applies.
5. **Zoom.** User-managed OAuth app with `cloud_recording:read:list_recording_files`, `cloud_recording:read:list_user_recordings`, `cloud_recording:read:meeting_transcript` ([28], [30]).
6. **Google Meet.** `meetings.space.readonly` for conference records, participants, transcript entries and smart notes; Workspace Events subscriptions for real-time lifecycle ([21], [22], [23], [24]).

**Conditionally inside (no platform admin consent, but an administrator can block the app):**

7. **Google Chat.** `chat.messages.readonly` and `chat.spaces.readonly` with user credentials ([20]); subject to Admin-console API controls ([25]).
8. **Slack.** `conversations.history` with a user token and the `:history` scopes ([32], [33]); subject to the workspace "Approve apps" setting ([34]).

**Manual / out-of-band, but inside the boundary:**

9. **Telegram.** Desktop "Export Telegram data" to JSON/HTML, imported as a file ([35]).
10. **Third-party transcription.** Sonix for Malay, or a self-hosted Whisper-class model, applied to recordings the user already holds ([42], [45]).

**Explicitly outside the boundary — do not design around these:**

- Teams **channel** messages (`ChannelMessage.Read.All`, admin consent required even delegated) ([8], [11]).
- Teams **transcripts/recordings via the dedicated Graph APIs** (`OnlineMeetingTranscript.Read.All`, `OnlineMeetingRecording.Read.All`, admin consent required) ([8], [12], [13]).
- **Teams Export APIs** (protected APIs, application permissions, admin-only approval) ([14]).
- Teams chat **delta** (application-only) ([8], [48]).
- **Gmail domain-wide delegation** and **`Mail.Read.All`** ([8], [27]).
- **WeCom session-content archiving** (enterprise admin console, paid archive service) ([37], [38]).
- **Zoom Server-to-Server OAuth** (account-level, admin-authorised scopes) ([29]).

**The gap that must be flagged: personal WhatsApp.** There is no participant-only automated connector for the consumer WhatsApp app. The only mechanism Meta documents for a user's own history is the on-device per-chat `.txt` export, which is manual, per-chat, and strips media to sharesheet attachments that "can't be re-imported" ([1]). The Business Platform's `history` webhook is not a substitute: it requires Solution/Tech Provider status, a WhatsApp **Business app** number, customer consent, a 180-day window, and it excludes group chats ([3], [4]). For a Malaysia-focused personal work archive, **personal WhatsApp is the single largest uncovered surface**, and any design should treat it as a manual-import feature rather than a connector.

---

## 7. Open questions / residual risk

1. **Personal WhatsApp may simply be unfixable.** I could not find any primary source describing an official, user-authorised path to a consumer WhatsApp account's history. Before closing issue #2, it is worth deciding explicitly whether the archive accepts a manual `.txt` import as the WhatsApp story, or whether WhatsApp is declared out of scope. (Evidence: [1], [3], [4], [6].)

2. **Tenant policy can veto every "no admin consent" claim.** Microsoft notes that even where admin consent is not required by default, "your organization's policies may still restrict user consent" ([7]); Google gives Workspace admins Trusted/Limited/Blocked classification of apps ([25]); Slack gives admins an "Approve apps" gate ([34]). The matrix's "No" column describes the platform default, not the user's actual workplace. A separate check of the user's own tenant settings is needed, and that belongs with the company-policy work in issue #7.

3. **The Teams file path needs empirical validation.** The claim that a user can read their own meeting transcript through delegated file permissions rests on two separately-sourced facts — recordings/transcripts land in OneDrive/SharePoint ([17]) and delegated file permissions do not require admin consent ([8]) — rather than on a single document describing that workflow. It should be tested against a real tenant before it becomes the design.

4. **Zoom recording ownership for non-hosts is unverified.** Whether a user-level OAuth grant can reach a cloud recording owned by another user's account could not be established from a primary source. If it cannot, Zoom coverage shrinks to meetings the user personally recorded. (Evidence: [28], [30].)

5. **Microsoft 365 vs Google Workspace share in Malaysia is not established.** The vendor-primary sources confirm both vendors' public-sector presence ([40], [41]) but give no installed-base split. Any prioritisation between the two connector families is a judgement call, not a sourced conclusion.

6. **Malaysian usage statistics remain unverified.** DataReportal 2025 Malaysia contains no WhatsApp figure ([39]) and the MCMC Internet Users Survey could not be retrieved ([47]). The seed note's WhatsApp user count and "number one platform" ranking are unsupported in this pass and should not be carried forward as fact. If a defensible number is needed, the MCMC Internet Users Survey and DOSM's ICT use survey are the primary sources to pursue.

7. **Malay/English code-switching accuracy is unquantified.** Native transcription excludes Malay entirely in Google Meet, Microsoft, and Otter ([26], [43], [44]); Sonix supports it ([42]); Zoom's position is unverified ([31]). None of these sources states an accuracy figure for code-switched Malay-English speech, and the Malaysian ASR literature treats it as a hard problem in its own right ([45]). Transcription quality on real Malaysian meetings is therefore an open, measurable risk, not a settled one.

8. **Retention ceilings will shape the architecture.** Google Meet transcript entries expire 30 days after the conference ([22]); WhatsApp's business history sync covers 180 days ([3]); Teams export reaches deleted messages for only 21 days and deleted users/teams for 30 ([14]). Any archive that is meant to be a long-term record must capture eagerly rather than backfill on demand.

9. **Slack and Google Chat consent mechanics differ from Microsoft's.** Neither platform has a per-permission admin-consent flag, so the "inside the boundary" verdict rests on the OAuth model plus the fact that an administrator *can* intervene ([25], [34]) rather than on an explicit "no admin consent required" statement. This asymmetry should be reflected in how confidently the shortlist is presented.

---

## Sources

1. WhatsApp Help Center, "How to export your chat history" — https://faq.whatsapp.com/1180414079177245 (retrieved 2026-09-25)
2. Meta, "About the WhatsApp Business Platform" (Cloud API overview) — https://developers.facebook.com/docs/whatsapp/cloud-api/overview (retrieved 2026-09-25 via search-engine user agent)
3. Meta, "history webhook reference" — https://developers.facebook.com/documentation/business-messaging/whatsapp/webhooks/reference/history.md (retrieved 2026-09-25)
4. Meta, "Onboard WhatsApp Business app users" (Coexistence / Embedded Signup) — https://developers.facebook.com/documentation/business-messaging/whatsapp/embedded-signup/onboarding-business-app-users.md (retrieved 2026-09-25)
5. Meta, WhatsApp Cloud API Message API reference — https://developers.facebook.com/docs/whatsapp/cloud-api/reference/messages (retrieved 2026-09-25)
6. Meta, WhatsApp Business Cloud API documentation index — https://developers.facebook.com/documentation/business-messaging/whatsapp/llms.txt (retrieved 2026-09-25)
7. Microsoft Learn, "Permissions and consent in the Microsoft identity platform / Microsoft Graph permissions overview" — https://learn.microsoft.com/en-us/graph/permissions-overview (retrieved 2026-09-25)
8. Microsoft Learn, "Microsoft Graph permissions reference" — https://learn.microsoft.com/en-us/graph/permissions-reference (retrieved 2026-09-25; permission-level `AdminConsentRequired` flags for `Chat.Read`, `Chat.ReadBasic`, `Chat.ReadWrite`, `Chat.Read.All`, `Chat.ReadWrite.All`, `ChannelMessage.Read.All`, `OnlineMeetingTranscript.Read.All`, `OnlineMeetingRecording.Read.All`, `CallRecords.Read.All`, `CallTranscripts.Read.All`, `OnlineMeetings.Read`, `Mail.Read`, `Mail.ReadBasic`, `Mail.ReadBasic.All`, `Files.Read`, `Files.Read.All`, `Sites.Read.All`, `User.Read.All`)
9. Microsoft Learn, "Get chatMessage" (permission tables for channel and chat) — https://learn.microsoft.com/en-us/graph/api/chatmessage-get (retrieved 2026-09-25)
10. Microsoft Learn, "chat: list messages" and its generated permissions table (`Chat.Read`) — https://learn.microsoft.com/en-us/graph/api/chat-list-messages (retrieved 2026-09-25)
11. Microsoft Learn, "channel: list messages" and its generated permissions table (`ChannelMessage.Read.All`) — https://learn.microsoft.com/en-us/graph/api/channel-list-messages (retrieved 2026-09-25)
12. Microsoft Learn, "Get callTranscript" (including tenant administrator controls for transcript access and `GraphAccessToTranscriptsDisabled`) — https://learn.microsoft.com/en-us/graph/api/calltranscript-get (retrieved 2026-09-25)
13. Microsoft Learn, "onlineMeeting: list transcripts" — https://learn.microsoft.com/en-us/graph/api/onlinemeeting-list-transcripts (retrieved 2026-09-25)
14. Microsoft Learn, "Export content with the Microsoft Teams Export APIs" — https://learn.microsoft.com/en-us/microsoftteams/export-teams-content (retrieved 2026-09-25)
15. Microsoft Learn, "Configure application access to online meetings or virtual events" — https://learn.microsoft.com/en-us/graph/cloud-communication-online-meeting-application-access-policy (retrieved 2026-09-25)
16. Microsoft Learn, "message: get" (permission table for `Mail.Read` / `Mail.ReadBasic`) — https://learn.microsoft.com/en-us/graph/api/message-get (retrieved 2026-09-25)
17. Microsoft Learn, "Manage Teams cloud meeting recording" (recording/transcript storage in OneDrive and SharePoint) — https://learn.microsoft.com/en-us/microsoftteams/cloud-recording (retrieved 2026-09-25)
18. Google, "Choose Gmail API scopes" (restricted scope classification of `gmail.readonly`) — https://developers.google.com/workspace/gmail/api/auth/scopes (retrieved 2026-09-25)
19. Google, "Gmail API overview" — https://developers.google.com/workspace/gmail/api/guides (retrieved 2026-09-25)
20. Google, "Authenticate and authorize as a Google Chat user" and "Authenticate and authorize with the Chat API" (scope list including `chat.messages.readonly`, `chat.spaces.readonly`, `chat.admin.*`) — https://developers.google.com/workspace/chat/authenticate-authorize-chat-user and https://developers.google.com/workspace/chat/api/guides/auth (retrieved 2026-09-25)
21. Google, "Google Meet REST API overview" — https://developers.google.com/workspace/meet/api/guides/overview (retrieved 2026-09-25)
22. Google, "Work with artifacts" (Meet recordings, transcripts, smart notes; organiser's Drive; 30-day transcript-entry retention; participant access) — https://developers.google.com/workspace/meet/api/guides/artifacts (retrieved 2026-09-25)
23. Google, "conferenceRecords.list" (authorization scopes `meetings.space.created`, `meetings.space.readonly`) — https://developers.google.com/workspace/meet/api/reference/rest/v2/conferenceRecords/list (retrieved 2026-09-25)
24. Google, "Subscribe to events using the Google Workspace Events API" and "Subscribe to Google Meet events" — https://developers.google.com/workspace/events and https://developers.google.com/workspace/events/guides/events-meet (retrieved 2026-09-25)
25. Google Workspace Admin Help, "Control which third-party & internal apps access Google Workspace data" (Trusted / Limited / Specific Google data / Blocked) — https://support.google.com/a/answer/7281227 (retrieved 2026-09-25)
26. Google Meet Help, meeting transcripts supported languages — https://support.google.com/meet/answer/12849897 (retrieved 2026-09-25)
27. Google, "Create access credentials" (service accounts and domain-wide delegation) — https://developers.google.com/workspace/guides/create-credentials (retrieved 2026-09-25)
28. Zoom, Zoom Meeting API OpenAPI specification (scopes for `GET /meetings/{meetingId}/recordings`, `GET /users/{userId}/recordings`, meeting transcripts) — https://developers.zoom.us/api-specs/zoom-api/methods/ZoomMeetingAPI-spec.json (retrieved 2026-09-25)
29. Zoom Developer Docs, "Server-to-Server OAuth" (account credentials, no user interaction, administrators authorise scopes) — https://developers.zoom.us/docs/internal-apps/s2s-oauth/ (retrieved 2026-09-25)
30. Zoom Developer Docs, "Create an OAuth app" (User-managed vs Admin-managed/account-level app) — https://developers.zoom.us/docs/integrations/create/ (retrieved 2026-09-25)
31. Zoom, AI Companion accessibility page ("translate and transcribe in 46 languages") — https://www.zoom.com/en/products/ai-assistant/features/accessibility/ (retrieved 2026-09-25; marketing claim about captions, not about stored transcript language)
32. Slack, "conversations.history" (bot and user token scopes) — https://api.slack.com/methods/conversations.history (retrieved 2026-09-25)
33. Slack, "Installing with OAuth" (`user_scope` parameter, user access tokens) — https://api.slack.com/authentication/oauth-v2 (retrieved 2026-09-25)
34. Slack Developer Docs, "Managing app approvals" (`admin.apps:read`, `admin.apps:write`, "Approve apps" setting) — https://docs.slack.dev/admins/managing-app-approvals/ (retrieved 2026-09-25)
35. Telegram, "Chat Export Tool, Better Notifications and More" (Desktop export to JSON/HTML) — https://telegram.org/blog/export-and-more (retrieved 2026-09-25)
36. Telegram, "messages.getHistory" ("Only users can use this method") — https://core.telegram.org/method/messages.getHistory (retrieved 2026-09-25)
37. WeCom Developer Centre, "会话内容存档" — https://developer.work.weixin.qq.com/document/path/91354 (retrieved 2026-09-25)
38. WeCom Developer Centre, "获取会话记录" (enterprise conversation records; public key configured in the enterprise admin console) — https://developer.work.weixin.qq.com/document/path/99824 (retrieved 2026-09-25)
39. DataReportal, "Digital 2025: Malaysia" — https://datareportal.com/reports/digital-2025-malaysia (retrieved 2026-09-25; note: contains no WhatsApp user figure for Malaysia)
40. Microsoft Malaysia News Center, "Government of Malaysia Builds Resilience by Adopting Government Technology (GovTech) in Partnership with Microsoft" (Bersama Malaysia; MAMPU Cloud Framework Agreement) — https://news.microsoft.com/en-my?p=23065 (retrieved 2026-09-25)
41. Google (Malaysia blog), "Advancing Malaysia Together: Google's US$2 Billion Investment to Power Malaysia's Digital Future" — https://blog.google/intl/ms-my/company-news/technology/2024_05_advancing-malaysia-together/ (retrieved 2026-09-25)
42. Sonix, supported languages (includes Malay) — https://sonix.ai/languages (retrieved 2026-09-25)
43. Otter.ai Help Center, "Supported languages" — https://help.otter.ai/hc/en-us/articles/360047247414-Supported-languages (retrieved 2026-09-25 via search-engine result snippet; the article body did not render for direct retrieval)
44. Microsoft Support, "Multilingual speech recognition in Microsoft Teams" (supported languages) — https://support.microsoft.com/en-us/teams/meetings/multilingual-speech-recognition-in-microsoft-teams (retrieved 2026-09-25)
45. M. B. Mustafa, I. Yek Chung Mei and L. Mat Kiah, "A Malay-English Code-Switching Bilingual ASR System", *Telematique* 22(1), 2023 — https://www.provinciajournal.com/index.php/telematique/article/view/1291 (retrieved 2026-09-25; full text is subscription-only, so only the abstract and metadata were consulted)
46. Notta, supported languages — https://www.notta.ai/en/languages (retrieved 2026-09-25; page body did not render, so Notta's Malay support is **unverified**)
47. Malaysian Communications and Multimedia Commission, "Internet Users Survey" — https://www.mcmc.gov.my/en/resources/statistics/internet-users-survey (retrieved 2026-09-25; page content is JavaScript-rendered and could **not** be retrieved)
48. Microsoft Learn, "chatMessage: delta" permissions table (application-only `Chat.Read.All`) — https://learn.microsoft.com/en-us/graph/api/chatmessage-delta (retrieved 2026-09-25)
49. Meta, machine-readable documentation index — https://developers.facebook.com/llms.txt (retrieved 2026-09-25)
