# Malaysia PDPA 2010 and the participant-only Personal Work Footprint archive

**Legal boundary memo and policy checklist**

| | |
|---|---|
| **Ticket** | GitHub issue #3 — "PDPA Malaysia 2010 boundary for participant-only archive" |
| **Feeds** | Issue #6 (spec scope), Issue #7 (company policy checklist) |
| **Date** | 25 September 2026 |
| **Sources retrieved** | 25 September 2026 (all URLs re-checked on that date) |
| **Status** | Research memo. **Not legal advice.** See the notice at the end of this section. |

> **NOT LEGAL ADVICE.** This document is desk research on Malaysian primary legislation, regulator-issued guidance and reported case law. It is written to inform a product scope decision and a company-policy checklist. It is not legal advice, it does not create a lawyer–client relationship, and it should not be relied on as a substitute for advice from a Malaysian advocate and solicitor qualified to practise in the relevant jurisdiction. Statutes are amended frequently; the text below reflects the law as retrieved on 25 September 2026.

---

## 1. Summary: the question and the answer

### 1.1 The question

The Footprint project is building a **participant-only Personal Work Footprint archive**: a single-user, locally-encrypted store that captures work artefacts — chat threads, meeting recordings, emails — where the user is a participant, recipient or attendee. It is intended as a personal recall and evidence tool, explicitly *not* covert surveillance. This memo answers: **where is the legal boundary of a strictly participant-only scope under Malaysian law, and what should the company-policy checklist (issue #7) operationalise?**

### 1.2 The answer in short

**1. The Personal Data Protection Act 2010 (Act 709) — "PDPA 2010" — is the operative statute, and it applies to this archive.** It applies to any person who processes personal data "in respect of commercial transactions" (s.2(1) [S1]). An encrypted, indexed, searchable archive of chats, recordings and emails is "processing" of "personal data" in a "relevant filing system" (s.4 [S1]). Work is a commercial context. The archive is therefore inside the Act, not outside it.

**2. The participant-only scope does *not* remove PDPA exposure, because the archive necessarily contains other people's personal data.** Every recorded meeting contains the voices and statements of the other attendees; every chat thread contains the other parties' messages; every email contains the sender's and other recipients' data. The user's own participation is **not** the other participants' consent. The archive is, in PDPA terms, the processing of *third-party* personal data, and the third parties are data subjects with enforceable rights against the user as the data controller — including a right of access (s.12, s.30) and a right to prevent processing likely to cause substantial damage or distress (s.42) [S1].

**3. There is no blanket "employment exception" in the PDPA 2010.** The ticket premise is loose. The only employment-linked exception in the Act is s.40(1)(b)(i), and it is confined to **sensitive personal data** and to rights or obligations *conferred or imposed by law* "in connection with employment" (not by contract) [S1]. There is no general exemption for employee, employer, or workplace processing anywhere in Part III (ss.45–46) [S1]. The relevant question is not "is there an employment exception" but **"who is the data controller"** — the individual archivist, or their employer.

**4. Malaysia has no two-party-consent recording statute, and "one-party vs two-party consent" is the wrong frame.** That framing is imported from United States wiretap law. Malaysia has no statute that requires the consent of all parties before a conversation may be recorded, and equally no statute that grants a participant a positive right to record. The real Malaysian touchpoints are: (a) PDPA processing rules — consent/notice/limited purpose; (b) the **Communications and Multimedia Act 1998 (Act 588) s.234** (interception), whose penalty was raised to **RM500,000 / 5 years** by the Communications and Multimedia (Amendment) Act 2025 (Act A1743) s.93 [S6][S7]; and (c) the **Evidence Act 1950 (Act 56)**, under which admissibility turns on *relevance and authenticity*, not on the lawfulness of the recording [S8][S9].

**5. Statute is a floor; company policy is the binding practical constraint, and it can be stricter.** A company may prohibit recording, may restrict the use of personal devices, and may require data-loss-prevention controls. Breach of an employer's policy is an employment-law risk (misconduct, and breach of the implied duty of fidelity and good faith) *even where the statute permits the conduct* [S18]. Policy cannot authorise what a statute forbids, and a permissive policy does not supply the data subjects' PDPA consent.

**6. The defensible boundary is therefore narrower than "participant-only".** Participant status is a *necessary* condition, not a sufficient one. The defensible scope is: **participant-only, overt wherever practicable, purpose-limited to the user's own recall, never disclosed to third parties, sensitive-data-excluded absent explicit consent, locally encrypted with no cross-border transfer, retention-limited, and compliant with the employer's policy and DLP rules.** Where any of those conditions fails — most acutely with **covert recording** — the residual risk concentrates and the boundary is no longer defensible on the current state of Malaysian law.

**7. There is a structural limit that a product decision should confront now.** Because other participants are data subjects, they can serve an access request under s.30 and require a copy of the personal data the user holds about them [S1]. A "private dossier" framing is in tension with that right. The honest position is that the archive is *single-user by design*, not *inaccessible by right*.

---

## 2. Corrections to the ticket's framing

The seed context for this ticket contained several leads that did not survive verification against primary sources. They are recorded here so the spec does not inherit them.

| Ticket premise | Verified position |
|---|---|
| There is an "employment exception" in the PDPA 2010 | **Incorrect as stated.** There is no blanket employment exemption. The only employment-linked provision is s.40(1)(b)(i), limited to *sensitive* personal data and to rights/obligations imposed by **law** in connection with employment [S1]. |
| "s.40 (disclosure)" | **Wrong section.** s.40 is *Processing of sensitive personal data*. Disclosure is governed by the **Disclosure Principle in s.8** and by **s.39** (*Extent of disclosure of personal data*) [S1]. |
| "PDPA 7 principles" | **Correct.** The seven Personal Data Protection Principles are set out in s.5(1) and elaborated in ss.6–12: General, Notice and Choice, Disclosure, Security, Retention, Data Integrity, Access [S1]. |
| "State variance unlikely" | **Confirmed.** Act 709 is federal legislation applying throughout Malaysia; no State personal-data-protection statute was located. Note s.3(1) is an *institutional* carve-out: the Act does not apply to the **Federal Government and State Governments** [S1]. |
| "s.3 application/exclusions — federal/state government excluded" | **Confirmed, and importantly *not* amended by the 2024 amendment.** Act A1727 makes no amendment to s.3 [S2]. |
| "Malaysia has no explicit two-party-consent statute" | **Confirmed — but this is not a permission.** There is also no statutory participant carve-out in CMA 1998 s.234, and no Malaysian authority located that decides whether a participant recording their own conversation commits the s.234 offence. See §6. |
| "the 2024 amendment (breach notification / DPO) applies" | **Correct, with a phased commencement that matters.** Commencement is **not** a single date: 1 Jan 2025, 1 Apr 2025 and 1 Jun 2025 for different provisions. See §4. |
| "Lee Ewe Poh v Dr Lim Teik Man [2011] 1 MLJ 835" | **Citation correct**, also reported at **[2011] 4 CLJ 397**; High Court Malaya, Pulau Pinang, Chew Soo Ho JC, judgment 2 September 2010 [S10]. |
| Implied: "PDPA has no household exemption" | **Wrong — it does.** s.45(1) exempts personal data "processed by an individual only for the purposes of that individual's personal, family or household affairs, including recreational purposes" [S1]. Whether a *work* archive fits that exemption is the key boundary question — see §3.4. |

---

## 3. Does the PDPA 2010 apply to a participant-only work archive?

### 3.1 Application: "in respect of commercial transactions"

s.2(1) provides that the Act applies to "any person who processes; and any person who has control over or authorizes the processing of, any personal data **in respect of commercial transactions**" [S1]. s.2(2) adds a territorial test: the person must be established in Malaysia (an individual is treated as established if physically present in Malaysia for not less than 180 days in one calendar year) or, if not established here, must use equipment in Malaysia for processing other than for transit [S1].

s.4 defines **"commercial transactions"** as "any transaction of a commercial nature, whether contractual or not, which includes any matters relating to the supply or exchange of goods or services, agency, investments, financing, banking and insurance", excluding credit reporting [S1].

Two points follow, and they cut in opposite directions:

- **Employment is not expressly listed.** The definition names supply/exchange of goods or services, agency, investments, financing, banking and insurance. It does **not** say "employment". Commentary routinely treats employer–employee processing as within scope (on the footing that employment is a supply or exchange of services), but I could not locate a primary Malaysian source — statute, gazette order, regulator guideline, or reported judgment — that squarely holds employment to be a "commercial transaction" for s.2 purposes. **This is a genuinely unresolved point and is recorded as such in §11.**
- **For this project the doubt is largely academic.** The archive is built for *work* purposes — recall of meetings, chats and correspondence arising from the user's working life, with a view to potential use as evidence in a dispute. That is not a hobby or a household activity. Whether the connection is characterised through employment, through the user's own trade or profession, or through the employer's commercial activity, the processing is commercial in character. The prudent design assumption is that the Act applies.

### 3.2 Who is the data controller? The individual, or the employer?

This is the question the ticket's "employment exception" framing was really circling, and it is the single most consequential determination in this memo.

- s.4 defines a **"data controller"** (renamed from "data user" by Act A1727, effective 1 April 2025) as a person who "either alone or jointly or in common with other persons processes any personal data or has control over or authorizes the processing of any personal data, but does not include a data processor" [S1][S2].
- s.4 defines a **"data processor"** as "any person, **other than an employee of the data user**, who processes the personal data solely on behalf of the data user, and does not process the personal data for any of his own purposes" [S1] (emphasis added).

The express exclusion of employees from the definition of "data processor" is significant. It means that when an employee processes personal data in the course of their employment, the employee is neither a data processor nor, for that processing, a separate data controller: **the employer is the data controller and the employee's processing is attributed to the employer.** The employer therefore carries the statutory obligations — the security policy, the retention standard, the DPO appointment, the breach notification — for data processed by its staff.

The consequence for Footprint is that the archive's legal character depends on *why* it exists:

- **Personal-archive characterisation.** If the archive is created and held by the individual, for the individual's own purposes (personal recall, and evidence to protect the individual's own position), on the individual's own device, not on the employer's systems, and not under the employer's direction — then the individual is the data controller for their own archive, and the analysis in §§3.3–3.5 applies to them directly.
- **Employment-processing characterisation.** If the archive is made in the course of employment, captures the employer's confidential information or client data, is stored on employer systems or devices, or is used for work purposes in a way the employer directs or benefits from — then the processing is likely to be treated as the employer's, and the employer's policy, DPO regime and DLP controls apply to it. In that characterisation the individual's private side-project is *unauthorised processing by an employee of the employer's data controller obligations*, and is very likely a breach of the employment contract and of the employer's acceptable-use policy.

