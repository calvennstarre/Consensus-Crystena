---
name: recommended-reading-list
description: "Generate a curated supplementary reading list from a syllabus using Consensus academic search. Hard requirement: before parsing, web search, analysis, or document creation, verify that a callable Consensus scholarly search tool is attached to the current conversation; if unavailable, stop and help the user add @Consensus. Never replace the required Consensus paper searches with web search or model knowledge. Use for uploaded syllabi, course outlines, curricula, supplementary readings, recent course research, or requests to update a syllabus with papers. Produces an editable Word reading list with verified Consensus links, plain-language summaries, discussion questions, and a search audit log."
---

# Recommended Reading List Generator

> **STOP:** Do not parse the syllabus, search the web, analyze papers, or generate a document until the Phase 0 Consensus preflight passes.

## Phase 0: Consensus Preflight — Hard Stop

This gate takes precedence over every other instruction in this skill.

### Availability test

Inspect the tools callable in the current conversation. Consensus is available only when a tool whose name or description identifies it as the **Consensus scholarly paper search** is present and callable.

These do **not** prove availability:

- The user attached this `SKILL.md` or `.skill` file.
- Consensus is installed or authorized elsewhere in ChatGPT.
- The user says Consensus has been installed.
- The conversation merely mentions `@Consensus`.

### If Consensus is unavailable

The only permitted actions are checking available tools, invoking ChatGPT's plugin-install request control when it can resolve Consensus, and telling the user how to add it. Do **not** parse the syllabus, search the web, use model knowledge to recommend papers, or create a document.

Send this message, then **end the turn**:

> Consensus is required to find and verify the research articles in this reading list so the workflow can reduce citation errors and hallucinations. Please install and authorize Consensus, then add it to this conversation using the **+** button in the message bar or by tagging `@Consensus`. Once it is attached, say **“Continue.”**

If ChatGPT exposes a plugin-install request control that can resolve Consensus, use it before sending the message. When the user says “Continue,” repeat the availability test; do not rely on their confirmation alone.

### If Consensus is available

Proceed to Phase 1. After parsing and user confirmation, the first external research action must be a Consensus search. Never use web search or model knowledge as a substitute for paper discovery.

## Overview

This skill takes a course syllabus (PDF, DOCX, or text) and produces a professional Word document containing a curated reading list of recent peer-reviewed research. It uses the Consensus scholarly search tool to find papers that map to each course topic. The output includes clickable links to each paper on Consensus, a one-sentence plain-language summary, and a thought-provoking discussion question that ties the paper back to the course's learning goals.

The instructor or student walks away with a ready-to-distribute .docx that supplements their textbook with the latest research.

## Data Integrity Principles

This skill calls an external search tool (Consensus) and builds a document from the results. The user is trusting that every paper in the final reading list actually came from Consensus in this session — not from your training data, not from a prior conversation, not hallucinated. This matters because fabricated citations in academic contexts can damage an instructor's credibility and mislead students.

To earn that trust, follow these principles throughout:

**Only use what Consensus returns.** Every paper title, author list, journal name, year, and URL in the final document must come from a Consensus search result received during this session. If you know of a great paper from your training data but Consensus didn't return it, do not include it. If you want to mention it conversationally, label it clearly: `[Not from Consensus — model knowledge]`.

**Confirm before moving on.** A search is not complete until you've seen the results come back. Don't mark a section as "done" based on having sent the query — wait for the response, inspect the papers returned, and then proceed to the next query.

**Track your numbers.** Throughout the search phase, maintain a running tally of three separate counts:
- **Queries sent**: how many Consensus searches you executed
- **Papers received**: how many total papers Consensus returned across all searches
- **Papers cited**: how many papers you selected for the final document

These numbers will appear in the audit log (see Phase 4). They help the user understand coverage — for instance, if the free tier returned 3 papers per query and you ran 12 queries, the user knows you worked with a pool of ~36 candidates.

**Surface gaps, don't fill them silently.** If a search returns zero results, or fewer results than expected, tell the user. Don't quietly substitute model knowledge. A section with one paper and a note saying "Consensus returned limited results for this topic — consider broadening the search or checking manually" is far more honest than padding with fabricated entries.

## Workflow

### Phase 1: Parse the Syllabus

Read the uploaded syllabus file and extract two things:

1. **Course topics** — the ordered list of subjects/units covered (e.g., "Protein Structure", "Enzyme Kinetics", "Lipid Metabolism"). These become the sections of the reading list.
2. **Learning outcomes** — the course-level goals (e.g., "Describe the central pathways that provide living organisms with energy"). These inform the discussion questions you'll write for each paper.

If the syllabus doesn't have explicit learning outcomes, infer 3-5 from the course description and topic list.

Group closely related topics together into sections (e.g., "Protein Structure" and "Protein Function" can become one section). Aim for 6-12 sections — enough granularity to be useful, but not so many that the document feels fragmented. Present this grouping to the user and confirm before searching.

