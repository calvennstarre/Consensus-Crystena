---
name: consensus-grant-finder
description: "Consensus-powered NIH grant research skill that requires the Consensus plugin. Use when someone wants to find grants for their research idea. Runs a 5-facet positioning analysis via Consensus (with draft Significance/Innovation language), maps the research to the right NIH institutes and study sections, finds NOSIs and funded overlap, and produces an editable Word document (.docx) with budget/scope guidance, submission timelines, and program officer recommendations. Trigger on: 'find grants for my research idea', 'what grants match my research', 'help me find NIH funding', 'grant opportunities for my research', or any grant-related request where speed and clarity matter."
---

# Consensus Grant Research

## Overview

This skill helps clinical researchers scope the NIH funding landscape for a research idea. It currently focuses on NIH funding — other funders (PCORI, DOD, VA, foundations) are not yet covered. It does three things:
1. **Research positioning analysis** — 5-facet Consensus search that produces gap quotes and draft positioning language for the grant's Significance/Innovation sections
2. **Institute mapping** — identifies which NIH institutes are actually funding this area
3. **Targeted grant discovery** — NOSIs, open FOAs, and funded overlap filtered by the mapped institutes

Output: a Word document (.docx) the researcher can edit, annotate, and build on — not a static PDF.

---

## Required Dependency: Consensus

The **Consensus plugin must be installed, authorized, and added to the current conversation** before the research-positioning workflow can run.

1. Inspect the tools available in the current conversation for the Consensus search tool.
2. If Consensus is installed and callable, use it for every academic search in Phase 2A.
3. If Consensus is unavailable, use ChatGPT's plugin-install request control to prompt the user to install the Consensus plugin and add it to the conversation when that control can resolve Consensus. Otherwise, explain that Consensus is required so this skill can accurately search research articles and reduce hallucinations; ask the user to install, authorize, and add it to the conversation, then pause.
4. If the user says Consensus is already installed but its search tool is still unavailable, explain that they can add it to the conversation by clicking the **+** button in the message bar or by tagging `@Consensus` in a message.
5. Confirm that the Consensus search tool is callable before continuing.

Do not begin the grant analysis or substitute general web search or model knowledge for the Consensus research-positioning searches while the dependency is unavailable.

---

## Agent Integrity Rules

These rules govern every step of this skill. They exist because grant applicants make real decisions based on this output — a fabricated citation or silently skipped search can waste months of effort.

### Execution discipline
- A step is not complete until its result is confirmed received. Never mark a search or API call "done" based on intent — only on a confirmed response.
- Run Consensus searches **sequentially, one at a time**, with a minimum 1-second pause between calls. The Consensus API enforces a 1 query/second rate limit — parallel calls will fail or be throttled. After each search, confirm results were received before proceeding to the next.
- Run RePORTER API calls sequentially as well — wait for each response before sending the next.

### Data sourcing
- Count only what was returned by tool calls in this session. Never supplement with information from training knowledge, prior conversations, or any source other than the current Consensus response, NIH RePORTER response, or official NIH page retrieved during this workflow.
- If a search returns fewer results than expected (e.g., 0–2 papers for a facet), surface that gap explicitly to the researcher. Do not fill it silently with training knowledge.
- If training knowledge is included for any reason (e.g., mechanism budget ranges, submission dates), label it `[Not from Consensus/RePORTER — reference information]` and exclude it from paper counts and citation lists.

### Counts & attribution
- Track three separate numbers per search facet: **queries sent**, **results received** (actually shown/returned in the response), and **results cited** in the final document. Never conflate them.
- Every cited paper must have a retrievable URL (Consensus link or DOI) from this session. No URL = not citable.
- **Detect Consensus plan-tier caps.** Consensus responses include language like "Found 20 papers, showing top 10." Parse this for every search. The number *shown* (not found) is your actual result count — you can only cite what was shown. Typical caps by tier:
  - **Unauthenticated / not logged in**: ~3 results shown
  - **Free plan**: ~10 results shown (out of ~20 found)
  - **Premium plan**: ~20 results shown
  If you detect a cap (e.g., "showing top 10" when 20 were found), log it in the audit log and surface it to the researcher: "Consensus found [N] papers but returned [M] due to plan limits. Citations are drawn only from the [M] shown." This is critical — the researcher needs to know whether sparse results reflect a genuine gap in the literature or just a plan-tier ceiling.
- Report all tool constraints that affect coverage: Consensus result caps per query, RePORTER pagination limits, and any rate-limit-induced gaps. The researcher should understand the ceiling on what this analysis can find.