**The practical boundary:** the archive should be personal, not employer-directed; the user should be the controller; and the employer's policy and DLP classification should be checked *before* any work content is ingested (see §7 and the checklist in §10).

### 3.3 The seven principles, mapped onto the archive

s.5(1) requires processing to comply with seven principles, elaborated in ss.6–12 [S1]. Applied to a participant-only archive:

| Principle | Provision | What it demands of the archive |
|---|---|---|
| **General** | s.6 | Do not process personal data without the data subject's consent, unless a s.6(2) ground applies. Sensitive personal data may only be processed in accordance with s.40. Additionally, data must be processed for a **lawful purpose directly related to an activity of the controller**, the processing must be **necessary** for that purpose, and the data must be **adequate but not excessive** (s.6(3)). |
| **Notice and Choice** | s.7 | The controller must, by **written notice** (in the national language **and** English), inform each data subject that their data is being processed, of the purposes, of the source, of access/correction rights, of the classes of third parties to whom it may be disclosed, of the choices available for limiting processing, and of whether supply is obligatory. The notice must be given as soon as practicable when data is first collected (s.7(2)). |
| **Disclosure** | s.8 (subject to s.39) | No disclosure to any party other than a class notified under s.7(1)(e), and no disclosure for a purpose other than the original or a directly related purpose, without consent. s.39 sets out the closed list of exceptions. |
| **Security** | s.9 | Take **practical steps** to protect data from loss, misuse, modification, unauthorised or accidental access/disclosure, alteration or destruction, having regard to the nature of the data, where it is stored, the equipment, personnel reliability, and secure transfer. |
| **Retention** | s.10 | Data must not be kept longer than necessary for the purpose; the controller must take all reasonable steps to destroy or permanently delete data no longer required. |
| **Data Integrity** | s.11 | Take reasonable steps to ensure data is accurate, complete, not misleading and kept up to date. |
| **Access** | s.12 (with ss.30–37) | Data subjects must be given access to their personal data and be able to correct it. |

Two of these deserve emphasis because they are structural rather than operational:

- **The Notice and Choice Principle is close to incompatible with covert recording.** s.7 requires written notice to each data subject as soon as practicable when their data is first collected [S1]. A recording made without the participants' knowledge cannot satisfy that. This is a *statutory* problem, independent of any company policy, and it is the strongest legal argument that **overt recording is the only clearly defensible mode** for this archive.
- **The Access Principle runs against the "private dossier" instinct.** Other participants are data subjects. On request they are entitled to access the personal data the user holds about them, and to have it corrected (ss.12, 30, 31, 34, 35) [S1]. A design that assumes the archive is legally invisible to the people in it is mistaken.

### 3.4 The household exemption (s.45(1)) — and why a work archive probably fails it

s.45(1) provides: "There shall be exempted from the provisions of this Act personal data processed by an individual **only for the purposes of that individual's personal, family or household affairs, including recreational purposes**" [S1].

The Commissioner's own public gloss on the Act describes this as "a comprehensive exemption to personal data processed by individuals for the purpose of personal, family or household affairs including recreation" [S16].

The exemption is the closest thing in Malaysian law to a private-use carve-out, and it is tempting to rely on it. **It should not be relied on here.** The words "**only** for the purposes of that individual's personal, family or household affairs" are a purpose test, not a location or device test. A Personal *Work* Footprint — assembled to recall work meetings, work chats and work correspondence, and intended to be usable as evidence in a workplace dispute — is a working or business purpose. It is not a family or household affair. The exemption is also the *only* privacy carve-out of its kind in the Act, so it is likely to be read no more generously than its words require.

**Conclusion: the s.45(1) household exemption should be treated as unavailable, and the archive designed on the assumption that the full Act applies.** If a narrower "personal notes only" variant of the product were ever contemplated (a private journal, not a work record), the analysis would differ — but that is not the destination described in issue #1.

### 3.5 What does *not* apply in practice

Three obligations that dominate most PDPA compliance programmes are, on the facts, unlikely to bite on a genuinely single-user personal archive. They should be checked, but not over-engineered for.