Use ChatGPT Work's interactive user-input control (commonly exposed as `request_user_input`) for confirmation. Ask a **Section plan** question with these choices:

- **Proceed as proposed (Recommended):** Search the displayed sections and learning outcomes.
- **Adjust the grouping:** Let the user add, remove, merge, split, or rename sections in free text.
- **Re-extract the syllabus:** Re-read the source because topics or outcomes appear incomplete.

If the control is unavailable, present the same options as a numbered list and wait. Do not emulate the control with Claude-specific prompt functions or code.

### Phase 2: Search Consensus for Each Section

For each section, craft 1-2 targeted search queries and run them using the Consensus MCP search tool. The goal is to find papers that are recent (set `year_min` to the current year minus 1), relevant to the topic, and ideally connected to real-world applications (food science, nutrition, health, disease — whatever the course emphasizes).

Before searching, record the final query plan and set `planned_queries` to its total query count. Phase 2 is complete only after every planned query has produced an actual Consensus response and appears in the audit log. A zero-result response counts as completed; an intended call, web result, model-generated citation, or failed call does not.

- The first external research action must be the first planned Consensus query.
- Do not use web search, browser tools, or another scholarly provider for paper discovery.
- If a Consensus call fails, follow the retry rules below. If the retry also fails, pause the workflow and tell the user which required section search remains incomplete. Do not replace it with web search or model knowledge.
- Do not write paper summaries or begin document generation until the audit log contains `planned_queries` completed Consensus responses.

Call the Consensus search tool with the required `query` parameter. Use only these supported optional filters when they improve the search:

| Parameter | Type | Purpose |
|---|---|---|
| `query` | string, required | The research topic, using specific academic terminology. |
| `year_min` | integer | Exclude papers published before this year. |
| `year_max` | integer | Exclude papers published after this year. |
| `study_types` | string array | Filter using `rct`, `meta-analysis`, `systematic review`, `literature review`, `case report`, `non-rct experimental`, `non-rct observational study`, `non-rct in vitro`, or `animal`. |
| `sjr_max` | integer | Set the maximum SCImago quartile, from `1` for Q1 through `4` for Q4. |
| `human` | boolean | Set to `true` for human-only evidence. |
| `sample_size_min` | integer | Exclude studies below a minimum sample size. |
| `medical_mode` | boolean | Set to `true` for clinical, medical, or evidence-based medicine topics. |
| `exclude_preprints` | boolean | Set to `true` when only peer-reviewed papers should be returned. |
| `duration_min` | integer | Minimum study duration in days. |
| `duration_max` | integer | Maximum study duration in days. |

Translate a user-specified publication window into `year_min` and/or `year_max`. These filters use publication-year precision, so do not imply a month-or-day cutoff. Record every filter exactly as sent.

**Query design tips:**
- Include the core topic plus an applied angle. For example, for a biochemistry course with a nutrition focus, don't just search "enzyme kinetics" — search "enzyme kinetics food processing applications" or "enzyme kinetics nutrition digestion".
- If the course has an obvious applied domain (food science, medicine, environmental science, etc.), weave that into every query. This surfaces papers that bridge theory and application, which makes for much better supplementary reading.
- Keep queries to 4-8 words. Overly long queries dilute relevance.

**Example queries for a Biochemistry & Nutrition course:**

| Section | Query |
|---------|-------|
| Nucleic Acids | `nucleic acid structure function food safety` |
| Amino Acids | `amino acid nutrition metabolism health` |
| Protein Structure & Function | `protein structure function food science` |
| Enzyme Function & Kinetics | `enzyme kinetics food processing applications` |
| Lipids & Membranes | `lipids biological membranes lipoproteins nutrition` |
| Carbohydrate Metabolism | `carbohydrate metabolism glycolysis nutrition` |
| Citric Acid Cycle & Oxidative Phosphorylation | `citric acid cycle oxidative phosphorylation energy` |
| Lipid Metabolism | `lipid metabolism fatty acid oxidation dietary` |
| Energy Metabolism in Disease | `energy metabolism diabetes insulin resistance` |
| Nitrogen Metabolism | `nitrogen metabolism urea cycle amino acid catabolism` |

**Rate limiting and sequential execution:** The Consensus search tool has a rate limit of one query per second. Run searches one at a time — send a query, wait for the response, process the results, then run `sleep 1` before sending the next query. Do not batch multiple Consensus searches into a single message or run them in parallel; they will be rate-limited and fail. This sequential approach also naturally ensures you confirm each result before moving on.

**Handling search failures:** If a Consensus search fails (timeout, error, empty response when results were expected):
1. Wait 3 seconds, then retry the same query once.
2. If the retry also fails, log the failure (section name, query, error), pause the workflow, and ask whether the user wants to retry later or revise the query. Do not move on as if the required search completed.
3. If you hit 3 consecutive failures across any queries, stop searching and alert the user — Consensus may be experiencing issues. Share what you've collected so far and ask how they'd like to proceed.
4. Never silently skip a failed search. Every failure must be noted in the audit log.