### Error handling
- On any tool failure (Consensus search error, RePORTER timeout, web fetch failure): wait 3 seconds, retry once, and log the failure.
- After 3 consecutive failures across any combination of tools: stop and alert the researcher before proceeding. Explain what failed and what data will be missing.
- Never silently skip a failed step. If a search couldn't be completed, note it in the output with `[Search failed — not included in analysis]`.

### Transparency & audit log
- Include an **Audit Log** section in the .docx (Section 9, after References) documenting: each tool call made, the query or endpoint used, how many results were returned, and how many were cited. This lets the researcher (and their mentor) understand exactly what the analysis is based on.
- Apply the same sourcing and attribution standards to the chat summary message as to the formal document. Don't cite papers in chat that aren't traceable to a tool response.

---

## Workflow

### Phase 1: Intake

First, ask the researcher to describe their research idea in a few sentences. Before the follow-up questions, set expectations:

> "Just so you know — this analysis focuses on NIH funding (institutes, mechanisms, NOSIs, RePORTER data). If you're also considering PCORI, DOD CDMRP, VA Merit, or foundation grants, I won't cover those here, but I can help you think through them separately."

Then use ChatGPT Work's interactive user-input control (commonly exposed as `request_user_input`) to ask 3 quick questions in one control. Keep each option mutually exclusive, put the recommended or most common option first when context supports one, and retain the control's free-text response for nuance.

**Question 1 — Career stage & mechanism fit**

Ask: "Which best describes your current position?" with these choices:

- **Trainee:** Pre-doctoral or postdoctoral; prioritize F31, F32, and T32 pathways.
- **Early-career faculty:** Pre-tenure with no major independent grant yet; prioritize K01, K08, K23, K99/R00, and R21 pathways.
- **Established investigator:** Mid-career or senior investigator; clarify the exact stage in a conditional follow-up.

If the user selects **Established investigator**, use a second interactive control to distinguish:

- **Mid-career:** Has or had a K award or first R01; consider R01, R21, and R34.
- **Senior investigator:** Consider R01, R35, P01, and U01.

**Question 2 — Preliminary data & prior submissions**

Ask: "Where are you in the process?" with these choices:

- **Starting from scratch:** No preliminary data yet.
- **Pilot evidence available:** Some pilot data or related publications.
- **Resubmission:** Previously submitted; clarify the review outcome in a conditional follow-up.

If the user selects **Resubmission**, ask whether the application received a score or was not scored (triaged). This matters because it changes the strategic recommendation: someone with no preliminary data may need an R21 or pilot-funding path first, while someone resubmitting needs differentiation and reviewer-response guidance.

**Question 3 — Institutional environment**

Ask: "What best describes your research environment?" with these choices:

- **Major research university:** R1 institution with relevant core facilities.
- **Academic medical center:** Teaching hospital or academic health system.
- **Another setting:** Clarify the environment in a conditional follow-up.

If the user selects **Another setting**, ask whether it is a smaller/community-based institution or a VA/military-affiliated environment. This affects which mechanisms are realistic and how to frame the Environment section of a grant.

If ChatGPT Work's interactive control is unavailable, present the same questions as concise numbered choices and wait for the answers. Do not emulate the control with Claude-specific functions or code.

Once you have the research idea + these 3 answers, proceed immediately:

> "Running your analysis now — novelty search, institute mapping, and grant matching."

---

### Phase 2A: Research Positioning Analysis (5 Consensus searches)

The goal of this phase is NOT to answer "is this novel?" — it's to produce **positioning language** the researcher can use in their grant application. The output should read like a draft of their Significance and Innovation sections: "The field has established X and Y, but no one has done Z in this population — that's the gap this project fills."

Run 5 searches using `Consensus:search` **sequentially** (one at a time, with at least a 1-second pause between each call). After each search, confirm results were received and log the count before moving to the next. If a search fails, wait 3 seconds, retry once, and log the outcome. Each search builds a different part of the positioning argument.

Call the Consensus search tool with the required `query` parameter. Use only these supported optional filters when they improve the search:

| Parameter | Type | Purpose |
|---|---|---|
| `query` | string, required | The research question or topic, using specific academic terminology. |
| `year_min` | integer | Exclude papers published before this year. |
| `year_max` | integer | Exclude papers published after this year. |
| `study_types` | string array | Filter using `rct`, `meta-analysis`, `systematic review`, `literature review`, `case report`, `non-rct experimental`, `non-rct observational study`, `non-rct in vitro`, or `animal`. |
| `sjr_max` | integer | Set the maximum SCImago quartile, from `1` for Q1 through `4` for Q4. |
| `human` | boolean | Set to `true` for human-only evidence. |
| `sample_size_min` | integer | Exclude studies below a minimum sample size. |
| `medical_mode` | boolean | Set to `true` for clinical, medical, or evidence-based medicine questions. |
| `exclude_preprints` | boolean | Set to `true` when only peer-reviewed papers should be returned. |
| `duration_min` | integer | Minimum study duration in days. |
| `duration_max` | integer | Maximum study duration in days. |