- **Registration (ss.14–16).** Registration is required only for data controllers falling within a class prescribed by order under s.14(1). The prescribed classes are in the Personal Data Protection (Class of Data Users) Order 2013 [P.U.(A) 336], as amended by P.U.(A) 326/2016 — communications licensees, banking and financial institutions, insurers, private healthcare, tourism and hospitality, airlines, private education, direct sales, professional and retail services, housing developers and utilities [S5]. **There is no employment, "employer", or general-business class.** Failure to register when required is an offence carrying a fine up to RM500,000 or 3 years' imprisonment (s.16(4)) [S1][S17], so if the archive were ever operated as a service for third parties the class question would need re-examination — but an individual's own archive does not fall within a prescribed class.
- **Data Protection Officer (s.12A).** Inserted by Act A1727 and in force from **1 June 2025** [S2][S3]. The Act itself states the obligation without a threshold; the conditions are set by the Commissioner's Circular No. 1/2025 and the DPO Guideline, which require appointment where processing involves **more than 20,000 data subjects**, or **sensitive personal data (including financial information) of more than 10,000 data subjects**, or **activities requiring regular and systematic monitoring of personal data** [S12]. A single-user personal archive does not meet those thresholds. Note, however, that if the employer meets them, the employer's DPO regime governs work data processed for the employer.
- **Data portability (s.43a).** Inserted by Act A1727, in force 1 June 2025 [S2][S3]. It gives a data subject the right to require transmission of their data to another controller of their choice, subject to technical feasibility and format compatibility. It is unlikely to be exercised against a personal archive, but the underlying design principle — that the data subject's data should be movable at their request — is worth remembering.

### 3.6 Cross-border transfer: keep the store in Malaysia

s.129 was substantially rewritten by Act A1727 s.12, in force **1 April 2025** [S2][S3]. The pre-amendment prohibition — no transfer to any place outside Malaysia unless that place was gazetted by the Minister — was replaced. Under the amended section, a data controller may transfer personal data to a place outside Malaysia where that place has a law substantially similar to Act 709, or ensures an adequate level of protection at least equivalent to that afforded by Act 709 (s.129(2)); and may transfer notwithstanding that, on grounds including the data subject's consent, necessity for a contract, legal proceedings, reasonable precautions and due diligence, and the vital interests of the data subject (s.129(3)) [S14]. The public consultation paper records the change as "abolishing the requirement to gazette permitted places for the transfer of any personal data outside Malaysia" [S15].

This matters directly to Footprint's hard requirement of a **local encrypted store**. Keeping the store in Malaysia, with no cloud synchronisation to foreign servers and no foreign-hosted backup, avoids the entire cross-border analysis — including the question whether the destination jurisdiction's law is "substantially similar" to Act 709. **A local-only design is not just a security choice; it is the cheapest way to stay outside s.129.**

---

## 4. The 2024 amendment, and why the commencement dates matter

The Personal Data Protection (Amendment) Act 2024 (Act A1727) received Royal Assent on 9 October 2024 and was published in the Gazette on 17 October 2024 [S2]. Its commencement is **phased**, appointed by the Minister of Digital under s.1(2) by P.U.(B) 522, dated 19 December 2024 and gazetted 24 December 2024 [S3]:

| Date | Provisions of A1727 in force | Effect |
|---|---|---|
| **1 January 2025** | ss.7, 11, 13, 14 | Amendment of s.16 (national-language text); amendment of s.67 (Fund bank accounts); amendment of s.136 (service of notices by electronic means); saving provision. Largely administrative. |
| **1 April 2025** | ss.2, 3, 4, 5, 8, 10, 12 | **The substantive core.** "Data user" → "**data controller**" throughout (s.2); new definitions including "**personal data breach**" and "**biometric data**" (now sensitive personal data), and "data subject" excluding a deceased individual (s.3); data processors made subject to the Security Principle with the s.5(2) penalty raised to RM1,000,000 / 3 years (s.4); the Security Principle in s.9 extended to data processors (s.5); data controller forums (s.8); deletion of s.48(e) (s.10); **cross-border transfer rewritten (s.12, amending s.129)**. |
| **1 June 2025** | ss.6, 9 | **New Division 1a of Part II**: s.12A (**appointment of a data protection officer**) and s.12B (**data breach notification**); and new **s.43a (data portability)**. |

Two consequences for the spec:

- **Terminology in the codebase and the spec should use "data controller", not "data user".** The rename took effect on 1 April 2025 and applies wherever the words appear in the principal Act (with two carve-outs: the definition of "register" in s.4, and s.9) [S2].
- **Breach notification is now live.** s.12B requires a data controller who has reason to believe a personal data breach has occurred to notify the Commissioner **as soon as practicable**, and to notify affected data subjects **without unnecessary delay** where the breach causes or is likely to cause significant harm [S2]. The Commissioner's Data Breach Notification Guideline (August 2025) operationalises this as **not later than 72 hours** from the occurrence of the breach for notification to the Commissioner, and **not later than 7 days** after the initial notification to the Commissioner for notification to affected data subjects [S13]. Contravention of s.12B(1) is an offence carrying a fine up to RM250,000 or 2 years' imprisonment [S2]. For a personal archive the practical relevance is the *security* obligation that sits behind it: if the encrypted store is breached, the user is a data controller with a notification duty in respect of the other participants' data.

---

## 5. The "employment exception": what actually exists

The ticket asked for the "employment exception" to be covered. What the Act actually contains is narrower and more specific, and should be stated accurately in the spec.

**There is no general exemption for employment processing.** Part III of Act 709 contains only two exemption provisions [S1]:

- **s.45(1)** — the personal/family/household exemption (§3.4 above).
- **s.45(2)** — a closed list of subject-matter exemptions: crime prevention/detection and investigation; apprehension or prosecution of offenders; assessment or collection of tax or duty; physical or mental health information; **statistics or research** (provided the data is not processed for any other purpose and results are not published in identifiable form); court orders and judgments; discharge of regulatory functions; and journalistic, literary or artistic purposes. Employment is **not** among them.
- **s.46** — a power for the Minister, on the Commissioner's recommendation, to make further exemptions by gazetted order.

The only employment-linked provision in the Act is inside the **sensitive personal data** regime:

> s.40(1)(b)(i): a data controller shall not process sensitive personal data except where the processing is necessary "for the purposes of exercising or performing any right or obligation which is **conferred or imposed by law** on the data controller **in connection with employment**" [S1].

Three limits on that exception matter:

1. **It applies only to sensitive personal data** — s.4 defines sensitive personal data as information about physical or mental health or condition, political opinions, religious beliefs or beliefs of a similar nature, the commission or alleged commission of an offence, **biometric data** (added by Act A1727 with effect from 1 April 2025), and anything else the Minister determines by order [S1][S2]. Ordinary meeting content is not sensitive personal data, so s.40 does not apply to it at all.
2. **It is confined to rights or obligations imposed by law**, not by contract. A contractual obligation owed by an employer to an employee does not engage it.
3. **s.40(2)** permits the Minister to exclude or add conditions to the s.40(1)(b)(i) ground by order [S1].

**How this should be framed in the spec:** the correct question is not "does the employment exception cover this?" but "**who is the data controller for this archive, and what is the lawful basis under s.6 or s.40 for processing the other participants' data?**" Consent, contract necessity, legal obligation, vital interests, administration of justice and statutory functions are the s.6(2) grounds available; s.40 governs sensitive data separately [S1].

---

## 6. Recording meetings: what Malaysian law actually says

### 6.1 "One-party vs two-party consent" is the wrong frame

Malaysia has **no** statute requiring the consent of all parties to a recording of a conversation, and **no** statute conferring a right to record. The one-party/two-party dichotomy comes from United States federal and state wiretap law and has no Malaysian statutory analogue. The proposition that "Malaysia is a one-party consent jurisdiction" is a simplification that is best avoided in the spec, because it implies a permission that does not exist. What exists is a set of separate constraints: PDPA processing rules, the CMA 1998 interception offence, and the law of evidence and confidence. Each is examined below.

### 6.2 PDPA 2010

