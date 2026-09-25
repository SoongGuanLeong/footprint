# Company policy checklist

*A practical audit instrument for checking a company handbook, acceptable-use policy and DLP regime against the legal boundary established in [`docs/research/pdpa-2010-boundary.md`](research/pdpa-2010-boundary.md). Deliverable for wayfinder ticket #7.*

> **Not legal advice.** This is a checklist, not counsel. The PDPA memo it derives from is research, and its own open questions are carried forward in §6 below.

---

## How to use this

This is the **instrument**, not a completed audit. It is written to be run against a real handbook, and its output is a list of findings, open questions for IT/HR, and evidence to retain.

**Honest note on scope:** the person this was written for is **not currently employed**, so there is no handbook to audit yet. The ticket's completion criterion is that the checklist exists, which this satisfies; running it is deferred until there is a handbook. The instrument is also what the product surfaces to other installers, who will each have a different employer — which is exactly why it is generic rather than written against one company's policy.

**Order of work:** read the handbook for clauses (§1) → ask IT/HR the questions the handbook does not answer (§2) → retain the evidence (§3) → apply the go/no-go gate (§4).

---

## Three rules of precedence, stated first

These come from the legal memo and they govern everything below. Getting them wrong is the most likely way to misread the rest.

1. **The statute is a floor, not a ceiling.** Nothing prevents an employer from imposing stricter requirements — prohibiting recording entirely, banning personal devices in meeting rooms, requiring DLP classification and endpoint controls, or requiring written authorisation before work content is exported. **Where policy is stricter than the statute, policy governs the employment relationship.**
2. **Breach of policy is a disciplinary risk even where the statute permits the conduct.** In *Izaidin Joinnie*, secretly recording a board meeting was treated as a breach of the **duty of fidelity and good faith**, and the employee's claim failed notwithstanding that the recording was not itself unlawful. **A technically lawful participant-only archive can still be misconduct.**
3. **Policy cannot authorise what a statute forbids, and a permissive policy does not supply consent.** Even a policy expressly permitting recording does not provide other participants' consent under s.6, does not discharge the s.7 notice obligation to them, and does not make sensitive personal data lawful under s.40. **Policy permission and statutory compliance are separate requirements, and both must be met.**

---

## 1. Clauses to look for in the handbook

For each row: find the clause, note the exact wording and its version date, and record whether it helps, hinders, or is silent.

| Clause type | What to look for | Why it matters | If found |
|---|---|---|---|
| **Acceptable use / IT usage** | Whether personal capture of work communications is permitted, prohibited, or unaddressed; whether personal storage of work content is allowed | This is the clause most likely to convert a lawful archive into misconduct (rule 2) | If prohibited, the archive is out of bounds on that device/account. Record the wording verbatim |
| **Data protection / privacy** | The company's PDPA posture: notice given to employees, stated purposes, retention rules, who the data controller is for employee-held data | Establishes whether the employer has already set purposes that constrain you | Note whether employees are given s.7 notice, and in both languages (s.7(3)) |
| **Confidentiality / NDA / IP** | Definitions of confidential information; scope of the duty; duration; carve-outs for personal notes | Client data and internal deliberations in a meeting satisfy the three-element breach-of-confidence test in *Lee Ewe Poh* | Note whether the definition is broad enough to cover ordinary meeting content |
| **Recording consent (meetings and calls)** | Whether meetings may be recorded; whether participants must be informed; whether a recording notice appears on invitations; whether the platform announces recording | This is the **B2 overt-collection** condition, the weakest point of the boundary and the one that cannot be engineered around | If recording requires notice, the product must nudge or enforce it. If recording is banned outright, that governs |
| **DLP / monitoring / audit disclosure** | Whether the employer monitors, logs, or inspects employee devices and communications; whether employees are notified; whether DLP agents exist and what they flag | Determines whether local capture triggers alerts, and whether you were on notice of monitoring | Record whether monitoring is disclosed. Undisclosed monitoring is itself a compliance question for the employer, not for you |
| **BYOD vs company device** | Whether personal devices may hold work data; whether a personal device must be enrolled in MDM; whether the employer can wipe it | Decides which machine may host the archive at all — the Q1/Q8 decision assumed a personal machine with no MDM | If the device is managed, the employer has administrative reach into the store's host. That is a different risk class |
| **Data classification** | The classification scheme (e.g. Public / Internal / Confidential / Restricted) and how content is marked | **The employer's classification is the operative filter.** Content classified confidential or restricted is a policy breach to ingest regardless of the archive's own security | Note the scheme and how to recognise classified content in practice |
| **Social media / external communication** | Restrictions on discussing or publishing work content externally | Publication is prohibited independently of the PDPA (s.8; *Lee Ewe Poh*) | Note any absolute prohibitions |
| **Disciplinary / misconduct** | How policy breaches are characterised; whether breach is expressly misconduct | Ties rule 2 to a concrete consequence | Note whether breach is listed as misconduct |
| **Sectoral rules** | Whether the employer is in a regulated sector (banking, insurance, licensed communications) with its own data-management regime | Sectoral requirements may be **stricter than the PDPA**. The legal memo did not verify any specific sectoral instrument, so this must be checked separately | If regulated, stop and check the sectoral regulator's requirements before proceeding |

**Public-sector users, read this before the table applies:** the PDPA does **not** apply to the Federal or State Governments (s.3(1)). That does **not** mean the data is unprotected — it means a different regime applies. The Official Secrets Act 1972 s.8 makes it an offence to communicate or improperly retain an official secret, and s.2 covers material classified Top Secret, Secret, Confidential or Restricted. **Treat OSA as a hard exclusion boundary, separate from and additional to the PDPA.** The PDPA exclusion for government is a change of regime, not an absence of rules.