Translate a user-specified publication window into `year_min` and/or `year_max`. These filters use publication-year precision, so do not imply a month-or-day cutoff. Record all filters in the audit log exactly as sent.

**Search 1 — What's established (the foundation)**
What does the field already agree on? This becomes the "what is known" setup in a Significance section.
```
query: "<the core research concept as a focused academic query>"
year_min: 2019
```

**Search 2 — The problem's importance (the stakes)**
Why does this matter? Prevalence, burden, outcomes, disparities. This is the "why now" and "why this matters" argument.
```
query: "<disease or condition prevalence outcomes burden in [target population]>"
year_min: 2019
```

**Search 3 — Current approaches and their limits (the competition)**
What interventions, methods, or solutions exist today? Understanding current practice reveals what's insufficient — which is the researcher's opening.
```
query: "<current clinical management interventions guidelines for [condition or domain]>"
year_min: 2019
```

**Search 4 — The proposed method in adjacent contexts (the innovation angle)**
Has anyone applied this specific approach elsewhere? If yes, that's validation. If no, that's novelty. Either way, it's useful positioning.
```
query: "<methodological approach or technique applied to [similar problems or populations]>"
year_min: 2019
```

**Search 5 — What researchers say is missing (the gap — most important)**
This is the highest-value search. Explicitly look for language where researchers call out gaps, limitations, and future directions. These quotes become the backbone of the positioning argument.
```
query: "<research gaps limitations unmet needs future directions in [the field]>"
year_min: 2019
```

**After all 5 complete, verify and then build the positioning brief:**

Before synthesizing, verify completeness. For each of the 5 searches, confirm you have a response with a result count. If any search returned 0 results or failed, note it explicitly — this is a gap the researcher needs to know about. Initialize a running audit log tracking: search facet, query used, results received, and results that will be cited.

The synthesis is the most important output of this entire skill. Don't just list counts — build an argument.

1. **Deduplicate** papers across all 5 searches by title/DOI. Papers found in multiple facets are high-signal. Record the pre- and post-deduplication counts.

2. **Extract gap quotes first** — scan abstracts for language like "future research should", "remains unclear", "no studies have", "limited evidence", "further investigation needed", "critical gap". Pull 3–5 of these with full citations. These are the centerpiece.

3. **Write a positioning narrative** (2–3 paragraphs) that the researcher could adapt for their Significance/Innovation sections:
   - Paragraph 1: "The field has established [X, Y, Z — from Searches 1-2]. The burden is significant because [stakes from Search 2]."
   - Paragraph 2: "Current approaches include [Search 3 findings], but they fall short because [gap quotes from Search 5]. Specifically, [quote 1] and [quote 2]."
   - Paragraph 3: "This project proposes [researcher's idea]. [Search 4 findings] suggest this approach is feasible, and the gap identified above shows it has not been applied to [this population/context]. This represents a [novel/underexplored] intersection."

4. **Build a supporting evidence table** — this is secondary to the narrative above:
   - Facet | Query Used | Results Received | Results Cited | Coverage Level (Minimal/Moderate/Substantial) | Key takeaway for positioning
   - If any facet received 0 results, include it in the table with a note: "No results returned — gap in coverage"

5. **Build the master paper list** with: title, authors, year, journal, citation_count, url — this feeds the bibliography

#### Evidence-depth gate

For each Consensus facet, identify at least 3 directly relevant papers when that many are available. Across all five facets, target 15–20 unique cited papers when the returned evidence supports that depth. Relevance takes priority over a numeric target: never add a marginal paper merely to increase the bibliography. If a facet has fewer than 3 directly relevant papers, state that explicitly in the supporting evidence table and audit log.

Gap quotations must be 3–5 complete, meaningful excerpts with named author/year attribution and a working Consensus hyperlink. Do not reduce them to unexplained sentence fragments. Quote only the language needed to establish the gap, and do not quote more than 25 words from any one paper; paraphrase the surrounding context.

Before leaving Phase 2A, verify that every paper selected for citation has a title, author list, year, journal, and retrievable Consensus URL or DOI from this session. Papers missing any required citation field may inform the coverage count but must not be cited as verified sources.

---

### Phase 2B: Institute Mapping + Grant Discovery (run after 2A)

#### Step 1: RePORTER funded project search