Recording a meeting is "collecting" personal data (s.4 defines "collect" as an act by which personal data enters into or comes under the control of a data controller) [S1]. It therefore engages:

- **s.6** — a lawful basis is needed for processing the other participants' personal data, plus the s.6(3) purpose-limitation, necessity and proportionality tests.
- **s.7** — written notice to each data subject as soon as practicable. This is the provision that most directly bears on covert recording: **a recording made without the participants' knowledge cannot satisfy the Notice and Choice Principle.**
- **s.40** — where the meeting content includes sensitive personal data (health, political opinions, religious beliefs, alleged offences, or biometric data), **explicit consent** is required (s.40(1)(a)) [S1][S2].
- **s.42** — a data subject may serve written notice requiring the controller to cease processing where it is causing or is likely to cause substantial damage or substantial distress and that damage or distress is unwarranted [S1].

A note on **biometric data**: Act A1727 added "biometric data" (any personal data resulting from technical processing relating to the physical, physiological or behavioural characteristics of a person) to sensitive personal data, effective 1 April 2025 [S2]. If the archive stores voice recordings and processes them to identify speakers — for example, speaker diarisation or voice matching — there is a real argument that this is processing of biometric data, attracting the **explicit consent** requirement in s.40(1)(a) and the s.40(3) offence provision. This is flagged as residual risk in §11.

### 6.3 Communications and Multimedia Act 1998 s.234

s.234(1) provides that a person who, **without lawful authority under this Act or any other written law** —

> (a) intercepts, attempts to intercept, or procures any other person to intercept or attempt to intercept, any communications;
> (b) discloses, or attempts to disclose, to any other person the contents of any communications, knowing or having reason to believe that the information was obtained through the interception of any communications in contravention of this section; or
> (c) uses, or attempts to use, the contents of any communications, knowing or having reason to believe that the information was obtained through the interception of any communications in contravention of this section,

commits an offence [S6].

Two definitions matter:

- **"communications"** is defined in s.6 as "any communication, whether between persons and persons, things and things, or persons and things, in the form of sound, data, text, visual images, signals or any other form or any combination of those forms" [S6].
- **"intercept"** is defined in s.6 as "the aural or other acquisition of the contents of any communications through the use of any electronic, mechanical, or other equipment, device or apparatus". Act A1743 s.6(d) broadened this to "the acquisition or reproduction or both of contents of any communications by way of aural or otherwise" [S6][S7].

The penalty in s.234(3) was **raised by the Communications and Multimedia (Amendment) Act 2025 (Act A1743) s.93** from a fine not exceeding RM50,000 / 1 year to a **fine not exceeding RM500,000 / imprisonment not exceeding 5 years**, or both [S7].

What can and cannot be concluded:

- **s.234 contains no participant carve-out.** Unlike some Commonwealth interception statutes, it does not say "without the authority of the sender or the recipient". So the text does not itself answer whether a participant recording their own conversation is "intercepting" it.
- **No Malaysian authority was located deciding the point.** I searched for and did not find a reported Malaysian judgment, regulator guideline, or MCMC instrument addressing whether a party to a conversation commits an offence under s.234 by recording it. **This is an unresolved question, and the spec should not assume it is settled in the user's favour.**
- **Context suggests s.234 is aimed at network-borne communications.** The CMA's interception architecture — "authorized interception" (defined as interception by a licensee permitted under s.265), the s.265 network interception capability obligation on licensees, and the s.252 power of the Public Prosecutor to authorise interception for investigations under the Act — is concerned with communications carried over network facilities and services [S6]. An in-person, non-networked meeting is therefore the weakest case for the section's application; a telephone call, or a video-conference carried over a network, is the strongest. On the statutory language alone, however, "communications" and "intercept" are drafted broadly enough that a cautious designer should not treat the in-person case as clearly outside.
- **Practical conclusion:** the participant-only design materially reduces CMA exposure relative to third-party recording — a participant is at least not a stranger acquiring someone else's communication — but it does not eliminate it. Covert recording of network-borne communications is the scenario in which CMA s.234 risk is highest, and the penalty is now substantial.

### 6.4 Evidence Act 1950 — admissibility

The Evidence Act 1950 does not contain a provision excluding evidence because it was unlawfully obtained. In **Mohd Ali Jaafar v Public Prosecutor [1998] 4 MLJ 210** (High Court, Melaka; Augustine Paul J), the court held that a tape recording is a **document within the meaning of s.3** of the Evidence Act 1950, and that its proof is governed by **ss.61 to 66** of that Act: the recording must be produced for the inspection of the court and played over, and the voices and the accuracy of the conversation must be proved. The court also approved the statement of principle that a judge has a discretion to exclude strictly admissible evidence where its prejudicial effect greatly outweighs its evidential value, following *R v Sang* [1980] AC 402 [S9].

In other words: **admissibility is a question of relevance and authenticity, not of the lawfulness of the acquisition.** The practical consequence for the archive is that the *integrity of the record* — an unbroken chain of custody, proof that the recording has not been altered, and the ability to identify the speakers — matters more to its evidential value than whether the recording was overt or covert.

The Industrial Court applies a related but distinct standard. Under **s.30(5) of the Industrial Relations Act 1967**, the Industrial Court is to act according to equity, good conscience and the substantial merits of the case without regard to technicalities and legal form. The following propositions are drawn from the Skrine analysis of the Industrial Court authorities [S18] — **the primary awards themselves are published in the Industrial Law Reports / LNS series, which are not freely accessible, and I was not able to retrieve them in full. They should be treated as secondary-sourced and verified before being relied on in a filing.**

- *Sanjungan Sekata Sdn Bhd v Liew Tiam Seng* [2003] 3 ILR 1155 — surreptitious tape recordings of a conversation in an employee's office were admitted, subject to the authenticity guidelines in *Mohd Ali Jaafar v PP* [1998] 4 CLJ 208. The Industrial Court's criteria addressed authenticity and chain of custody; **none of them included whether the recording was made without consent or knowledge** [S18].
- *Yap Fat v Southern Investment Bank Bhd / Southern Bank Berhad* [2010] 3 ILR 350 — admissibility of illegally obtained evidence, if relevant, is consistent with s.30(5) IRA 1967 [S18].
- *Justin Maurice Read v Petroliam Nasional Berhad* [2017] 3 ILR 527 — a covertly made handphone recording, later copied to a personal computer and a pen drive, was **not** admitted: there was a break in the chain of possession, the recorded persons could not confirm accuracy, and the claimant could not eliminate doubt of tampering or editing. The court also criticised surreptitious recording as unethical — while noting there is no legal bar to admissibility [S18].
- *Izaidin Joinnie v Amanah Saham Sarawak Berhad* [2018] 2 LNS 1787 — a compliance officer who secretly recorded a board meeting. Although this was not a stated reason for dismissal, the Industrial Court held **obiter** that the secret recording was the "ultimate act of incompatibility", that "there could be no excuse and no justification" for it, and that the officer had acted in **breach of his duty of fidelity or good faith and confidence** owed to the company. His unjust-dismissal claim failed [S18].

The last two authorities are the ones that should shape the product's risk posture. They establish that **the evidential value of a covert recording is fragile** (authenticity and chain of custody must survive scrutiny, and a personal-device pipeline is exactly the pattern that failed in *Justin Maurice Read*), and that **the act of covertly recording a workplace meeting can itself be characterised as misconduct** — a breach of the duty of fidelity and good faith — even where the recording is not unlawful.

### 6.5 Civil liability: privacy and breach of confidence