**After each search, record what you got.** For each query, note: the query string, how many papers were returned, and which papers (if any) you're selecting. This is the raw material for the audit log.

**Tool constraints to be aware of:** Detect the result cap from the actual Consensus responses rather than assuming a fixed plan tier. Record both papers found and papers shown/returned when the response provides both numbers. Only the papers actually returned are citable. Report any detected cap so the user can distinguish thin literature from limited tool coverage.

**Paper selection criteria** (in priority order):
1. Relevance to the course topic
2. Review papers and meta-analyses over narrow primary research (reviews give students a broader entry point)
3. Higher citation counts (signals quality and influence)
4. Clear connection to the course's applied domain

Select 1-3 papers per section from what Consensus returned, aiming for 15-25 papers total in the final document.

Before Phase 3, verify that the audit log contains `planned_queries` completed Consensus responses. Set `consensus_searches_complete = true` only after this check. If the counts do not match, return to the incomplete search rather than continuing.

### Phase 3: Write Summaries and Discussion Questions

Do not begin this phase unless `consensus_searches_complete = true`. Every selected paper must be traceable to a logged Consensus response from the current conversation.

For each selected paper, write two things. Base your summaries only on information present in the Consensus search results (title, abstract, metadata). If the search result doesn't provide enough context for a meaningful summary, say so briefly rather than inventing details.

**One-sentence summary:** A plain-language sentence that captures what the paper found or reviewed, written for an undergraduate audience. Avoid jargon where possible; if a technical term is necessary, define it in parentheses. The summary should make a student think "I want to read this."

Good: "This review maps how different diets — Mediterranean, Nordic, and vegetarian — reshape the types of fat molecules circulating in your blood, with implications for heart disease risk."

Bad: "This paper reviews lipidomic profiles across dietary interventions and their cardiometabolic implications." (Too jargon-heavy for undergrads.)

**Discussion question:** A thought-provoking question that connects the paper's findings back to one of the course's learning outcomes. The question should push students beyond recall — ask them to apply, analyze, or evaluate. Frame it as something an instructor might pose in class or on an assignment.

Good: "If dietary fat quality can reshape your lipoprotein lipidome, what does this suggest about the biochemical basis for dietary guidelines recommending unsaturated over saturated fats?"

Bad: "What did the authors find?" (Too simple — just recall.)

### Phase 4: Generate the .docx Document

Do not generate the document unless `consensus_searches_complete = true` and the audit log contains every planned Consensus query.

Use ChatGPT's built-in **documents workflow** to create an editable `.docx`. Invoke and follow the available document-creation skill or workflow, including its render-and-verify process. Do not install a document library, run the former bundled JavaScript generator, or hard-code internal paths.

Build the document with:

- A title block containing the course title, optional subtitle, generation date, and publication-year range.
- A short introduction explaining that the readings were selected from Consensus results returned in the current conversation.
- A clearly styled **Course Learning Outcomes** section.
- Numbered papers organized beneath the confirmed topic-section headings.
- For every paper: a clickable title linked to its full Consensus URL, authors, journal, publication year, plain-language summary, and discussion question.
- A final **Search Audit Log** containing each section, query, filters, papers found, papers shown/returned, papers selected, status, failures, detected result caps, and summary counts.

Use a clean academic design with readable headings, consistent spacing, real numbered-list formatting, and tables that fit within page margins. Render the completed `.docx` to page images and inspect every page for overflow, clipping, broken links, awkward page breaks, orphaned headings, or excessive whitespace. Correct any problems and repeat validation until the document is visually sound.

### Phase 5: Deliver to User

Name the file after the course when possible (for example, `Biochemistry-Supplementary-Reading-List.docx`) and make it available through ChatGPT's file-delivery workflow.

Along with the link, include a brief **audit summary** in your chat message. Apply the same sourcing and attribution standards here as in the document — your chat message is part of the deliverable. Format it something like:

> **Search summary:** Ran 12 Consensus queries across 10 topic sections. Consensus returned 34 papers total. Selected 18 for the final reading list. Detected result cap: [state the cap only if the responses established one]. No search failures.
>
> **Sections with limited results:** "Nitrogen Metabolism" returned only 1 relevant paper. You may want to supplement this section manually.
>
> **Failures:** None (or list any that occurred with the query and error).

If any searches failed, any sections had zero results, or any model knowledge was used for any reason, surface those facts here. The goal is full transparency — the user should never have to wonder "did this actually come from Consensus?"

## Important Notes

- **Year range:** Default to papers from the last 1-2 years. If the user asks for a wider range, adjust `year_min` in the Consensus searches accordingly.
- **Consensus result limits:** Detect and report the actual result cap from the responses. Do not infer the user's plan from an assumed fixed number.
- **Non-English syllabi:** The skill works with any language — extract topics and search in English (Consensus indexes English-language papers), but note in the document if the course is taught in another language.
- **Multiple file types:** The syllabus might be a PDF, DOCX, image, or pasted text. Use ChatGPT Work's built-in PDF, document, text, or vision workflow appropriate to the source format.