**IMPORTANT: The NIH RePORTER API requires POST requests. Use ChatGPT Work's available command-execution tool with `curl`, or another available tool that supports JSON POST requests. Do not use a GET-only web lookup for this API.**

Run two searches through the command-execution tool — narrow (AND) and broad (OR):

Compute the fiscal year window dynamically: use the current year and the 3 preceding years (e.g., if running in 2026, use `[2023, 2024, 2025, 2026]`). This keeps results current without manual updates.

```bash
# Narrow search — 3-4 core terms with AND
CURRENT_YEAR=$(date +%Y)
FY_RANGE="[$((CURRENT_YEAR-3)), $((CURRENT_YEAR-2)), $((CURRENT_YEAR-1)), $CURRENT_YEAR]"

curl -s -X POST "https://api.reporter.nih.gov/v2/projects/search" \
  -H "Content-Type: application/json" \
  -d '{
    "criteria": {
      "advanced_text_search": {
        "operator": "and",
        "search_field": "projecttitle,terms,abstract",
        "search_text": "NARROW KEYWORDS"
      },
      "fiscal_years": '"$FY_RANGE"',
      "include_active_projects": true
    },
    "limit": 20,
    "offset": 0,
    "include_fields": ["ProjectTitle","AbstractText","ActivityCode","AwardAmount","FiscalYear","PrincipalInvestigators","Organization","OpportunityNumber","AgencyIcAdmin","ProjectNum","ProjectNumSplit","StudySection"]
  }'

# Broad search — 5-6 related terms with OR
curl -s -X POST "https://api.reporter.nih.gov/v2/projects/search" \
  -H "Content-Type: application/json" \
  -d '{
    "criteria": {
      "advanced_text_search": {
        "operator": "or",
        "search_field": "projecttitle,terms",
        "search_text": "BROADER KEYWORDS"
      },
      "fiscal_years": '"$FY_RANGE"',
      "include_active_projects": true
    },
    "limit": 20,
    "offset": 0,
    "include_fields": ["ProjectTitle","AbstractText","ActivityCode","AwardAmount","FiscalYear","PrincipalInvestigators","Organization","OpportunityNumber","AgencyIcAdmin","ProjectNum","ProjectNumSplit","StudySection"]
  }'
```

Run the narrow search first. Confirm a valid JSON response before running the broad search. If either call fails, wait 3 seconds and retry once. Log both outcomes (success with result count, or failure).

Parse both responses with the available scripting environment, combine results, and deduplicate by `project_num`. Record: narrow results received, broad results received, total after dedup.

#### RePORTER search-quality gate

A valid JSON response is not automatically a usable portfolio. Review the returned project titles and abstracts for substantive relevance to the researcher's population, method, clinical setting, intervention, or intended outcome.

- If fewer than half of the returned projects are substantively relevant, or the broad query produces an obviously unfocused portfolio, do not use that response for institute or study-section ranking.
- Refine the keywords and run up to 2 additional targeted RePORTER searches, favoring focused AND combinations or closely related concept pairs.
- Confirm each refinement returned valid JSON, log its query and counts, and include it in the audit log.
- Combine and deduplicate only substantively relevant projects for the final portfolio analysis. Report excluded-query totals separately so a large irrelevant result set is not mistaken for strong funding activity.
- If no focused search produces at least 5 relevant projects, proceed with the sparse portfolio but label all institute and study-section conclusions as preliminary.

#### Step 2: Institute Mapping

From the deduplicated relevant projects, produce two separate institute rankings:

1. **Portfolio-volume ranking** — tally `agency_ic_admin.abbreviation` to show which institutes have the most relevant projects.
2. **Design-and-mission-fit ranking** — assess which institutes fund projects most similar in population, methodology, clinical setting, and intended outcome.

Do not automatically recommend the institute with the largest raw count. A high-volume institute may be funding basic science while a lower-volume institute funds the clinical or methodological design the researcher is proposing. Base the primary recommendation on design and mission fit, report the volume ranking separately, and explain any difference between them.

Example:

> "NIAID has the largest raw HSV portfolio, but most projects are basic virology or therapeutic development. NICHD has fewer awards but the closest methodological overlap with multicenter pediatric clinical prediction research; therefore NICHD is the stronger primary fit and NIAID is a plausible secondary assignment."

This distinction is one of the most actionable findings for the researcher — surface it clearly so they know where to direct the application and why.

#### Step 3: Study Section Mapping

From the combined project results, tally `study_section` values. This tells the researcher which review panels are evaluating work in their space — a critical strategic decision that most applicants don't think about systematically.

For the top 2–3 study sections by count, include:
- Study section abbreviation and full name
- Number of projects from the results reviewed there
- A note on fit: "This panel has a track record of reviewing [type of work] — your application would be in familiar territory here."