- **Lee Ewe Poh v Dr Lim Teik Man & Anor [2011] 4 CLJ 397** (HC, Pulau Pinang; Chew Soo Ho JC, 2 September 2010): a surgeon photographed a patient's private part during surgery without her knowledge or consent. The court held that, the Court of Appeal in *Maslinda Ishak v Mohd Tahir Osman & Ors* [2009] 6 CLJ 653 having accepted invasion of privacy as a cause of action (albeit without the point being squarely argued), the plaintiff could maintain a claim for invasion of privacy under the Malaysian common law. In the alternative, the court found the **three elements of breach of confidence** satisfied: the information had the necessary quality of confidence; it was imparted in circumstances importing an obligation of confidence; and there was unauthorised use or disclosure [S10].
- **Dr Bernadine Malini Martin v MPH Magazines Sdn Bhd & Ors [2006] 2 CLJ 1117** — the court held that the law of Malaysia "does not make an invasion of privacy an actionable wrongdoing" [S19, secondary].

The two decisions are difficult to reconcile, and the position is unsettled. The reliable point for present purposes is the **alternative** holding in *Lee Ewe Poh*: **breach of confidence is a well-established cause of action in Malaysia and does not depend on the existence of a free-standing tort of privacy.** A recorded meeting may contain information imparted in confidence — commercial information, personal disclosures, privileged legal discussion — and its recording, retention or disclosure can found a breach-of-confidence claim irrespective of whether a privacy tort is available. For a participant who is present when the information is disclosed, the "circumstances importing an obligation of confidence" element is fact-sensitive and may be weaker than in *Lee Ewe Poh* (a doctor–patient relationship), but it is not absent, particularly for employer confidential information and client data.

### 6.6 Synthesis on recording consent

The honest statement of Malaysian law for the spec is:

1. There is no statutory two-party consent requirement, and no statutory one-party permission.
2. A participant recording their own conversation does not obviously fall within CMA s.234, but no Malaysian authority decides the question, and the penalty for a contravention is now RM500,000 / 5 years.
3. The PDPA imposes a **notice** obligation (s.7) that covert recording cannot satisfy, and a **consent or other lawful basis** requirement (s.6, and s.40 for sensitive data).
4. Covertly obtained recordings are generally **admissible** in Malaysian proceedings if relevant and authentic, but authenticity is a real hurdle, and the *act* of covertly recording a workplace meeting can be **misconduct** in employment law and can found a **breach of confidence** claim.
5. Therefore: **overt recording, with notice, is the defensible mode.** Covert recording is the mode where every one of the above risks concentrates, and where the product's "not covert surveillance" positioning is hardest to sustain.

---

## 7. Company policy versus statute

Three rules of precedence should be stated plainly in the spec and in the checklist.

**7.1 The statute is a floor, not a ceiling.** PDPA 2010, CMA 1998 and the Evidence Act 1950 set minimum standards and criminal or regulatory consequences. Nothing prevents an employer from imposing stricter requirements — prohibiting recording entirely, banning personal devices in meeting rooms, requiring DLP classification and endpoint controls, or requiring express written authorisation before work content is exported. Where policy is stricter than the statute, **policy governs the employment relationship.**

**7.2 Breach of policy is a disciplinary and employment risk even where the statute permits the conduct.** The Industrial Court's reasoning in *Izaidin Joinnie* is directly on point: secretly recording a board meeting was treated as a breach of the **duty of fidelity or good faith and confidence** owed to the company, and the employee's claim failed notwithstanding that the recording was not itself unlawful [S18]. The practical implication for Footprint is that **a technically lawful participant-only archive can still be misconduct** if it breaches the employer's acceptable-use, confidentiality or DLP policies, or if it captures confidential information the employee is bound to protect.

**7.3 Policy cannot authorise what a statute forbids, and a permissive policy does not supply consent.** An employer cannot waive a statutory offence or a third party's PDPA rights by policy. Even a policy that expressly permits recording does not provide the other participants' consent under s.6, does not discharge the s.7 notice obligation to those participants, and does not make sensitive personal data lawful under s.40. Policy permission and statutory compliance are separate requirements, both of which must be met.

**7.4 The employer's DLP classification is the operative filter.** If work content is classified by the employer as confidential, restricted, or subject to DLP controls, then ingesting it into a personal archive is a policy breach regardless of the archive's own security. The boundary condition for the product is that **the user must be able to exclude employer-classified content**, and the checklist should require the user to confirm they have checked.

---

## 8. Confidential information, DLP and sectoral constraints

**8.1 Employer confidential information and third-party personal data** are the two highest-value contents an archive is likely to capture, and both are protected independently of the PDPA:

- **Breach of confidence** — the alternative holding in *Lee Ewe Poh* [S10] sets out the three-element test. Commercial information, client data and internal deliberations disclosed in a meeting can satisfy it.
- **Contractual and implied duties** — confidentiality clauses in the employment contract, and the implied duty of fidelity and good faith (see *Izaidin Joinnie* [S18]).

**8.2 Official secrets (public sector users).** The Official Secrets Act 1972 (Act 88) s.8 makes it an offence for a person in possession or control of an official secret to communicate it to an unauthorised person, or to retain it in contravention of his duty, among other things. s.2 defines "official secret" to include any document specified in the Schedule and any information or material classified "Top Secret", "Secret", "Confidential" or "Restricted" by a Minister, a Menteri Besar or Chief Minister of a State, or an appointed public officer [S11]. A public-sector user archiving work meetings must therefore treat OSA s.8 as a hard exclusion boundary, separate from and additional to the PDPA. Note also the interaction with s.3(1) PDPA: the Act does not apply to the Federal Government and State Governments [S1], so a civil servant's work processing may sit outside the PDPA while remaining squarely inside the OSA and public-service disciplinary rules. **The PDPA exclusion for government does not mean the data is unprotected — it means a different regime applies.**

**8.3 Sectoral DLP rules.** Beyond the PDPA, sectoral regulators impose their own data-management and loss-prevention obligations on regulated firms — for example, banking and financial institutions, insurers, and licensed communications providers. I did not retrieve and verify the text of any specific sectoral DLP instrument within the scope of this memo, and I therefore do not cite one. The checklist item is nevertheless important: **if the user works in a regulated sector, the sectoral regulator's data-management requirements may be stricter than the PDPA, and must be checked separately.**

**8.4 The PDP Standard 2015 is the concrete security benchmark.** The Personal Data Protection Regulations 2013 [P.U.(A) 335] require a data controller to develop and implement a security policy for the purposes of s.9, and to comply with the security, retention and integrity standards prescribed by the Commissioner [S15]. The Commissioner's **Personal Data Protection Standard 2015** is the operative instrument. Its security standard for electronically processed personal data requires, among other things: registering all personnel involved in processing; terminating access rights on cessation of employment or contract; controlling and limiting staff access to personal data by purpose; issuing user IDs and passwords to authorised staff; revoking credentials immediately when no longer required; and physical security procedures for data storage locations [S4]. The Standard also prescribes retention and data-integrity standards, and imposes requirements on the secure transfer of personal data, including through cloud computing services [S4][S15].

For Footprint, the Standard 2015 is the natural yardstick for the encrypted local store: encryption at rest, single-user access control, credential management, physical security of the device, and no uncontrolled transfer. The Commissioner's public consultation on amending P.U.(A) 335 — Public Consultation Paper No. 4/2025, open 22 August to 8 September 2025 — confirms that the security, retention and integrity standards are under revision to align with the 2024 amendments [S15]. **The Standard may therefore change; the checklist should reference it by name and date rather than hard-coding its requirements.**

---

## 9. The defensible boundary statement on participant-only scope