---

## 2. Questions to ask IT/HR

The handbook rarely answers these. Ask, and **write down the answers with the date and who gave them** — an undocumented verbal assurance is worth little later.

### Device and endpoint

1. Is this machine managed (MDM/Intune/Jamf)? Do you have administrative access to it?
2. Is a DLP or endpoint agent installed? What does it monitor, and what does it flag?
3. Is the disk encrypted, and who holds the recovery key?
4. May I install software on this machine? Is there an approved-software list?
5. Can you wipe or remotely access this machine?

### Accounts and access

6. May I authorise third-party applications against my own work account (delegated OAuth), or is that blocked at tenant level?
7. Do I have access to my own mailbox and chat history via an API, or only through the official client?
8. Is there a policy on exporting my own work data? Does it need approval?
9. What happens to my accounts and data when I leave?

### Recording

10. May meetings be recorded? Is participant notice required, and how is it given?
11. Does the platform notify participants automatically when recording starts?
12. Is there a retention rule for recordings, and where do they live?
13. Are there meeting types where recording is absolutely prohibited (legal, HR, board)?

### Monitoring and classification

14. Am I notified that my activity may be monitored or audited?
15. How is confidential content marked, and how do I recognise it in practice?
16. Is there a DLP rule that would flag large exports or unusual API access?
17. Who should I ask about whether a specific item is classified?

### Retention and exit

18. How long must I keep work records?
19. What must I delete or return when I leave?
20. May I keep personal notes about my own work after leaving?

---

## 3. Evidence to keep

Retain these so compliance can be **demonstrated**, not merely asserted. The legal memo's H-group items and the PDPA's record-keeping obligation (s.44) both point this way.

| Evidence | Why |
|---|---|
| The handbook / policy version you relied on, with its **version date** | Policy changes; you need the version in force when you acted |
| Written answers from IT/HR, with **date and name** | Verbal assurances are unprovable |
| The **boundary statement** you applied (the B1–B10 conditions from the legal memo) | Shows a defined, documented scope rather than ad-hoc capture |
| **Notice given** to other participants where recording occurred | Directly evidences the B2 overt-collection condition |
| Consent records, where explicit consent was relied on | Required for sensitive personal data (s.40(1)(a)) |
| Your **classification determinations** — what you treated as confidential and why | Shows the employer's classification filter was applied |
| **Deletion log** — what was deleted, when, and why | Evidences retention compliance (s.44) |
| The **breach-notification route** you would follow if the store were compromised | PDPA s.12B: 72 hours to the Commissioner, 7 days to data subjects |

---

## 4. The go/no-go gate

Three boundary conditions are the ones most likely to fail in practice, and they are where a **stop** should be a real stop rather than a judgement call.

| Gate | Condition | Stop if… |
|---|---|---|
| **G1 — Overt collection** | The archive is not covert; participants are informed, or participation in a disclosed, established recording practice | You would be concealing the capture. **This cannot be engineered around** — it is behavioural, and a covert archive is not defensible under current Malaysian law |
| **G2 — No third-party disclosure** | Nothing is shared, forwarded, published, or made accessible to anyone else | Any disclosure is contemplated. The only exceptions are compulsion by law, or a valid access/correction request under ss.30–37 |
| **G3 — Employer policy and classification** | The archive complies with acceptable-use, confidentiality and DLP policy, and no employer-classified content is ingested | The handbook is silent or ambiguous, the device is managed and you have not confirmed, or content is classified confidential/restricted |

**If any gate fails, the archive is not defensible** — regardless of how well the encryption works. These are the conditions the product's own design should nudge or enforce, and they are why the boundary rules sit before the architecture in the spec.

---

## 5. Re-check triggers

Re-run this checklist when any of these changes:

- The PDP Regulations 2013 or the **PDP Standard** are amended. The Commissioner's Public Consultation Paper No. 4/2025 (22 August – 8 September 2025) confirms the security, retention and integrity standards are under revision. **Reference the Standard by name and date; do not hard-code its requirements.**
- The employer's handbook, acceptable-use or DLP policy is revised.
- You change employer, or move between a personal and a managed device.
- You enter or leave a regulated sector.
- You begin recording meetings in a context where you had not before (new meeting type, new participants, new jurisdiction).

---

## 6. Carried forward — known gaps in this instrument

Stated so the checklist is not mistaken for a complete answer:

1. **Sectoral DLP instruments were not verified.** The legal memo retrieved and cited none, so §1's sectoral row says "check separately" and cites nothing. If you are in a regulated sector, that check is genuinely outstanding.
2. **Whether employment is a "commercial transaction"** under s.2 is not authoritatively settled. The memo proceeds on the prudent assumption that the PDPA applies; the checklist inherits that assumption.
3. **Whether a participant recording their own conversation is "interception"** under CMA 1998 s.234 is undecided — no judgment, MCMC guideline or instrument was located.
4. **Whether speaker-identification or voice matching processes "biometric data"** is unresolved. Treated as potential biometric processing until determined otherwise.
5. **Four Industrial Court awards** (*Sanjungan Sekata*, *Yap Fat*, *Justin Maurice Read*, *Izaidin Joinnie*) were not retrieved in primary form; the propositions above come from Skrine's analysis and should be verified before being relied on in a filing.
6. **No consolidated AGC reprint** of Act 709 incorporating the 2024 amendments exists yet, so amended sections are described by reading the original text together with the amending words.

Full detail, statutory citations and source list: [`docs/research/pdpa-2010-boundary.md`](research/pdpa-2010-boundary.md).