If study section data is missing or sparse in the results, note that honestly rather than guessing. The researcher can verify assignments via the CSR study section roster pages at `https://public.csr.nih.gov/studysections`.

#### Step 4: NOSI Detection

From the project results, extract all unique `opportunity_number` values. Filter for any that start with `NOT-` — these are Notices of Special Interest. For each NOSI found:

```
GET https://grants.nih.gov/grants/guide/notice-files/{NOSI_NUMBER}.html
```

Open each NOSI from the official NIH Guide page one at a time using ChatGPT's public web lookup or command-execution tool. Parse the response for:
- Notice title (in `<h1>` or `<title>` tag)
- Institute (paragraph text near top)
- Purpose/scope (first substantive paragraph)
- Expiration or deadline if present

If a NOSI fetch fails, log it as `[NOSI {number} — fetch failed, not included]` and move on. If no NOSIs are found in project results, note this honestly — don't fabricate.

#### Step 5: Grant mechanism matching (career stage + project scope)

From the funded project results, extract unique `opportunity_number` values that are FOAs (start with `PA-`, `PAR-`, `PAS-`, or `RFA-`).

Mechanism selection isn't just about career stage — it's about whether the *project scope* fits the mechanism. Use both career stage and project characteristics to recommend mechanisms.

**Mechanism reference (career stage + budget + scope):**

Treat these as planning ranges. Verify the budget, duration, eligibility, and required preliminary data for every recommended opportunity against its current official NIH announcement before presenting it as active guidance.

| Mechanism | Typical budget | Duration | Best for | Prelim data needed? |
|-----------|---------------|----------|----------|-------------------|
| F31/F32 | Stipend + tuition/fees | 1–3 yrs | Trainees (pre-doc / postdoc) | Some |
| T32 | Institutional training grant | 2–5 yrs | Trainees via mentor's program | N/A |
| R03 | ~$50K/yr direct | 2 yrs | Small, self-contained projects; pilot studies | Minimal |
| R21 | ~$275K total direct | 2 yrs | Exploratory/developmental; new directions | Minimal to moderate |
| K01/K08/K23 | $100–250K/yr (includes salary) | 3–5 yrs | Early-career mentored development | Moderate |
| K99/R00 | K99: ~$100K/yr; R00: ~$250K/yr | 2+3 yrs | Postdoc → faculty transition | Moderate |
| R01 | ~$250K+/yr modular direct | 3–5 yrs | Full research project, the workhorse | Strong |
| R34 | ~$450K total direct | 3 yrs | Clinical trial planning (no execution) | Moderate |
| R61/R33 | Phased, varies | 2+3 yrs | High-risk ideas with milestone gates | Moderate for R61 |
| R35 | ~$750K/yr direct | 7 yrs | Outstanding investigator, broad program | Extensive track record |
| P01 | Multi-component, $1M+/yr | 5 yrs | Multi-PI program projects | Extensive |
| U01 | Varies, often $500K+/yr | 3–5 yrs | Cooperative agreements with NIH involvement | Strong |
| DP1/DP2 | ~$500K/yr direct | 5 yrs | Director's awards for bold, high-risk ideas | Varies |

**How to match:**
- Filter first by career stage (matching intake categories)
- Then filter by project scope: Does the researcher's idea fit in a 2-year $275K R21, or does it need a 5-year R01? If they described a large multi-site study, an R21 won't cover it. If they described a pilot, an R01 is overkill.
- Consider preliminary data from intake: "starting from scratch" → R21, R03, or R61. "Have pilot data" → R01 may be realistic. "Resubmission" → match the original mechanism unless the score suggests a mismatch.

For the top 3 FOAs, construct the link:
- PA/PAR/PAS types: `https://grants.nih.gov/grants/guide/pa-files/{FOA}.html`
- RFA types: `https://grants.nih.gov/grants/guide/rfa-files/{FOA}.html`

---

### Phase 3: Generate the DOCX

#### Report-fidelity contract — hard pass/fail requirement

The built-in documents workflow is the construction, formatting, rendering, and visual-QA mechanism. It must not summarize, merge, renumber, or omit the required report components below in the name of brevity or generic document design.

Create all 9 numbered Heading 1 sections exactly and in this order:

1. Executive Summary
2. Your Positioning in the Field
3. Where NIH Is Funding This Work
4. Recommended Funding Opportunities
5. Similar Funded Research
6. Where This Work Gets Reviewed
7. Strategic Recommendations & Next Steps
8. References
9. Data Sources & Methodology (Audit Log)