This is deliverable (a). It is written to be quoted directly into the spec (issue #6).

> **Boundary statement.** The archive may be treated as defensible under Malaysian law only where **all** of the following conditions hold. Participant status is a necessary condition, but it is not sufficient on its own.
>
> **B1. Genuine participation.** The user was a party to the communication, a named recipient of the message, or an invited attendee of the meeting. Content the user merely overheard, was forwarded, or obtained from a third party is out of scope.
>
> **B2. Overt collection.** The user does not conceal the fact of capture from the other participants. Where the meeting is recorded, the participants are informed, or the recording is made in circumstances in which participation in a recorded meeting is the established and disclosed practice. **Covert recording is out of scope**, because it cannot satisfy the Notice and Choice Principle (PDPA s.7) and concentrates the CMA s.234, breach-of-confidence and employment-misconduct risks.
>
> **B3. Purpose limitation to the user's own recall.** The sole purpose is the user's own memory, continuity and ability to protect their own position. The archive is not used to monitor, evaluate, rank, or build a dossier on any other person, and is not used for any unlawful purpose.
>
> **B4. No disclosure to any third party.** No sharing, publishing, forwarding or granting of access to any other person, except where compelled by law or required to comply with a valid access or correction request under PDPA ss.30–37. The Disclosure Principle (s.8) and s.39 exceptions are the outer limit.
>
> **B5. Sensitive personal data excluded or explicitly consented to.** Health, political opinions, religious or similar beliefs, commission or alleged commission of an offence, and biometric data are excluded from the archive unless the user holds **explicit** consent (PDPA s.40(1)(a)). Speaker-identification features that process voice characteristics should be treated as biometric processing until determined otherwise.
>
> **B6. Local, encrypted, single-user storage.** The store remains in Malaysia; it is encrypted at rest; access is limited to the user; there is no cloud synchronisation, no foreign backup, and no cross-border transfer (PDPA s.129, as amended with effect from 1 April 2025).
>
> **B7. Retention limited to the purpose, with deletion honoured.** Data is retained no longer than necessary (s.10), and is deleted on the data subject's request or on a s.42 notice, subject only to a lawful ground for refusal.
>
> **B8. Data-subject rights honoured.** Requests from other participants for access to, or correction of, their own personal data are acknowledged and dealt with (ss.12, 30–37).
>
> **B9. Employer policy and DLP classification respected.** The user has confirmed that the archive complies with their employer's acceptable-use, confidentiality, and data-loss-prevention policies, and that no content classified as confidential or restricted by the employer is ingested.
>
> **B10. Not a service to others.** The archive is a personal single-user tool. It is not offered to third parties as a product or service, which would raise the registration (s.15), DPO (s.12A) and cross-border questions in a materially different form.

**Where the boundary is weakest.** The conditions most likely to fail in practice, and the ones the spec should design against, are **B2 (overt collection)**, **B4 (no disclosure)** and **B9 (employer policy)**. Of these, B2 is the one that cannot be engineered around: it is a behavioural condition, and the product should either enforce or strongly nudge it, because a covert archive is not a defensible product under the current state of Malaysian law.

---

## 10. Policy checklist for issue #7

This is deliverable (b). It is written as a checklist that the company-policy ticket can operationalise directly, with the statutory or other source for each item.

### A. Scope and purpose

| # | Item | Source |
|---|---|---|
| A1 | Define the permitted scope as **participant-only**: communications to which the user is a party, a named recipient, or an invited attendee. | Product scope; PDPA s.6(3) necessity/minimality [S1] |
| A2 | State the **sole purpose** as the user's own recall and protection of their own position; prohibit use for monitoring, evaluation or dossier-building on others. | PDPA s.6(3)(a)–(b) [S1] |
| A3 | Require an explicit **proportionality test** before ingest: is this artefact necessary for the stated purpose? | PDPA s.6(3)(c) [S1] |
| A4 | Prohibit ingest of any content the employer classifies as confidential, restricted, or DLP-controlled. | Employer policy; OSA 1972 s.8 for public-sector content [S11] |

### B. Lawful basis and notice

| # | Item | Source |
|---|---|---|
| B1 | Require that recording be **overt**; participants informed, or a disclosed and established practice. | PDPA s.7 (Notice and Choice) [S1] |
| B2 | Where notice is given, ensure it covers the s.7(1) items: that data is processed, a description, the purposes, the source, access/correction rights, the classes of third parties, the choices for limiting processing, and whether supply is obligatory. | PDPA s.7(1) [S1] |
| B3 | Provide the notice in **both the national language and English**. | PDPA s.7(3) [S1] |
| B4 | Record the **lawful basis** for processing (consent, or a s.6(2) ground) for each ingest source. | PDPA s.6(1)–(2) [S1] |
| B5 | Treat consent in an employment context with caution: document that consent was freely given and can be withdrawn. | PDPA s.38 (withdrawal) [S1] |

### C. Sensitive personal data

| # | Item | Source |
|---|---|---|
| C1 | Exclude sensitive personal data by default; permit only with **explicit** consent. | PDPA s.40(1)(a) [S1] |
| C2 | Define sensitive personal data to include health, political opinions, religious or similar beliefs, commission or alleged commission of an offence, and **biometric data**. | PDPA s.4, as amended by A1727 s.3(c) (in force 1 Apr 2025) [S1][S2] |
| C3 | Treat speaker-identification or voice-matching features as potential biometric processing pending legal review. | PDPA s.4 "biometric data" [S2]; see §11 |
| C4 | Note that the s.40(1)(b)(i) employment ground is limited to rights/obligations imposed by **law**, and does not cover ordinary contractual obligations. | PDPA s.40(1)(b)(i), s.40(2) [S1] |

### D. Security

| # | Item | Source |
|---|---|---|
| D1 | Encrypt the store at rest; single-user access control; credential management. | PDPA s.9(1); PDP Standard 2015, security standard [S1][S4] |
| D2 | Keep the store **in Malaysia**; no cloud sync, no foreign backup. | PDPA s.129 as amended (1 Apr 2025) [S2][S3][S14] |
| D3 | Secure the physical device; apply physical storage controls. | PDP Standard 2015 [S4] |
| D4 | Where any processor is engaged, ensure they comply with the Security Principle. | PDPA s.5(1a) and s.9(1)–(2), as amended by A1727 (1 Apr 2025) [S2] |
| D5 | Maintain an unbroken chain of custody with integrity hashes, so that the record's authenticity can be proved if relied on. | Evidence Act 1950 ss.3, 61–66; *Mohd Ali Jaafar v PP* [1998] 4 MLJ 210 [S8][S9] |
| D6 | Do not edit, re-encode or transcode originals; keep originals immutable and derivations separate. | *Justin Maurice Read v Petroliam Nasional Bhd* [2017] 3 ILR 527 (via [S18]) |

### E. Retention and deletion

| # | Item | Source |
|---|---|---|
| E1 | Set retention limits tied to the stated purpose; delete when the purpose is spent. | PDPA s.10(1)–(2) [S1] |
| E2 | Implement deletion on request by a data subject, and on a s.42 notice, subject to lawful grounds for refusal. | PDPA ss.38, 42 [S1] |
| E3 | Log deletions so that compliance can be demonstrated. | PDPA s.44 (records) [S1] |

### F. Data-subject rights

| # | Item | Source |
|---|---|---|
| F1 | Provide a process to receive and answer access requests from other participants. | PDPA ss.12, 30, 31 [S1] |
| F2 | Provide a process for correction requests. | PDPA ss.34, 35 [S1] |
| F3 | Provide a process to receive a notice to cease processing on damage/distress grounds. | PDPA s.42 [S1] |
| F4 | Document the grounds on which a request may lawfully be refused. | PDPA ss.32, 36 [S1] |

### G. Third parties and confidentiality

| # | Item | Source |
|---|---|---|
| G1 | Prohibit disclosure to any third party; no sharing, forwarding or access grants. | PDPA ss.8, 39 [S1] |
| G2 | Prohibit publication of any content, including on social media. | PDPA s.8; *Lee Ewe Poh* [S10] |
| G3 | Exclude employer confidential information, client data and privileged legal material. | Breach of confidence (*Lee Ewe Poh* [S10]); implied duty of fidelity (*Izaidin Joinnie*, via [S18]) |
| G4 | Exclude official secrets (public-sector users). | OSA 1972 ss.2, 8 [S11] |

### H. Governance, breach and review

| # | Item | Source |
|---|---|---|
| H1 | Assign responsibility for the archive to a named person (self-assessed, given the single-user design). | PDPA s.12A (DPO) — threshold-based; see §3.5 [S2][S12] |
| H2 | Document the breach-notification route and timelines, in case the store is compromised. | PDPA s.12B (1 Jun 2025) [S2]; DBN Guideline: 72 hours to the Commissioner, 7 days to data subjects [S13] |
| H3 | Confirm whether registration is required — for a personal single-user archive it is not, as employment is not a prescribed class. | PDPA ss.14–16; P.U.(A) 336/2013 [S1][S5][S17] |
| H4 | Re-check the employer's acceptable-use, confidentiality and DLP policies, and the sectoral regulator's requirements where applicable. | §7, §8.3 |
| H5 | Re-review this checklist whenever the PDP Regulations 2013 or the PDP Standard are amended. | Public Consultation Paper No. 4/2025 (22 Aug – 8 Sep 2025) [S15] |

---

## 11. Open questions / residual risk

These are the points on which this memo cannot give a firm answer, and which the spec should carry as explicit residual risk.

**R1. Whether employment processing is a "commercial transaction" under s.2 is not authoritatively settled.** The s.4 definition of "commercial transactions" does not mention employment. I could not locate a primary Malaysian source resolving the point. The memo proceeds on the prudent assumption that the Act applies to work-related processing. *Impact: if the point were ever resolved the other way, a large part of the PDPA analysis would fall away — but the confidentiality, OSA, CMA and employment-policy risks would not.*

**R2. Whether a participant recording their own conversation is an "interception" under CMA 1998 s.234 is undecided.** No Malaysian judgment, guideline or regulator instrument was located. The section has no participant carve-out, "communications" and "intercept" are broadly defined, and the penalty is now RM500,000 / 5 years. *Impact: covert recording of network-borne communications (calls, video conferences) carries the highest exposure. Overt recording and in-person meetings carry less, but not zero.*

**R3. Whether speaker-identification features process "biometric data" is unresolved.** "Biometric data" was added to sensitive personal data with effect from 1 April 2025. The definition — "any personal data resulting from technical processing relating to the physical, physiological or behavioural characteristics of a person" — arguably captures voice processing for identification. If so, explicit consent under s.40(1)(a) is required, and contravention is an offence under s.40(3). *Impact: a design decision on voice matching / speaker diarisation should be taken with legal input.*

**R4. The status of a free-standing tort of invasion of privacy in Malaysia is unsettled.** *Lee Ewe Poh* (HC, 2010) held it actionable, reasoning from the Court of Appeal's non-decision in *Maslinda Ishak*; *Dr Bernadine Malini Martin* (2006) held the opposite. *Impact: breach of confidence is the more reliable cause of action against an archive, and the risk analysis should be run on that basis rather than on privacy.*

**R5. The Industrial Court authorities were not retrieved in primary form.** *Sanjungan Sekata* [2003] 3 ILR 1155, *Yap Fat* [2010] 3 ILR 350, *Justin Maurice Read* [2017] 3 ILR 527 and *Izaidin Joinnie* [2018] 2 LNS 1787 are reported in the Industrial Law Reports / LNS series, which are not freely accessible. The propositions in §6.4 are taken from the Skrine analysis [S18] and **should be verified against the primary awards before being relied on in any filing.**

**R6. The proposition attributed to *Benjamin William Hawkes v Public Prosecutor* [2020] 5 MLJ 417 was not verified.** A secondary commentary cites this Federal Court decision for the proposition that illegally obtained evidence is admissible because admissibility turns on relevancy [S19]. I was unable to retrieve the primary judgment, and the accessible summaries suggest the case turned principally on the prosecution's disclosure obligations under s.51A of the Criminal Procedure Code. **This memo does not rely on *Hawkes*.** The proposition is supported instead by *Mohd Ali Jaafar v PP* [1998] 4 MLJ 210, which approves the *R v Sang* discretion and treats the legality of acquisition as distinct from admissibility [S9].

**R7. There is no consolidated reprint of Act 709 published by the Attorney General's Chambers incorporating Act A1727.** The AGC's legislative timeline for Act 709 records the original (2010), a 2016 reprint, 2023 reprints and subsidiary legislation, the October 2024 amendment (A1727) and September 2025 subsidiary legislation — but no post-amendment consolidated reprint of the principal Act [S20]. Where this memo describes the amended text of a section, it does so by reading the original provision together with the amending words in Act A1727, and it says so. **Anyone relying on the amended text should verify against a current consolidated edition.**

**R8. The PDP Regulations 2013 and the PDP Standard are under active revision.** Public Consultation Paper No. 4/2025 proposed amendments to P.U.(A) 335/2013, including alignment with the new provisions and revision of the security, retention and integrity standards [S15]. *Impact: the security requirements in the checklist (§10, section D) should be treated as a moving target and re-verified periodically.*

**R9. The s.45(1) household exemption is not a safe harbour for this product.** It is a purpose test ("only for the purposes of that individual's personal, family or household affairs"), and a work archive fails it. If a future product variant were genuinely personal (a private journal with no work content and no evidential purpose), the analysis would change materially. *Impact: the exemption should not be relied on to justify any work-content ingest.*

**R10. The archive's evidential value is inversely related to its legal defensibility.** The scenarios in which a covert recording would be most useful as evidence (*Justin Maurice Read* type disputes, board meetings, disciplinary hearings) are precisely the scenarios in which the recording is most exposed to PDPA s.7 breach, employment-misconduct characterisation, breach-of-confidence claims, and authenticity challenges. This tension is a product-positioning problem, not a legal one, but it should be surfaced to whoever owns issue #6.

---

## 12. Sources

All sources retrieved on **25 September 2026**.

### Primary legislation (Attorney General's Chambers, Laws of Malaysia)

- **[S1]** *Personal Data Protection Act 2010* (Act 709), original text as enacted — https://lom.agc.gov.my/ilims/upload/portal/akta/outputaktap/Act%20709%20ori.pdf (sections cited: ss.2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 38, 39, 40, 41, 42, 43, 44, 45, 46, 129)
- **[S2]** *Personal Data Protection (Amendment) Act 2024* (Act A1727) — https://lom.agc.gov.my/ilims/upload/portal/akta/outputaktap/2430673_BI/Act%20A1727.pdf
- **[S3]** *P.U.(B) 522 — Personal Data Protection (Amendment) Act 2024: Appointment of Date of Coming into Operation*, dated 19 December 2024, gazetted 24 December 2024 — https://www.pdp.gov.my/ppdpv1/wp-content/uploads/2024/12/PENETAPAN-TARIKH-PERMULAAN-KUAT-KUASA.pdf
- **[S5]** *Personal Data Protection (Class of Data Users) Order 2013* [P.U.(A) 336] — https://lom.agc.gov.my/ilims/upload/portal/akta/outputp/pua_20131114_P.U.%20(A)%20336-PERINTAH%20PERLINDUNGAN%20DATA%20PERIBADI%20(GOLONGAN%20PENGGUNA%20DATA)%202013.pdf
- **[S6]** *Communications and Multimedia Act 1998* (Act 588) — https://lom.agc.gov.my/ilims/upload/portal/akta/LOM/EN/Act%20588.pdf (sections cited: s.6 definitions of "communications", "intercept", "authorized interception", "interception capability"; ss.233, 234, 235, 252, 265)
- **[S7]** *Communications and Multimedia (Amendment) Act 2025* (Act A1743) — https://lom.agc.gov.my/ilims/upload/portal/akta/outputaktap/2669326_BI/A1743%20BI-%20COMMUNICATIONS%20AND%20MULTIMEDIA%20(AMENDMENT)%20ACT%202025.pdf (sections cited: s.6, s.93 amending s.234(3))
- **[S8]** *Evidence Act 1950* (Act 56), Reprint 2017 — https://lom.agc.gov.my/ilims/upload/portal/akta/LOM/EN/reprint%20Act%2056%20(Reprint%202017).pdf (sections cited: s.3, ss.61–66)
- **[S11]** *Official Secrets Act 1972* (Act 88) — https://lom.agc.gov.my/ilims/upload/portal/akta/LOM/EN/Act%2088.pdf (sections cited: s.2 definition of "official secret"; s.8)
- **[S21]** *Personal Data Protection Act 2010* (Act 709), Reprint 14 June 2016 — https://lom.agc.gov.my/ilims/upload/portal/akta/LOM/EN/Act%20709%2014%206%202016.pdf

### Regulator instruments and official pages (Personal Data Protection Department / Commissioner, Malaysia)

- **[S4]** *Personal Data Protection Standard 2015* (Standard Perlindungan Data Peribadi 2015), issued under the Personal Data Protection Regulations 2013 [P.U.(A) 335] — https://www.pdp.gov.my/ppdpv1/wp-content/uploads/2024/07/LatestStandard.pdf (official text is in Bahasa Malaysia; security standard at ss.4–5, retention at s.6, data integrity at s.7)
- **[S12]** *Personal Data Protection Guideline: Data Protection Officer*, PDP, August 2025 — https://www.pdp.gov.my/ppdpv1/wp-content/uploads/2025/08/GP_DPO_ENG.pdf (DPO thresholds at para 4.2; read with Commissioner's Circular No. 1/2025)
- **[S13]** *Personal Data Protection Guideline: Data Breach Notification*, PDP, August 2025 — https://www.pdp.gov.my/ppdpv1/wp-content/uploads/2025/08/GP_DBN_ENG.pdf (72-hour and 7-day timeframes at paras 6.1 and 9.1)
- **[S14]** *Personal Data Protection Guideline: Cross-Border Transfer of Personal Data*, PDP, August 2025 — https://www.pdp.gov.my/ppdpv1/wp-content/uploads/2025/08/BUKU-GARIS-PANDUAN-PEMINDAHAN-DATA-PERIBADI-RENTAS-SEMPADAN-CBPDT-BI.pdf (conditions under ss.129(2) and 129(3))
- **[S15]** *Public Consultation Paper No. 4/2025: Proposed Amendments to the Personal Data Protection Regulations 2013 [P.U.(A) 335/2013]*, PDP, consultation period 22 August – 8 September 2025 — https://www.pdp.gov.my/ppdpv1/wp-content/uploads/2025/08/Kertas_Konsultasi_Awam_Bil_4_2025_PUA335_Final_ENG-1.pdf
- **[S16]** PDP official page, *Application and Non-Application of the Act* — https://www.pdp.gov.my/ppdpv1/en/akta/application-and-non-application-of-the-act/
- **[S17]** PDP official page, *Registration of Data Controller* — https://www.pdp.gov.my/ppdpv1/en/registration-of-data-controller/
- **[S20]** AGC Laws of Malaysia, Act 709 legislative timeline and commencement record (commencement 15 November 2013 by P.U.(B) 464/2013) — https://lom.agc.gov.my/act-detail.php?a=YWN0PTcwOSZsYW5nPUJNfDNiYWIwNTM1ZWIwODQ2MjFjYjYxYmMzYTVjNGU5NGRjYzE0MWE3ZTBjNGQyODEyODkzNWJlNWJlZmQ5Y2MxOWQ=

### Case law

- **[S9]** *Mohd Ali Jaafar v Public Prosecutor* [1998] 4 MLJ 210 (High Court, Melaka; Augustine Paul J) — reported judgment text retrieved via the Internet Archive: https://archive.org/download/mlj_1998_4/MLJ%201998%204%20210_text.pdf. Held: a tape recording is a document within s.3 of the Evidence Act 1950; proof is governed by ss.61–66; the recording must be played in court and the voices and accuracy proved; the court approved the *R v Sang* [1980] AC 402 discretion to exclude strictly admissible evidence where prejudicial effect greatly outweighs evidential value.
- **[S10]** *Lee Ewe Poh v Dr Lim Teik Man & Anor* [2011] 4 CLJ 397 (also reported [2011] 1 MLJ 835) (High Court Malaya, Pulau Pinang; Chew Soo Ho JC; judgment 2 September 2010) — judgment PDF: https://www.dataguidance.com/sites/default/files/lee_ewe_poh.pdf. Held (inter alia): invasion of privacy is actionable under the Malaysian common law, following the Court of Appeal's acceptance of the cause of action in *Maslinda Ishak v Mohd Tahir Osman & Ors* [2009] 6 CLJ 653; alternatively, the three elements of breach of confidence were satisfied.

### Secondary sources (used only where primary text was not retrievable — flagged as such)

- **[S18]** Skrine (Sara Lau), *"Hello? Is This Thing On?"*, December 2019 — https://www.skrine.com/insights/newsletter/december-2019/hello-is-this-thing-on-%E2%80%9D. **Secondary.** Source for the Industrial Court propositions and citations in §6.4: *Sanjungan Sekata Sdn Bhd v Liew Tiam Seng* [2003] 3 ILR 1155; *Yap Fat v Southern Investment Bank Bhd / Southern Bank Berhad* [2010] 3 ILR 350; *Justin Maurice Read v Petroliam Nasional Berhad* [2017] 3 ILR 527; *Izaidin Joinnie v Amanah Saham Sarawak Berhad* [2018] 2 LNS 1787. The primary awards were **not** retrieved.
- **[S19]** Marcus Tan & Co, *Secret Recordings Admissible in Dismissal Cases*, 9 May 2024 — https://mtco.my/secret-recordings-admissible-in-dismissal-cases/. **Secondary.** Source for the citation of *Dr Bernadine Malini Martin v MPH Magazines Sdn Bhd & Ors* [2006] 2 CLJ 1117 and for the *Benjamin William Hawkes v Public Prosecutor* [2020] 5 MLJ 417 proposition that this memo **does not rely on** (see R6).

### Sources sought but not retrieved

- The primary Industrial Court awards in [S18] (Industrial Law Reports / LNS — not freely accessible).
- *Maslinda Ishak v Mohd Tahir Osman & Ors* [2009] 6 CLJ 653 in full — the relevant Court of Appeal passages are quoted at length inside [S10], which is how they are cited here.
- *Dr Bernadine Malini Martin v MPH Magazines Sdn Bhd & Ors* [2006] 2 CLJ 1117 in full.
- *Benjamin William Hawkes v Public Prosecutor* [2020] 5 MLJ 417 in full.
- Any MCMC guideline, circular or determination addressing whether a participant recording their own conversation falls within CMA 1998 s.234 — none was located.
- A consolidated reprint of Act 709 incorporating Act A1727 (see R7).

---

*End of memo. Prepared for GitHub issue #3; feeds issues #6 and #7. Not legal advice.*
