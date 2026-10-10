# Adapting the ChatGPT templates for Claude

The three files in `original-chatgpt-templates/` were published by Consensus for ChatGPT (they mention
"ChatGPT Work" and the ChatGPT plugin system). The workflow logic carries over to Claude. The tool-specific
parts do not. Consensus also publishes Claude versions on its website, which were not available when this kit
was assembled; if the user has them, prefer those as the base and apply the customization from the interview.

## Translate these

| In the ChatGPT template | In the Claude skill |
|---|---|
| "ChatGPT Work", "Work in ChatGPT" | Claude chat, or Cowork if she is on the desktop app and wants long tasks |
| `request_user_input` interactive control | Ask in chat with 2 to 4 short numbered options; use Claude's interactive question tool only if it is present |
| Plugin-install request control | Tell her to connect Consensus via **+ > Connectors > Manage connectors > Browse > Consensus > Connect** |
| "Tag @Consensus in the message" | Make sure Consensus is switched on in the **+** menu for the conversation |
| "ChatGPT's built-in documents workflow" and render/inspect page images | Claude's document or file-creation ability if available (it produces .docx); otherwise write the report in chat. Keep the render-and-check habit only if a rendering tool exists. |
| "ChatGPT's file-delivery workflow" | Provide the file for download, or the full text in chat |
| `Consensus:search` tool name | Whatever the Consensus search tool is called in this conversation; look at the tools listed |
| "ChatGPT's public web lookup or command-execution tool" with `curl` (RePORTER, NIH Guide) | Claude's web fetch / web search tool, or code execution if available. If she is on a plan or surface without them, skip the RePORTER step and say so in the audit log. |

## Keep these (they are the reason the templates are trustworthy)

- Preflight: confirm the Consensus tool is callable before doing anything else.
- Sequential searches with at least a one-second pause; confirm each result before the next.
- Retry a failed call once after 3 seconds; stop and alert after 3 consecutive failures; never skip silently.
- Track three counts separately: papers found, papers shown, papers cited. Detect plan caps from the
  "Found N, showing top M" line and report them.
- Only cite papers returned in this session with a retrievable Consensus URL. Label anything else
  `[Not from Consensus - model knowledge]` and exclude it from counts.
- An audit log of every search, its filters, and its outcome.
- Checkpoints: confirm sub-areas and depth with the user before the big searches.
- Supported filters for the Consensus search tool (from the templates): `query` (required), `year_min`,
  `year_max`, `study_types`, `sjr_max`, `human`, `sample_size_min`, `medical_mode`, `exclude_preprints`,
  `duration_min`, `duration_max`. If the tool in the conversation shows a different set, trust the tool.

## Personalization points to put in each generated skill

- Field and sub-field vocabulary, including synonyms and older terminology.
- Default framework (PICO is the template default; SPIDER for qualitative; decomposition for technology).
- Default depth (5, 10 or 20 searches) and a warning of the cost on a free Consensus account.
- Year window defaults, `exclude_preprints` default (the Consensus demo surfaced a preprint and the speaker
  suggested adding a preprint disclaimer rather than banning them; ask which she prefers).
- Citation style for the bibliography.
- Output format and length.
- A rule: always show the proposed sub-areas or section plan and wait for approval before searching.

## Grant-finder specifics

- The shipped grant skill is NIH-only: Consensus gap analysis, then NIH RePORTER (a public API that needs
  POST requests), NIH Guide notices (NOSIs), grants.gov, mechanism matching (F, K, R, etc.) and NIH receipt
  dates. Its own text says to verify budgets and dates against current official NIH pages.
- For a non-NIH researcher, keep the Consensus gap-analysis phase and replace the funder phase. Do not invent
  funder endpoints. At run time, search for the funder's official funded-projects database and open-call
  pages, state which sources were actually reached, and mark anything unreachable as `Not verified`.
- If her field is not biomedical, the NIH institute and study-section sections are probably irrelevant.
  Say so and drop them.