Missing or sparse data never authorizes removing a required section. Retain the section, explain the limitation, and place `Not returned`, `Insufficient data`, or `No verified match found` in the appropriate table. Do not renumber later sections. If study-section data is unavailable, Section 6 must still appear and must explain that no empirical panel ranking can be made from the returned records.

Do not generate the report unless all 5 required Consensus searches have returned responses and are present in the running audit log. Do not deliver the report unless it passes the pre-delivery completeness check later in this phase.

Use ChatGPT's built-in **documents workflow** to create the `.docx`. Invoke and follow the available document-creation skill or workflow, including its render-and-verify process. Do not hard-code an internal skill path, install a document library, or require custom JavaScript unless the built-in documents workflow explicitly calls for it. The output must remain an editable Word document so the researcher can copy sections into grant applications and add notes.

#### Report structure

**Section 1 — Executive Summary**
- Title: "Grant Research Overview: [research topic]"
- Date, career stage, institutional environment
- Executive Summary: 3–4 bullet points covering novelty level, top institute, top grant, key differentiator

**Section 2 — Research Positioning**
- Heading 1: "2. Your Positioning in the Field"
- Heading 2: "The gap, in researchers' own words" — lead with 3–5 italicized, attributed gap excerpts with inline citations hyperlinked to Consensus. These are the most valuable part of the document because they give the researcher citable evidence that the gap is recognized.
- Heading 2: "Positioning narrative (draft language you can adapt)" — write 2–3 substantive paragraphs in a tone the researcher can adapt directly into Significance and Innovation. Use phrases such as "The field has established...", "Current approaches fall short because...", and "This project addresses an unmet need by...".
- Heading 2: "Supporting evidence table" — include all five facets, even if one returned no results. This table supports the narrative rather than replacing it.

**Section 3 — Target Institutes**
- Heading: "Where NIH Is Funding This Work"
- Institute ranking table showing the relevant-project count and design/mission fit for each institute.
- A 2–3 paragraph interpretation that distinguishes raw portfolio volume from methodological and mission fit. Explain any case where the recommended institute is not the raw-count leader.

**Section 4 — Grant Opportunities**
- Heading: "Recommended Funding Opportunities"
- If NOSIs found: bold callout at top — "Active Notice of Special Interest: [title] ([number]) — [1-sentence summary]"
- Top 3 grants table using the exact funding-opportunities schema below; hyperlink every FOA number to its verified official NIH page.
- For each recommended grant, include a short paragraph: why this mechanism fits the *project scope* (not just career stage), what the budget allows, and whether the researcher's preliminary data situation is a good match. E.g., "An R21 ($275K over 2 years) would let you establish [specific aim] as proof-of-concept before pursuing a full R01. Your pilot data on [X] is sufficient for this mechanism."

**Section 5 — Funded Overlap**
- Heading: "Similar Funded Research"
- Top 5 projects table using the exact similar-funded-projects schema below; include both the award/fiscal-year field and a dedicated verified NIH RePORTER link.
- Differentiation paragraph: "Your idea differs from existing funded work in [2–3 specific ways]"

**Section 6 — Study Sections**
- Heading: "Where This Work Gets Reviewed"
- Table: Study Section | Full Name | Projects Found | Fit Notes
- A 1–2 paragraph interpretation on which panel is the best match and why. If the returned records lack study-section data, retain the section and table, state the limitation, and recommend verification with the program officer and CSR directory without inventing a panel assignment.

**Section 7 — Strategic Recommendations & Next Steps**
- 4–6 recommendations using a real numbered list, not bullets (e.g., "Target NIAMS with an R21 to establish preliminary data before an R01 submission")
- **Always include a program officer recommendation.** This is the single most valuable piece of advice for any grant applicant. Format it as: "Contact the program officer at [institute] before submitting. POs can confirm your idea fits the institute's priorities, suggest the right FOA, and flag potential issues. Find staff contacts at https://www.nih.gov/institutes-nih/list-nih-institutes-centers-offices → [specific institute] → 'Staff' or 'Program Officers' page."
- If the researcher indicated "resubmission" in intake, include guidance on how to frame the response to reviewers and what changed
- **Include a submission timeline note** based on the recommended mechanism. Before using the reference dates below, verify the current schedule on an official NIH submission-policy page. If the live NIH schedule differs, use and link the live schedule rather than the table:

  | Mechanism | Standard receipt dates |
  |-----------|----------------------|
  | R01, R21, R03 | Feb 5, Jun 5, Oct 5 |
  | K awards (K01, K08, K23, K99) | Feb 12, Jun 12, Oct 12 |
  | R34, R61/R33 | Feb 16, Jun 16, Oct 16 |
  | F31, F32 | Apr 8, Aug 8, Dec 8 |

  Note which upcoming cycle is realistic given how much work remains (e.g., "If you have pilot data ready, the October cycle is feasible. If you still need preliminary results, targeting February gives you more runway.")
- One-paragraph closing on positioning

**Section 8 — References**
- Numbered list of all papers cited anywhere in the report
- Format: `[N] Authors (Year). Title. Journal.` with title hyperlinked to its Consensus URL
- Group by section if more than 15 papers: "Novelty Analysis Sources", "Funded Project Sources"
- Include every paper referenced by an inline citation — no citations without a bibliography entry

**Section 9 — Audit Log**
- Heading 1: "9. Data Sources & Methodology (Audit Log)"
- This section provides full transparency on what tools were called, what was returned, and what made it into the report. It helps the researcher and their mentor assess the completeness of the analysis.
- Use these Heading 2 subsections in this order: "Consensus searches (Phase 2A)", "RePORTER searches (Phase 2B)", "NOSI detection", "Tool constraints", and "Failed / retried steps".
- **Consensus searches table**: use the exact Consensus-audit schema below.
- **Consensus plan-tier note**: If any search response indicates a cap (e.g., "Found 20 papers, showing top 10"), call this out explicitly: "Consensus plan limit detected: [M] of [N] papers returned per query. Only the [M] shown papers are citable. A higher-tier plan would surface more results." If all found papers were shown, note that too — it confirms full coverage.
- **RePORTER searches table**: use the exact RePORTER-audit schema below and include every refinement query.
- **NOSI fetches table** (if applicable): NOSI Number | Fetch Status | Included in Report (Yes/No)
- **Summary stats**: Total unique papers shown (post-dedup), total cited, total RePORTER projects found, total cited. Distinguish "found by Consensus" from "shown to us" from "cited in this report."
- **Tool constraints note**: "Consensus returned up to [M] results per query (plan limit: [tier if detectable]). RePORTER queries were limited to 20 results each. These caps mean this analysis covers the most relevant results, not an exhaustive literature search. If results seem sparse for a facet, it may reflect plan limits rather than a genuine gap in the literature."
- **Failed steps** (if any): List any searches or fetches that failed and were not recoverable, so the researcher knows where coverage is incomplete

#### Required table schemas

Use these exact columns. Do not combine, rename, or remove columns to save space:

- **Supporting evidence:** Facet | Query Used | Results Received | Results Cited | Coverage Level | Key Takeaway
- **Institute ranking:** Institute | Full Name | Relevant Projects Found | Design and Mission Fit
- **Funding opportunities:** Mechanism | FOA Number | Institute | Budget & Duration | Fit for the Researcher
- **Similar funded projects:** Project Title | PI | Institution | Award / Fiscal Year | Link
- **Study sections:** Study Section | Full Name | Projects Found | Fit Notes
- **Submission timeline:** Mechanism | Verified Standard Receipt Dates
- **Consensus audit:** Search # | Facet | Query Used | Papers Found | Papers Shown | Papers Cited | Status
- **RePORTER audit:** Search Type | Keywords | Total / Returned | Relevant Projects Used | Status

If a required dataset is unavailable, keep the table and insert a single explanatory row. A limitation note does not replace the required table.

#### Link construction

Use the built-in documents workflow to render every citation and grant reference as a clickable hyperlink:

- Paper citations link to the full Consensus URL returned during this session.
- FOA and NOSI references link to their official NIH Guide pages.
- Funded projects link to `https://reporter.nih.gov/project-details/{project_num}`.

Never display a constructed link as verified unless the corresponding identifier was returned by a current tool call and the target format is valid.

#### Layout contract

Use a continuous, information-dense research-report layout rather than a standalone cover page. Apply these specifications deliberately instead of relying on Word defaults:

- US Letter portrait with 0.75-inch margins
- Arial throughout; 10pt body text; 16pt title; 11pt Heading 1; 10pt Heading 2; 8.5–9pt table text
- Compact but readable paragraph rhythm: approximately 3–5pt after body paragraphs and 6–8pt before major headings
- Put the title, descriptive subtitle, and a compact metadata table at the top of page 1; do not consume a page with decorative cover content
- Navy (`#1a3a5c`) headings; light blue (`#e8f0f8`) table headers; alternating white and `#f5f8fc` rows
- Blue clickable hyperlinks for citations, FOAs, NIH pages, and funded projects
- A restrained footer containing the abbreviated topic, report date, and page number
- NOSI callout, when present: bold text with a restrained amber highlight
- Keep headings with the following paragraph or table; avoid orphaned headings and split table headers
- Target 8–10 pages when the returned evidence supports that depth. Do not pad sparse evidence, but do not compress or omit required sections merely to reduce the page count.

#### Pre-delivery completeness check

After generating the DOCX, inspect its structure programmatically using the available document or OOXML tooling, then render it to page images and inspect every page. If programmatic inspection is unavailable, reopen the generated document and perform the same structural checks manually; do not claim that validation was automated.

Fail the report, revise it, and repeat validation if any of the following is true:

- Fewer than 9 numbered Heading 1 sections exist, the headings are out of order, or section numbering is discontinuous
- Any required Heading 2 subsection is missing
- Section 6 was removed because study-section data was sparse or unavailable
- A required table is absent, or its column headings were renamed, combined, or removed
- Section 2 does not include 3–5 attributed gap excerpts, unless the evidence-depth limitation is stated explicitly
- The positioning narrative is shorter than 2 substantive paragraphs without a documented evidence limitation
- Recommendations are formatted as bullets rather than a real numbered list
- The verified submission-timeline table is missing
- The audit log does not distinguish papers found, papers shown, and papers cited
- An inline citation lacks a matching bibliography entry, or a cited bibliography entry lacks a retrievable hyperlink from this session
- An FOA, NOSI, official NIH reference, or funded project presented as verified lacks its required hyperlink
- The document is shorter than 7 pages even though sufficient evidence was returned to satisfy the evidence-depth gate
- The rendered pages contain clipping, overflow, broken tables, unreadable hyperlinks, excessive whitespace, or orphaned headings

Record the check internally. Do not deliver the document until every applicable item passes. If a check cannot pass because source data were not returned, preserve the relevant section and describe the limitation rather than silently omitting content.

---

### Phase 4: Deliver

Name the document `grant-research-overview.docx` unless a topic-specific filename would be clearer. Make it available through ChatGPT's file-delivery workflow.

Close with a brief summary:

> "Here's your grant research overview as a Word doc — you can edit it, pull sections into your application, or share it with your mentor. Key findings:
> - **Positioning:** [1 sentence on the gap and how the researcher's idea fills it]
> - **Best-fit institute:** [institute] — [N] active projects in your area
> - **Top grant:** [FOA] ([mechanism]) — [1 sentence why]
> - **Recommended study section:** [abbreviation] — [1 sentence on fit]
> - **Next step:** Contact the program officer at [institute] before submitting
> - [If NOSI found]: ⚠️ Active NOSI: [number] — deadline [date if found]
>
> Clickable links throughout — citations go to Consensus, FOAs go to NIH Guide, and funded projects go to NIH Reporter.
>
> This is a starting point, not a final product. As you refine your specific aims, come back and run this again — the positioning narrative will sharpen as your idea gets more specific."

---

## Notes

- **NIH RePORTER API requires POST** — use ChatGPT Work's available command-execution tool with `curl`, or another available JSON POST-capable tool: `https://api.reporter.nih.gov/v2/projects/search` — no authentication required.
- **DOCX generation uses ChatGPT's built-in documents workflow** — render and inspect every page before delivery.
- **Consensus rate limit**: 1 query per second. Run searches sequentially with a `sleep 1` or equivalent pause between calls. Never run Consensus searches in parallel.
- **Consensus plan-tier detection**: After each search, parse the response for language like "Found X papers, showing top Y." The *shown* count is what you can actually cite. Typical caps: unauthenticated ~3, free ~10, premium ~20. If you see a cap, tell the researcher — sparse results on a facet might be a plan limit, not a literature gap. Log the detected tier in the audit log.
- NOSI files are available from the official NIH Guide at `https://grants.nih.gov/grants/guide/notice-files/{number}.html`; retrieve them using ChatGPT's public web lookup or command-execution tool.
- If RePORTER returns fewer than 5 results, the broad OR search should compensate — combine and deduplicate both. Surface the low count to the researcher.
- If no NOSIs surface from project results, omit only the NOSI callout or NOSI-specific table; retain the required grant-opportunities and audit sections and state that no verified NOSI was found
- Career stage filtering is a signal, not a hard filter — mention if a grant is a stretch for their stage
- The transparency about *which institute* and *why* is the most actionable part of the report — don't rush past it
- Every inline citation `[N]` must have a corresponding bibliography entry — track all cited papers from the start. No URL from this session = not citable.
- Keep the document focused and actionable while preserving every required section. Target 8–10 pages including bibliography and audit log when the returned evidence supports that depth; do not add filler or expand into full appendices.
- **Never pad results with training knowledge.** If a search returns sparse results, say so. If you include reference information (e.g., mechanism budgets, submission dates), label it as such — it's not from a tool call in this session.
